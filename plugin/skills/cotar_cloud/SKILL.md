# /cotar_cloud — Cotação de Arquitetura AWS

Gera uma estimativa oficial no AWS Pricing Calculator a partir da descrição de uma solução em nuvem, retornando um link compartilhável pronto para enviar ao parceiro (AWS) ou ao cliente.

---

## Quando usar

O usuário invoca `/cotar_cloud` e fornece uma descrição da arquitetura — pode ser o texto da proposta, um resumo técnico, ou uma lista de serviços com volumetria. A descrição pode estar em português ou inglês.

---

## Fluxo obrigatório

### 1. Extrair informações da descrição

Antes de chamar qualquer tool, identifique:

- **Região AWS** — padrão `sa-east-1` (São Paulo) para clientes no Brasil. Se o cliente tiver restrição legal (ex: bancos com LGPD/BACEN), confirme a região antes de prosseguir.
- **Serviços AWS** — liste todos os serviços necessários. Use os paralelos corretos:
  - Armazenamento de dados: `S3`
  - Banco relacional: `RDS` (especificar engine: MySQL, PostgreSQL, etc.)
  - Processamento batch: `Glue` ou `EMR`
  - Compute: `EC2` (especificar família de instância)
  - Serverless compute: `Lambda`
  - Mensageria/streaming: `MSK` (Kafka gerenciado) ou `Kinesis`
  - Data warehouse: `Redshift` (provisionado ou Serverless)
  - Orquestração: `MWAA` (Airflow gerenciado)
  - Segredos/credenciais: `Secrets Manager`
  - Banco de vetores: verificar se disponível na calculadora; se não, calcular à parte
- **Databricks** — se presente, identificar:
  - Tipo de cluster: Job Cluster (mais barato) ou All-Purpose Cluster
  - Família de instância EC2 subjacente (ex: `m5.xlarge`, `r5.2xlarge`)
  - Horas de uso por dia e dias por mês
- **Volumetria** — GB de dados, RPM, eventos/segundo, horas de processamento
- **Ambientes** — padrão: Produção, Homologação, Desenvolvimento

Se alguma informação crítica estiver faltando (especialmente região ou tamanho de instância para serviços de compute), pergunte antes de continuar.

---

### 2. Criar o estimate com grupos de ambiente

Use as tools na sequência:

```
create_estimate
  → description: "Cotação [Nome do Cliente/Projeto] — [data]"

# Para cada ambiente, criar um grupo:
add_service (grupo: "Produção")
add_service (grupo: "Homologação")
add_service (grupo: "Desenvolvimento")
```

**Regras de sizing por ambiente:**

| Ambiente        | Horas de uso    | Hardware         |
| --------------- | --------------- | ---------------- |
| Produção        | 100% (730h/mês) | Tamanho definido |
| Homologação     | ~30% (200h/mês) | Mesmo hardware   |
| Desenvolvimento | ~50% horas prod | 1 tier abaixo    |

Exemplos de "1 tier abaixo": `m5.xlarge` → `m5.large`, `r5.2xlarge` → `r5.xlarge`.

Serviços que não variam por ambiente (ex: Secrets Manager, Route 53) podem ser adicionados fora dos grupos, como serviços compartilhados.

**Sempre usar preços sob demanda (on-demand).** Nunca usar Reserved Instances, Savings Plans ou Spot — a cotação para parceiro sempre considera o preço cheio.

---

### 3. Exportar e retornar o link

```
export_estimate
```

Retorna o link oficial do `calculator.aws`. Esse é o link que vai para o parceiro AWS.

---

### 4. Calcular Databricks separadamente (se aplicável)

O Databricks on AWS usa **duas calculadoras**:

- **AWS Pricing Calculator** → custo da instância EC2 (já incluído no estimate acima)
- **Databricks Calculator** → custo de DBU

**Sempre forneça uma estimativa de DBU** — não omita o cálculo por incerteza. Use a tabela abaixo como referência e sinalize que o arquiteto deve confirmar na calculadora oficial.

#### Tabela de referência de DBU (AWS, pay-as-you-go)

| Tipo de cluster     | DBU/hora por nó | Preço/DBU (sa-east-1) |
| ------------------- | --------------- | --------------------- |
| Jobs Compute        | 1.0 DBU         | ~$0.20                |
| All-Purpose Compute | 1.0 DBU         | ~$0.40                |
| Jobs Compute Light  | 0.5 DBU         | ~$0.20                |

> Esses valores são referência para estimativa. O preço exato varia por instância e plano contratado — confirmar sempre em databricks.com/product/pricing.

#### Fórmula

```
DBU/hora × nº de nós × horas/mês × preço/DBU = custo DBU mensal
```

**Exemplo** — Job Cluster, 1 nó m5.xlarge, 3h/dia × 20 dias:

```
1.0 DBU/h × 1 nó × 60h × $0.20 = $12,00/mês (Produção)
```

Aplique a mesma lógica proporcional para Homologação e Desenvolvimento (mesma regra de horas dos outros serviços).

Informe que o arquiteto deve:

1. Acessar [databricks.com/product/pricing](https://www.databricks.com/product/pricing)
2. Configurar o mesmo cenário para confirmar os valores e obter o print oficial
3. Anexar o print junto ao link AWS na proposta

---

### 5. Serviços fora da calculadora

Se um serviço necessário não estiver disponível na calculadora AWS (ex: Vector Search, licenças de Power Platform), informe explicitamente:

```
⚠️ [Nome do Serviço] não está disponível na calculadora oficial.
Preço disponível em: [URL da pricing page do serviço]
Cálculo manual: [descreva a regra de cobrança e o valor estimado]
Este valor deve ser informado separadamente ao parceiro por e-mail.
```

---

## Formato de saída

Ao final, apresente sempre:

```
✅ Estimate gerado com sucesso

🔗 Link oficial (AWS): https://calculator.aws/...
   → Enviar este link ao parceiro AWS

📊 Resumo de custos mensais:
   Produção:      $X.XXX/mês
   Homologação:   $X.XXX/mês
   Desenvolvimento: $XXX/mês
   ─────────────────────────
   Total AWS:     $X.XXX/mês

[Se houver Databricks:]
🔷 Databricks (calcular separado):
   EC2 já incluído no estimate AWS acima
   DBU estimado: ~$XXX/mês ([tipo de cluster] × [horas] × [DBU/h])
   → Acessar databricks.com/product/pricing para o print oficial
   Total estimado (AWS + Databricks): ~$X.XXX/mês

[Se houver serviços fora da calculadora:]
⚠️ Serviços não cobertos pela calculadora oficial:
   - [serviço]: ~$XX/mês (fonte: [URL])

📋 Serviços incluídos:
   [lista dos serviços adicionados ao estimate]

💡 Próximos passos:
   1. Revisar o estimate no link acima e ajustar parâmetros se necessário
   2. Salvar/exportar o link para anexar à proposta no sistema interno
   [3. Coletar print da calculadora Databricks, se aplicável]
```

---

## Erros comuns a evitar

- Não usar preços com desconto (Reserved, Savings Plans, Spot) — sempre on-demand
- Não esquecer de separar os 3 ambientes — o parceiro espera essa estrutura
- Não somar o custo EC2 do Databricks duas vezes — ele já está no estimate AWS
- Não assumir região sem confirmar com o cliente que tem restrição de compliance
- Não pular a pergunta de sizing quando a descrição for vaga — um cluster errado distorce toda a proposta
