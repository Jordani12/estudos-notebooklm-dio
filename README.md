# 🐍 Python Intermediário: APIs + Persistência com SQLite

Caderno temático criado com o **NotebookLM** como parte do desafio de projeto "Explore o Poder do NotebookLM" (DIO). O objetivo foi usar IA para curar, organizar e sintetizar conhecimento técnico sobre consumo de APIs REST em Python e persistência de dados com SQLite.

## 🎯 Contexto e Objetivos

Atualmente estou avançando em Python além dos fundamentos (POO, comprehensions, módulos da stdlib) e entrando em **bibliotecas externas** — já tendo trabalhado com `requests` num projeto de consumo de API (geocoding + forecast). O próximo passo natural no meu aprendizado é **SQLite**, para persistência de dados.

Por isso, o recorte deste caderno temático é:

> **Consumo de APIs externas com `requests` (incluindo tratamento de erros e retry) + introdução à persistência de dados com `sqlite3`.**

Objetivo de estudo: entender como estruturar um fluxo completo — requisitar dados de uma API, tratar falhas de rede de forma resiliente, e persistir esses dados de forma segura num banco SQLite.

## 📚 Curadoria de Fontes

| # | Fonte | Tipo | O que cobre |
|---|-------|------|--------------|
| 1 | [Data Persistence — Python 3.14.7 Documentation](https://docs.python.org/3/library/persistence.html) | Documentação oficial | Visão geral dos mecanismos de persistência de dados do Python |
| 2 | [sqlite3 — DB-API 2.0 interface for SQLite databases](https://docs.python.org/3/library/sqlite3.html) | Documentação oficial | Referência completa do módulo `sqlite3`, conexões, cursores, DB-API 2.0 |
| 3 | [SQLite Tutorial - An Easy Way to Master SQLite Fast](https://www.sqlitetutorial.net/) | Tutorial prático | Criação de tabelas, constraints (PK, FK, NOT NULL, UNIQUE, CHECK), design de schema |
| 4 | [How to Use Python Requests Retry with Proxying](https://www.roundproxies.com/blog/how-to-use-python-requests-retry-with-proxying/) | Artigo técnico | Retry logic, backoff exponencial, tratamento de erros HTTP temporários |
| 5 | [vinta/awesome-python](https://github.com/vinta/awesome-python) | Repositório curado (GitHub) | Panorama geral do ecossistema Python (bibliotecas e frameworks) |

**Critério de seleção:** priorizei variedade de formato (2 docs oficiais + 1 tutorial + 1 artigo técnico + 1 repositório curado) e relevância direta ao recorte escolhido, descartando fontes redundantes (múltiplos "awesome-python" repetidos) ou fora de escopo (ex: `urllib.request`, já substituído por `requests` no meu fluxo de trabalho).

## 🛠️ Engenharia de Prompts e Cicatrizes

Documentação das perguntas estratégicas feitas ao NotebookLM, o que funcionou e os ajustes necessários.

### Prompt 1 — Comparação entre fontes
> *"Compare como as fontes oficiais de sqlite3 e o tutorial explicam a criação de tabelas — há diferenças de abordagem?"*

**Resultado:** excelente. A IA gerou uma tabela comparativa clara mostrando que a documentação oficial foca em **execução via código Python** (`sqlite3.connect()`, `cur.execute()`), enquanto o SQLite Tutorial foca em **fundamentação SQL/DDL** (design de schema, constraints). Também identificou a diferença de tratamento de tipos: a doc oficial destaca a *tipagem flexível* do SQLite, o tutorial detalha *type affinity* e *storage classes*.

*Sem cicatrizes aqui — prompt específico o suficiente para gerar resposta rica de primeira.*

### Prompt 2 — Glossário guiado por fonte específica
> *"Gere um glossário dos termos de persistência de dados com SQLite baseado nessas fontes"*

**Resultado:** ótimo glossário estruturado em categorias (Conceitos Fundamentais, Conexão e Execução, Segurança e Transações, Tipagem e Mapeamento Extensível), cobrindo desde `Connection`/`Cursor` até `placeholders` para prevenção de SQL injection e o sistema de `Adapter`/`Converter` para tipos customizados.

**Dica de ouro aplicada:** ao restringir a fonte (clicando na fonte `sqlite3` especificamente antes de perguntar), a resposta ficou mais precisa e técnica do que seria com todas as 5 fontes ativas — evitou diluição de foco.

### Prompt 3 — Boas práticas de resiliência HTTP
> *"Quais as principais boas práticas de tratamento de erro e resiliência em requisições HTTP apresentadas nas fontes?"*

**Resultado:** resposta estruturada em 4 blocos práticos: (1) lógica de retry com `HTTPAdapter`/`urllib3.util.retry.Retry`, (2) filtragem de erros temporários (500, 502, 503, 504, 429) vs. permanentes (404), (3) backoff exponencial (`delay = backoff_factor * (2 ** retry_count)`), (4) combinação de retry com rotação de proxy para web scraping.

*Prompt já veio bem direcionado por ter especificado "apresentadas nas fontes" — evitou que a IA trouxesse conhecimento genérico fora do que foi curado.*

### Prompt 4 — Síntese de integração (o mais valioso)
> *"Como integrar consumo de API (requests) com persistência em SQLite num projeto Python?"*

**Resultado:** a resposta mais completa do caderno — um fluxo de 4 etapas conectando **todas** as fontes: cliente HTTP resiliente com `Session`+`Retry` → consumo/parsing do payload JSON → inicialização de conexão e schema no SQLite → persistência seno com `executemany()` e `placeholders` (prevenção de SQL injection) → gestão de transações (`commit`/`rollback` via `with conn:`).

**Cicatriz real:** esse prompt só funcionou bem *depois* dos prompts 1-3 terem sido feitos — a IA conseguiu cruzar informação das duas fontes (requests + sqlite3) porque o contexto de cada uma já tinha sido explorado individualmente antes. Perguntar isso "a frio" como primeira pergunta provavelmente traria uma resposta mais rasa.

## 📖 Miniguia de Estudo (Entrega Final)

### Resumo Estruturado

**Consumo de API com `requests`:**
- Use `requests.Session()` reutilizável combinada com `HTTPAdapter` para configurar retries automáticos
- Trate apenas erros temporários (500, 502, 503, 504, 429) com retry — erros permanentes (404) não devem ser reexecutados
- Aplique backoff exponencial entre tentativas para não sobrecarregar o servidor
- Sempre defina `timeout` explícito nas requisições

**Persistência com `sqlite3`:**
- Conecte com `sqlite3.connect()` e crie um cursor com `con.cursor()`
- Use **sempre** placeholders (`?` ou `:nome`) em queries parametrizadas — nunca concatenação de strings (previne SQL injection)
- Para inserções em lote, prefira `executemany()` por performance
- Gerencie transações explicitamente com `commit()`/`rollback()`, ou use `with conn:` para commit automático
- Feche a conexão com `con.close()` para evitar vazamento de recursos

### Glossário

| Termo | Definição |
|-------|-----------|
| **DB-API 2.0** | Especificação padrão do Python (PEP 249) para interfaces de banco de dados |
| **Connection** | Objeto que representa a conexão aberta com o banco (disco ou memória) |
| **Cursor** | Estrutura usada para executar comandos SQL e percorrer resultados |
| **Placeholder** | Marcador de posição (`?` ou `:nome`) usado em queries parametrizadas para prevenir SQL injection |
| **Tipagem Flexível** | Recurso do SQLite em que declarar tipo de coluna é opcional |
| **Retry Logic** | Estratégia de reenviar requisições automaticamente após falhas temporárias |
| **Backoff Exponencial** | Técnica de aumentar progressivamente o intervalo entre tentativas de retry |
| **Transient Error** | Erro temporário (ex: 503) que justifica retry, diferente de erro permanente (ex: 404) |

### Prompts Reutilizáveis

```
1. "Compare como [fonte A] e [fonte B] explicam [conceito X] — há diferenças de abordagem?"
2. "Gere um glossário dos termos de [tema] baseado nessas fontes"
3. "Quais as boas práticas de [tema] apresentadas nas fontes?" (força a IA a ficar restrita ao material curado)
4. "Como integrar [conceito A] com [conceito B] num projeto real?" (funciona melhor depois de já ter explorado cada conceito separadamente)
```

---

*Projeto desenvolvido como parte do desafio "Treinando uma IA de Aprendizagem: Explore o Poder do NotebookLM" — DIO.*
