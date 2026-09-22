
# 🚀 Pipeline Lakehouse com Databricks, dbt e Integração Contínua

## 📖 Objetivo

Este projeto demonstra a construção de uma solução de Engenharia de Dados utilizando **Databricks, Delta Lake e dbt**, aplicando práticas comuns em ambientes corporativos.

A solução contempla o fluxo de extração, persistência, transformação, qualidade de dados, rastreabilidade e integração contínua, desde a origem em **SQL Server** até a disponibilização dos dados para consumo analítico no **Power BI**.

---

# 🏗 Arquitetura da Solução

![Pipeline ELT Databricks + DBT](https://github.com/devrafael26/dbt-databricks-cicd/blob/main/ELT%20Databricks%20DBT.png?raw=true)

## Fluxo de Dados

```text
SQL Server
     │
     ▼
Python / Notebook
     │
     ▼
Databricks + Delta Lake
     │
     ▼
dbt
Raw → Staging → Mart
     │
     ▼
Power BI
```

## Automação e Integração Contínua

```text
GitHub
   │
   ▼
GitHub Actions
   │
   ├── dbt deps
   ├── dbt parse
   ├── dbt ls
   └── dbt build + tests
                │
                ▼
           Databricks
```

O fluxo de dados e o fluxo de automação são independentes: o **GitHub Actions** não transporta dados para o Power BI. Ele automatiza a validação e execução do projeto dbt.

---

# 🛠 Stack Tecnológica

| Tecnologia | Finalidade |
|---|---|
| SQL Server | Banco de dados de origem |
| Python | Extração dos dados |
| Databricks | Plataforma Lakehouse |
| Delta Lake | Armazenamento transacional |
| dbt | Transformações, testes e documentação |
| GitHub Actions | Automação e Integração Contínua |
| Power BI | Visualização e consumo analítico |

---

# 🧠 Decisões de Arquitetura

O projeto foi desenvolvido com o objetivo de reproduzir práticas encontradas em cenários reais de Engenharia de Dados.

## Por que Databricks?

- Plataforma unificada para processamento e análise de dados.
- Integração nativa com Delta Lake.
- Escalabilidade para workloads analíticos.
- Suporte a pipelines de dados e processamento distribuído.

## Por que Delta Lake?

- Transações ACID.
- Evolução de schema.
- Versionamento de dados.
- Time Travel.
- Confiabilidade para workloads analíticos.

## Por que dbt?

- Organização das transformações em camadas.
- Reutilização de código SQL.
- Construção de dependências entre models com `ref()`.
- Testes automatizados de qualidade.
- Documentação associada aos modelos.
- Facilidade de manutenção e rastreabilidade.

## Por que GitHub Actions?

- Automação das validações.
- Integração Contínua.
- Execução padronizada do projeto dbt.
- Redução de erros manuais.
- Validação automática de alterações realizadas no código.

---

# 🏛 Arquitetura de Dados

A solução foi organizada em múltiplas camadas para separar responsabilidades, facilitar manutenção e aumentar a rastreabilidade.

## Bronze

A camada Bronze representa a persistência inicial dos dados extraídos da origem no Databricks.

### Características

- Dados próximos ao formato recebido da origem.
- Persistência em Delta Lake.
- Sem aplicação das principais regras de negócio.
- Base para as transformações realizadas posteriormente.

> A camada **Bronze** representa a persistência inicial no Databricks, enquanto a camada **Raw** faz parte da estrutura lógica do projeto dbt.

---

## Raw

A camada Raw representa os dados próximos à estrutura da fonte e concentra validações relacionadas à origem.

Nesta camada foram implementados mecanismos para detectar alterações estruturais antes que elas sejam propagadas.

### Validações utilizadas

- `assert_source_structure`
- Validação de tipos de dados
- Validação de colunas esperadas
- Testes de domínio e nulidade quando aplicáveis

O teste customizado `assert_source_structure` consulta metadados da fonte e compara a estrutura encontrada com a estrutura esperada definida no projeto.

---

## Staging

A camada Staging é responsável pela padronização e preparação dos dados.

### Responsabilidades

- Padronização de colunas.
- Conversão e tratamento de tipos.
- Limpeza dos dados.
- Normalização.
- Preparação para consumo analítico.
- Aplicação de testes de qualidade.

### Exemplos de testes

- `not_null`
- `unique`
- `non_negative`
- `accepted_values`

---

## Mart

A camada Mart representa a camada destinada ao consumo analítico.

Nesta etapa são aplicadas agregações e regras de negócio para disponibilizar modelos prontos para ferramentas de Business Intelligence.

Exemplos de análises produzidas pelo projeto incluem:

- Receita mensal.
- Clientes com maior volume de compras.
- Produtos mais vendidos.
- Distribuição de pagamentos.
- Análises por categoria.

---

# ✅ Qualidade dos Dados

Um dos objetivos do projeto é detectar inconsistências antes que elas impactem as camadas analíticas.

Os testes são declarados principalmente nos arquivos `schema.yml` associados aos models e sources.

## Testes nativos do dbt utilizados

- `not_null`
- `unique`
- `accepted_values`

## Testes customizados

### `non_negative`

Implementado em:

```text
macros/non_negative.sql
```

Valida se determinadas métricas numéricas apresentam valores negativos indevidos.

### `assert_source_structure`

Implementado em:

```text
macros/assert_source_structure.sql
```

Valida a estrutura da fonte comparando colunas e tipos esperados com os metadados encontrados no banco de origem.

## Como os testes estão organizados

```text
schema.yml
    │
    ├── associa testes aos models
    └── associa testes às colunas
            │
            ▼
Testes nativos do dbt
ou
macros customizadas
```

A pasta `tests/` está disponível para **singular tests**, mas esse padrão não foi necessário na implementação atual.

---

# 🔗 Dependências entre Models

As dependências entre os models são definidas utilizando `ref()`.

Exemplo conceitual:

```sql
select *
from {{ ref('stg_ecommerce') }}
```

Isso permite ao dbt construir automaticamente o DAG de execução e determinar a ordem correta entre os modelos.

Exemplo:

```text
src_ecommerce
      │
      ▼
stg_ecommerce
      │
      ├── top_customers
      ├── monthly_revenue
      ├── top_5_products
      └── sales_by_category
```

---

# 🔄 Integração Contínua

O projeto utiliza **GitHub Actions** para automatizar a validação das alterações realizadas no código dbt.

O workflow executa etapas como:

```bash
dbt deps
dbt parse
dbt ls --select staging+
dbt build --select staging+
```

## O que cada etapa faz

### `dbt deps`

Instala as dependências declaradas no projeto.

### `dbt parse`

Valida a estrutura do projeto e processa os arquivos dbt.

### `dbt ls`

Lista os recursos selecionados e ajuda a validar o escopo da execução.

### `dbt build`

Executa os models selecionados e os testes associados.

Neste projeto, a seleção:

```bash
--select staging+
```

executa os recursos de Staging e seus descendentes no DAG.

> O workflow atual caracteriza principalmente um processo de **Integração Contínua (CI)**. Não há uma etapa separada de promoção/deploy para um ambiente de produção.

---

# 📚 Práticas de Governança e Rastreabilidade

O projeto incorpora práticas que contribuem para governança e rastreabilidade dos dados.

Entre elas:

- Organização em camadas.
- Versionamento de código com Git.
- Histórico de alterações.
- Testes automatizados.
- Documentação associada aos models.
- DAG de dependências construído pelo dbt.
- Separação entre transformação, testes e automação.
- Padronização do processo de execução.

O projeto não representa uma implementação completa de governança corporativa com recursos como RBAC, mascaramento de dados ou políticas centralizadas de acesso.

---

# 📊 Consumo dos Dados

Após o processamento, os modelos da camada **Mart** são disponibilizados para consumo através do **Power BI**.

Isso permite construir dashboards e indicadores utilizando dados já transformados, documentados e validados.

---

# 📂 Estrutura do Projeto

```text
.
├── .github/
│   └── workflows/
│       └── dbt-ci-cd.yml
│
├── models/
│   ├── raw/
│   │   └── sqlserver/
│   │
│   ├── staging/
│   │   ├── databricks/
│   │   └── sqlserver/
│   │
│   └── mart/
│       └── databricks/
│
├── macros/
│   ├── assert_source_structure.sql
│   └── non_negative.sql
│
├── tests/
├── seeds/
├── snapshots/
├── analyses/
│
├── notebooks/
│   └── extract_sqlserver.ipynb
│
├── data/
│   └── sample/
│
├── powerbi/
│   └── relatorio_vendas.pbix
│
├── docs/
│   └── images/
│       └── architecture_elt_databricks_dbt.png
│
├── dbt_project.yml
├── packages.yml
├── .gitignore
└── README.md
```

---

# 📁 Papel das Principais Pastas

## `models/`

Contém as transformações SQL executadas pelo dbt.

Os models estão organizados em camadas:

```text
raw
staging
mart
```

## `macros/`

Contém lógica reutilizável escrita com SQL e Jinja.

Neste projeto, também contém a implementação dos testes customizados:

```text
non_negative.sql
assert_source_structure.sql
```

## `notebooks/`

Contém o notebook utilizado para extração dos dados do SQL Server.

## `tests/`

Reservada para singular tests em SQL.

Atualmente não possui testes implementados porque os testes do projeto foram construídos utilizando `schema.yml` e macros.

## `seeds/`

Reservada para arquivos CSV que poderiam ser carregados diretamente pelo dbt com `dbt seed`.

Não é utilizada atualmente.

## `snapshots/`

Reservada para snapshots do dbt, normalmente utilizados para histórico de alterações e cenários de SCD.

Não é utilizada atualmente.

## `analyses/`

Reservada para queries analíticas que podem ser compiladas pelo dbt sem serem materializadas como models.

Não é utilizada atualmente.

## `.github/workflows/`

Contém o workflow responsável pela automação e Integração Contínua.

---

# 🧪 Execução Local

Com o ambiente dbt configurado e um `profiles.yml` válido, o projeto pode ser validado localmente.

## Validar configuração

```bash
dbt debug
```

## Validar estrutura do projeto

```bash
dbt parse
```

## Listar recursos

```bash
dbt ls
```

## Executar testes

```bash
dbt test
```

## Executar models e testes associados

```bash
dbt build
```

## Executar Staging e seus descendentes

```bash
dbt build --select staging+
```

---

# 🎯 Principais Competências Demonstradas

- Engenharia de Dados
- Arquitetura Lakehouse
- Databricks
- Delta Lake
- dbt
- SQL
- Python
- ELT
- Data Quality
- Testes Automatizados
- Macros dbt
- Git
- GitHub Actions
- Integração Contínua
- Arquitetura em Camadas
- Modelagem Analítica
- Rastreabilidade de Dados
- Práticas de Governança
- Power BI

---

# 📌 Observações

Este projeto foi desenvolvido com objetivo de estudo e demonstração prática de conceitos de Engenharia de Dados.

Alguns componentes podem ser evoluídos em versões futuras, como:

- Automação completa da ingestão.
- Separação formal entre ambientes de desenvolvimento, homologação e produção.
- Deploy contínuo para produção.
- Implementação de políticas de segurança e governança com Unity Catalog.
- Observabilidade e monitoramento de pipelines.
- Expansão da cobertura de testes.

---

# 👨‍💻 Autor

**Rafael Diniz Ramos**

Data Engineer | Data Platform | Analytics Engineering

