# Cloud Pricing Calculator

Atualizado: 25/06/2026

Skill para Claude Code que gera estimativas de custo em nuvem a partir da descrição de uma arquitetura, retornando um link oficial do AWS Pricing Calculator pronto para enviar ao parceiro AWS.

**Grupo 2:** Sarah + André

## Escopo

### Problema

Arquitetos de soluções montam estimativas de custo manualmente nas calculadoras oficiais de cada provider (AWS, Azure, Databricks). O processo é lento, sujeito a erro e gera retrabalho: a conta interna não basta — para solicitar incentivo ao parceiro, é obrigatório enviar a calculadora oficial preenchida.

### Solução

Skill `/cotar_cloud` para Claude Code, integrada ao MCP oficial da AWS (`aws-samples/sample-aws-pricing-calculator-mcp`). O arquiteto cola a descrição da arquitetura e recebe:

- Link compartilhável oficial do `calculator.aws` com os serviços organizados em grupos de ambiente (Produção, Homologação, Desenvolvimento)
- Estimativa de custo de DBU do Databricks calculada separadamente, com instrução para gerar o print oficial

### Decisões de escopo

**Provider:** AWS + Databricks on AWS — foco do MVP, baseado nas entrevistas com os SAs da Dataside (maior volume de uso e melhor suporte programático via MCP).

**O que está dentro do escopo:**

- Serviços AWS cobertos pela calculadora oficial
- Databricks Job Cluster e All-Purpose Cluster (custo EC2 no estimate + estimativa de DBU separada)
- 3 ambientes: Produção, Homologação e Desenvolvimento com sizing proporcional
- Preços sempre on-demand (sem descontos)

**O que está fora do escopo no MVP:**

- Azure e Google Cloud
- Serviços fora da calculadora oficial (ex: Vector Search) — sinalizados ao arquiteto com link para a pricing page

### Arquitetura da solução

```
Arquiteto → /cotar_cloud + descrição da arquitetura
                  ↓
            Claude Code (SKILL.md)
                  ↓
         MCP: aws-pricing-calculator
         (aws-samples/sample-aws-pricing-calculator-mcp)
                  ↓
    create_estimate → add_service (prod/homolog/dev) → export_estimate
                  ↓
        Link oficial calculator.aws
        + Estimativa DBU Databricks
```

### Referências

- [MCP AWS Pricing Calculator](https://github.com/aws-samples/sample-aws-pricing-calculator-mcp)
- [Calculadora oficial AWS](https://calculator.aws)
- [Calculadora Databricks](https://www.databricks.com/product/pricing)
- [Instalação do plugin](plugin/INSTALL.md)

---

## Log de organização

Registro do planejamento e execução ao longo das duas semanas do desafio.

| Data          | Esperado                                                             | Cumprido                                                              |
| ------------- | -------------------------------------------------------------------- | --------------------------------------------------------------------- |
| 22/06         | Reunião de alinhamento e formulação de perguntas para as entrevistas | Reunião feita                                                         |
| 23/06         | Entrevista com Nelson (SA — Eficiência Operacional)                  | Feito — entendimento do fluxo atual de estimativa e calculadoras      |
| 24/06         | Entrevista com Oscar (Head de Dados e IA)                            | Feito — entendimento do sistema de propostas e critérios de aceitação |
| 25/06         | Decisão de escopo e abordagem técnica                                | Feito — AWS + Databricks via skill Claude Code + MCP                  |
| 26/06 – 29/06 | Desenvolvimento do primeiro protótipo (skill + MCP)                  | Em andamento                                                          |
| 29/06         | Reunião com Cauã para apresentação do primeiro protótipo             | Agendado                                                              |
| 01/07         | Aprimoramento do protótipo e apresentação para SA / Oscar            | A ser marcado                                                         |
| 03/07         | **DATA FINAL DE ENTREGA**                                            |                                                                       |
