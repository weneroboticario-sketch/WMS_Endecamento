# Sistema Web Corporativo de Endereçamento de Estoque

Sistema profissional para operação logística com login, bipagem inteligente, consulta automática de localização, etiquetas CODE128, importação/exportação Excel no modelo `LinhaSeparacao` e histórico básico de auditoria.

## Stack

- Frontend: React + TypeScript + Vite
- Backend: Node.js + Express
- Banco local: SQLite via `node:sqlite`
- Excel: `xlsx`
- Código de barras: `jsbarcode`
- PDF: `jspdf`
- PWA: `manifest.webmanifest` + `service worker`

## Como instalar

```bash
pnpm install
```

## Como rodar em desenvolvimento

```bash
pnpm dev
```

Abra no computador servidor:

```text
http://127.0.0.1:5173
```

API local:

```text
http://127.0.0.1:3333
```

## Como rodar em produção local

```bash
pnpm build
pnpm start
```

Abra:

```text
http://127.0.0.1:3333
```

## Deploy na Vercel

O projeto está preparado para Vercel com:

- `vercel.json`
- Frontend Vite em `dist`
- API serverless catch-all em `api/[...path].js`
- Build command: `pnpm build`
- Output directory: `dist`

No painel da Vercel:

1. Importe o repositório do GitHub.
2. Em `Framework Preset`, use `Vite`.
3. Em `Build Command`, use:

```bash
pnpm vite build
```

4. Em `Output Directory`, use:

```text
dist
```

5. Se o repositório tiver uma pasta acima do projeto, configure `Root Directory` para:

```text
enderecamento-estoque-profissional
```

6. Configure as variáveis de ambiente:

```text
AUTH_SECRET=uma-chave-grande-e-secreta
DATABASE_URL=postgresql://usuario:senha@host:5432/banco?sslmode=require
```

Sem `DATABASE_URL`, a API sobe em modo demonstração em memória. Esse modo serve para testar o deploy, mas os dados podem sumir porque funções serverless não mantêm memória de forma confiável.

Para produção real na Vercel, use Postgres. Opções comuns:

- Vercel Postgres/Neon
- Supabase Postgres
- Railway Postgres
- Neon Postgres

Depois de configurar `DATABASE_URL`, faça `Redeploy`. A API cria as tabelas automaticamente no primeiro acesso.

## Como acessar pela rede local

O frontend e o backend escutam em `0.0.0.0`, permitindo acesso por outros dispositivos na mesma rede.

No computador servidor, descubra o IP:

```powershell
ipconfig
```

Em outro computador, tablet ou celular na mesma rede:

```text
http://IP-DO-SERVIDOR:5173
```

Em produção local:

```text
http://IP-DO-SERVIDOR:3333
```

## Como usar no celular

Abra o sistema no navegador do celular e use `Adicionar à tela inicial`. O app inclui PWA básico para abrir em modo standalone.

Leitores Bluetooth/USB que funcionam como teclado são compatíveis com os campos de bipagem.

## Login

Usuários de teste:

```text
admin@estoque.local / admin123
supervisor@estoque.local / supervisor123
operador@estoque.local / operador123
```

Perfis:

- Administrador: acesso total
- Supervisor: consulta, exportação, edição e relatórios
- Operador: bipagem, consulta e endereçamento

## Fluxo de bipagem

1. Faça login.
2. Acesse `Bipagem / Endereçamento`.
3. Bipe ou digite o SKU.
4. Pressione Enter.
5. Se o SKU já tiver localização, o sistema mostra onde guardar.
6. Se o SKU não tiver localização, o sistema pede a prateleira.
7. Bipe a prateleira.
8. O sistema normaliza o endereço e grava automaticamente se a configuração estiver ativa.
9. O campo SKU volta ao foco para o próximo produto.

## Padrão da prateleira

Formato técnico:

```text
R01-RK01-L01-A
```

Também aceita entradas como:

```text
R1-RK1-L1-A
R01-RK01-L01-A
R10-RK03-L04-AA
```

O sistema normaliza internamente para o padrão com dois dígitos.

## Regras da Área Linha Separação

- 1 - Alto Giro
- 2 - Médio Giro
- 3 - Área RF
- 4 - Baixo Giro
- 5 - Picking by Light

Na tela aparece o nome. No Excel sai o código numérico.

## Conferencia Obrigatoria

O campo existe no banco e na exportação, mas nesta fase sai sempre como `0`.

## Etiquetas

A tela `Gerar Etiquetas` permite:

- Etiqueta individual
- Etiquetas em lote
- Etiquetas a partir de planilha `LinhaSeparacao`
- Impressão
- PDF

Cada etiqueta contém código de barras CODE128 e texto visível do endereço.

## Exportar Excel

A exportação gera:

```text
LinhaSeparacao_Enderecamento_DD-MM-AAAA.xlsx
```

A aba se chama:

```text
LinhaSeparacao
```

Colunas exatamente na ordem:

```text
Nome estacao
Nr Rack
Area Linha Separaçao
Linha
Coluna
Codigo Material
Conferencia Obrigatoria
```

Filtros disponíveis:

- Rua
- Rack
- Área
- Período
- Usuário
- SKU

## Importar Excel

A tela `Importar Excel` lê a aba `LinhaSeparacao` no mesmo modelo. Se faltar coluna obrigatória, o sistema informa:

```text
Coluna obrigatória ausente: [nome da coluna]
```

## Histórico e auditoria

O sistema registra:

- Login realizado
- SKU consultado
- Localização consultada
- SKU endereçado
- Endereço removido
- Endereço alterado
- Excel importado
- Excel exportado

## Banco de dados

O SQLite é criado automaticamente em:

```text
data/enderecamento.sqlite
```

Tabelas principais:

- `usuarios`
- `localizacoes`
- `enderecamentos`
- `historico`
- `configuracoes`

## Módulos futuros

A arquitetura foi organizada para evoluir para:

- Conferência
- Inventário
- Movimentação
- Relatórios de divergências, produtos sem endereço e endereços vazios
