<!-- README consolidado (opção A) -->

# fb2pg – Firebird → PostgreSQL DDL Converter

[![Python](https://img.shields.io/badge/Python-3.6%2B-blue.svg)](https://www.python.org/downloads/) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Conversor de DDL Firebird/InterBase para PostgreSQL com heurísticas para sequências, autoincrement, domains e colunas computadas. Gera saída legível e anota o que precisa de revisão manual.

## 🎯 Funcionalidades
- Conversão de tipos (BLOB SUB_TYPE → TEXT/BYTEA; remoção de SEGMENT SIZE / CHARACTER SET / COLLATE / SCALE / SUB_TYPE)
- Sequências: GENERATOR / SET GENERATOR / GEN_ID → CREATE SEQUENCE / setval() / nextval()
- PK numérica simples → criação de sequence + DEFAULT nextval()
- Triggers de autoincrement (NEW.col = GEN_ID(seq,1)) → IDENTITY (trigger comentada)
- Colunas COMPUTED BY → GENERATED ALWAYS AS (...) STORED ou comentário (--computed comment)
- Domains: manter (keep) ou substituir (replace) com propagação de DEFAULT / NOT NULL
- Normalização de terminadores (SET TERM) e remoção de comandos sem equivalente
- RECREATE / CREATE OR ALTER adaptados para PostgreSQL
- Comentários TODO para partes não convertidas (procedures, functions, triggers complexas, exceptions, CHECK de domain em replace)

## 🚀 Instalação
```bash
git clone <repository-url>
cd fb2pg
python fb2pg.py --help
# (Opcional) chmod +x fb2pg.py
```

Pré-requisitos: Python 3.6+ e arquivo .sql exportado do Firebird.

## 📝 Uso
```bash
python fb2pg.py origem.sql [opções]
```
Opções:

| Opção | Descrição | Padrão |
| -o / --output | Arquivo de saída | {input}.pg.sql |
| --computed | generated | generated/comment |
| --domains | keep/replace | keep |
| --verbose | Logs de depuração | false |

Exemplos:
```
python fb2pg.py schema.sql
python fb2pg.py schema.sql --computed comment
```

## 🔄 Conversões Principais
| Firebird | PostgreSQL | Observação |
|----------|------------|-----------|
| BLOB / SUB_TYPE 0 | BYTEA | Binário |
| SET GENERATOR seq TO N | SELECT setval('seq', N) | Ajuste inicial |
| GEN_ID(seq,1) | nextval('seq') | Próximo valor |
| RECREATE TABLE X | DROP TABLE IF EXISTS X; CREATE TABLE X | Sequência segura |
| CREATE OR ALTER VIEW | CREATE OR REPLACE VIEW | Equivalente |
| Trigger autoincrement | IDENTITY + comentário | Heurística |

Removidos/comentados: SET SQL DIALECT, SET NAMES, SET CLIENTLIB, CREATE DATABASE.

## 🧱 Domains
Keep (padrão):
```sql
CREATE DOMAIN DM_STR_50 AS VARCHAR(50);
CREATE TABLE PRODUTO ( ID INTEGER PRIMARY KEY, DESCRICAO DM_STR_50 );
```
Replace:
```sql
-- TODO: DOMAIN DM_STR_50 convertido para uso direto do tipo VARCHAR(50)
-- CREATE DOMAIN DM_STR_50 AS VARCHAR(50);
CREATE TABLE PRODUTO ( ID INTEGER PRIMARY KEY, DESCRICAO VARCHAR(50) );
TOTAL NUMERIC(10,2) COMPUTED BY (PRECO * QTD)
-- PostgreSQL (--computed generated)
TOTAL NUMERIC(10,2) GENERATED ALWAYS AS (PRECO * QTD) STORED
-- PostgreSQL (--computed comment)
TOTAL NUMERIC(10,2) /* TODO: Firebird COMPUTED BY (PRECO * QTD) - revisar */
```

## ⚡ Autoincrement por Trigger
```sql
## � Estrutura do Projeto
```
fb2pg/
├── fb2pg.py
├── README.md
├── test_firebird.sql
└── test_firebird.sql.pg.sql
```

## 🧪 Teste Rápido
```
python fb2pg.py test_firebird.sql --verbose
head -30 test_firebird.sql.pg.sql
```

## ⚠️ Limitações
- Domains com CHECK (replace) → CHECK vira comentário TODO
- Repetições de FKs / índices não deduplicadas
- PK compostas não recebem sequence
- NUMERIC/DECIMAL não reduzido automaticamente para INTEGER/BIGINT
- Procedures / functions / triggers complexas apenas comentadas
- Não migra dados (apenas DDL)

## ✅ Recomendações Pós-Conversão
1. Revisar comentários TODO
2. Deduplicar índices / FKs repetidos
3. Ajustar tipos (UUID, JSONB, etc.)
4. Criar banco/roles manualmente
5. Executar migração de dados (COPY / ETL)

## 📋 Roadmap (Resumo)
- Suporte multi-arquivo
- Deduplicação de FKs/índices
- IDENTITY opcional direto em PK simples
- Mapeamento semântico de domains (CNPJ/CPF etc.)
- Conversão parcial de procedures
- Relatório de conversão / JSON output

## 📄 Licença
MIT (ver `LICENSE`).

## 👤 Autor
Mayron Cachina

## 📞 Suporte
Abra uma issue com: versão Python, SO e trecho reduzido do DDL.

---
**fb2pg** – Migração Firebird → PostgreSQL com clareza e automação.