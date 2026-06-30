# Instalação do Plugin cotar_cloud

## Pré-requisitos

- Node.js 18+
- Claude Code (extensão VS Code ou CLI)
- Git

---

## 1. Clonar o repositório com o submódulo do MCP

```bash
git clone --recurse-submodules https://github.com/AndreViniNe/dais-cloud-calculator.git
cd dais-cloud-calculator/
```

> Se já clonou sem `--recurse-submodules`, execute dentro da pasta do projeto:
> ```bash
> git submodule update --init --recursive
> ```

---

## 2. Build do MCP server

```bash
cd plugin/mcps/aws-pricing-calculator
npm install
npm run build
cd ../../..
```

Após o build, o arquivo `plugin/mcps/aws-pricing-calculator/dist/mcp-server.js` estará disponível.

---

## 3. Registrar o MCP no Claude Code. 
### 3.1 Claude Code CLI. 
Adicione o MCP ao config do Claude Code. Abra ou crie o arquivo ```~/.claude/mcp.json``` e adicione:
```
{
  "mcpServers": {
    "aws-pricing-calculator": {
      "command": "node",
      "args": ["/caminho/absoluto/para/seu-repo/plugin/mcps/aws-pricing-calculator/dist/mcp-server.js"]
    }
  }
}
``` 
Substitua pelo caminho absoluto real da sua máquina. Depois, no terminal, na raiz do seu projeto:
```bash
claude
```

### 3.2 Cowork (Claude Desktop)
Vá em **Settings → Capabilities → MCP Servers**, adicione um novo servidor com:

- **Command**: node
- **Args**: ```/caminho/absoluto/para/seu-repo/plugin/mcps/aws-pricing-calculator/dist/mcp-server.js```

Para a skill, copie a pasta ```plugin/skills/cotar_cloud``` para ```~/.claude/skills/```.


### 3.3 Extensão VS Code
No terminal, na raiz do projeto:

```bash
claude mcp add aws-pricing-calculator node "$(pwd)/plugin/mcps/aws-pricing-calculator/dist/mcp-server.js"
```

Verifique se foi registrado:

```bash
claude mcp list
```

Deve aparecer `aws-pricing-calculator` na lista.

---

## 4. Instalar o comando /cotar_cloud

```bash
mkdir -p .claude/commands
cp plugin/skills/cotar_cloud/SKILL.md .claude/commands/cotar_cloud.md
```

O comando fica disponível no nível do projeto — qualquer pessoa que clonar o repositório e seguir este guia terá acesso a ele.

---

## 5. Testar a instalação

Reinicie o Claude Code no VS Code (feche e reabra o painel) e envie:

```
/cotar_cloud

Teste simples: EC2 t3.medium em sa-east-1,
S3 com 100GB, RDS MySQL db.t3.small. Só produção.
```

Se retornar um link `https://calculator.aws/...`, a instalação está correta.

---

## Estrutura de arquivos

```
projeto/
├── README.md
├── .claude/
│   └── commands/
│       └── cotar_cloud.md          ← comando gerado no passo 4
└── plugin/
    ├── INSTALL.md                  ← este arquivo
    ├── mcp.json                    ← referência de configuração do MCP
    └── skills/
    │   └── cotar_cloud/
    │       └── SKILL.md            ← fonte do comando
    └── mcps/
        └── aws-pricing-calculator/ ← submódulo do MCP da AWS
            ├── dist/
            │   └── mcp-server.js   ← gerado após npm run build
            └── package.json
```

---

## Referências

- MCP Server AWS: https://github.com/aws-samples/sample-aws-pricing-calculator-mcp
- Calculadora oficial AWS: https://calculator.aws
- Calculadora Databricks: https://www.databricks.com/product/pricing