# DataCore

Plataforma de centralização, organização e exploração de dados empresariais provenientes de planilhas (Excel/CSV).

## Status do Projeto

> ⚠️ **Fase de planejamento.** Este repositório ainda **não contém código publicado** — encontra-se em fase de documentação e definição de arquitetura. Tudo o que está descrito abaixo representa a visão do produto e o plano de execução; nenhuma funcionalidade está implementada.

## Visão Geral

A plataforma permitirá que usuários enviem suas planilhas, organizem-nas em pastas (com relações entre si) e, futuramente, explorem e cruzem esses dados de forma inteligente.

**Pitch:** projeto de portfólio que combina backend, processamento de arquivos, banco de dados, autenticação, armazenamento em nuvem e, futuramente, dados/IA.

## Conceito do Produto

O usuário acessará um painel e organizará seus arquivos assim:

```
📊 Meu DataCore
├── 📁 Financeiro
│   ├── 📄 vendas_janeiro.xlsx
│   ├── 📄 vendas_fevereiro.xlsx
│   └── 📁 Relatórios
│       └── 📄 fechamento.xlsx
├── 📁 Recursos Humanos
│   ├── 📄 funcionarios.xlsx
│   └── 📄 salarios.xlsx
└── 📁 Projetos
    ├── 📄 projeto_a.xlsx
    └── 📄 projeto_b.csv
```

O diferencial: não é apenas um "Google Drive de planilhas" — o sistema **entenderá** os dados.

Ao enviar `vendas_janeiro.xlsx`, o backend **identificará** automaticamente as colunas (Data, Cliente, Produto, Quantidade, Valor, Vendedor) e **passará a oferecer**:

- 📊 Visualizar dados
- 🔎 Pesquisar
- 📈 Criar gráficos *(futuro)*
- 🔄 Atualizar dados
- 📥 Baixar original
- 🔗 Relacionar com outra planilha *(futuro)*

## Arquitetura e Stack

```
Frontend
   ↓
FastAPI
   ↓
Services
   ↓
PostgreSQL
```

### Stack Backend
- **FastAPI** — framework principal da API
- **PostgreSQL** — banco de dados relacional (metadados)
- **SQLAlchemy** — ORM
- **Pydantic** — validação de dados
- **JWT/OAuth2** — autenticação

### Bibliotecas de Processamento
- **pandas** — leitura e processamento de Excel/CSV
- **openpyxl** — leitura de arquivos `.xlsx`

### Armazenamento
Separado em duas camadas:

1. **Arquivo físico** → Object Storage (MinIO local, AWS S3 ou Supabase Storage)
2. **Metadados** → PostgreSQL (`User`, `Folder`, `File`, `Dataset`, `Column`, `Permission`)

> ⚠️ **Decisão arquitetural:** não transformar todas as células das planilhas em registros do PostgreSQL desde o início — com 100 arquivos × 50.000 linhas × 20 colunas, o volume de dados explode rapidamente. No início: arquivo bruto no Object Storage + metadados no Postgres, processando sob demanda ou em background. Se o produto crescer, evoluir para `S3 → Parquet → DuckDB → Data Processing → PostgreSQL`.

### Frontend
- **MVP:** HTML + CSS + JavaScript simples
- **Produto mais sério (futuro):** React + TypeScript ou Next.js

## Modos de Deploy (visão)

Um diferencial planejado é oferecer **dois modos de operação** usando a mesma base de código, mudando apenas a camada de storage:

### Modo Cloud (futuro)
- Object Storage: AWS S3 ou Supabase Storage
- Banco: PostgreSQL gerenciado (RDS, Supabase, etc.)
- Acesso via internet, multiusuário remoto

### Modo Rede Local / On-Premise (visão pós-MVP)
- Object Storage: **MinIO** rodando em servidor local — compatível com S3, então o mesmo código da aplicação funciona sem alteração
- Banco: PostgreSQL local, no mesmo servidor
- Acesso via IP local pelos computadores da rede (funciona como um "NAS inteligente" para a empresa)
- Sem dependência de internet para uso diário
- Indicado para empresas com restrições de compliance/dados sensíveis

> **On-premise está fora do escopo do MVP** e é tratado como visão futura. Requer um servidor central sempre ligado executando FastAPI + PostgreSQL + MinIO.

## Roadmap

### Fase 1 — Arquivos (base do produto)
- [ ] Cadastro/login
- [ ] Dashboard
- [ ] Criar pasta
- [ ] Renomear pasta
- [ ] Excluir pasta
- [ ] Upload de `.xlsx`
- [ ] Upload de `.csv`
- [ ] Download de arquivo
- [ ] Excluir arquivo

### Fase 2 — Processamento
- [ ] Ler Excel
- [ ] Ler CSV
- [ ] Identificar colunas automaticamente
- [ ] Mostrar preview dos dados

### Fase 3 — Dados
- [ ] Filtros
- [ ] Busca
- [ ] Ordenação
- [ ] Gráficos
- [ ] Estatísticas básicas
- [ ] Relacionamentos entre datasets

### Fase 4 — IA
- [ ] Perguntas em linguagem natural sobre os dados
- [ ] Geração automática de consultas
- [ ] Visualização dos resultados em gráficos

**Pós-MVP (planejado):** versionamento de arquivos, validação de schema no reenvio, log de auditoria, permissões granulares por linha/coluna, alertas automáticos, templates de planilha por setor e exportação de resultados processados.

## Escopo do MVP

O objetivo é uma versão demonstrável do fluxo fechado: **logar → criar uma pasta → subir uma planilha → ver o preview dos dados com colunas identificadas → baixar o arquivo de volta**. Esse fluxo, sozinho, demonstra o conceito central do produto — o sistema entende os dados, não é só um "Drive de Excel".

### Stack mínima obrigatória
| Camada | Tecnologia |
|---|---|
| API | FastAPI |
| Banco (metadados) | PostgreSQL |
| ORM | SQLAlchemy |
| Validação | Pydantic |
| Armazenamento de arquivo | MinIO (local) ou Supabase Storage (cloud rápido) |
| Processamento de planilha | pandas + openpyxl |
| Autenticação | JWT (simples, sem OAuth2 social no MVP) |
| Frontend | HTML + CSS + JavaScript simples |

> Celery + Redis **não são obrigatórios no MVP** — o processamento inicial pode ser síncrono (o usuário espera alguns segundos ao subir o arquivo).

### Funcionalidades mínimas
1. Cadastro e login de usuário (JWT)
2. Criar / renomear / excluir pasta
3. Upload de arquivo `.xlsx` e `.csv`
4. Leitura do arquivo com pandas e identificação automática das colunas
5. Preview dos dados em tabela (as primeiras N linhas)
6. Download do arquivo original
7. Excluir arquivo

### Explicitamente FORA do MVP (fica para depois)
- Relacionamento entre planilhas (grafo de datasets)
- Busca semântica e perguntas em linguagem natural (IA)
- Gráficos e estatísticas avançadas
- Processamento assíncrono (Celery/Redis)
- Permissões granulares por linha/coluna
- Versionamento de arquivos e log de auditoria
- Modo de deploy on-premise (rede local via MinIO em servidor próprio)
- Alertas automáticos e templates por setor

## Contribuição

Contribuições são bem-vindas. Abra uma issue ou um pull request.

## Licença

[MIT](LICENSE) © 2026 Kaio Mendonça