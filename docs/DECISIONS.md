# Decisões — RotaZFM

Status: registro inicial  
Data de abertura: 09/11/2026

## DEC-001 — Nome oficial

- Data: 09/11/2026
- Decisão: usar **RotaZFM** em toda comunicação, código, repositório e pitch.
- Motivo: o `Plan.md` identifica inconsistências anteriores entre RotaZFM, RouteZFM e ZFMroute.
- Alternativas descartadas: RouteZFM, ZFMroute.
- Responsável: time.

## DEC-002 — IAV determinístico

- Data: 09/11/2026
- Decisão: o IAV é calculado por regras determinísticas; nenhum LLM calcula score ou escolhe rota.
- Motivo: explicabilidade, reprodutibilidade e menor risco de alucinação.
- Alternativas descartadas: scoring por LLM como núcleo.
- Responsável: backend.

## DEC-003 — Custo separado da viabilidade

- Data: 09/11/2026
- Decisão: custo é camada estimada e secundária, usada apenas para ordenar rotas já viáveis.
- Motivo: evitar misturar viabilidade física/regulatória com estimativa frágil de preço.
- Alternativas descartadas: menor custo total como parte do IAV.
- Responsável: produto/backend.

## DEC-004 — AIHOT como referência, não como base confirmada

- Data: 09/11/2026
- Decisão: tratar AIHOT.news como referência estrutural; adoção como base exige spike de até 2 dias.
- Motivo: AIHOT é um framework de curadoria de notícias com LLM, clustering e reports; o RotaZFM precisa de motor determinístico e dados públicos estruturados.
- Alternativas descartadas: importar AIHOT inteiro sem validação.
- Responsável: backend.

## DEC-005 — Justificativa determinística primeiro

- Data: 09/11/2026
- Decisão: começar com justificativa por template determinístico a partir do JSON do IAV.
- Motivo: elimina dependência de LLM no caminho crítico e reduz risco de alucinação.
- Alternativas descartadas: LLM obrigatório na justificativa.
- Responsável: backend.

## DEC-006 — Distância/duração base via distbrasil

- Data: 09/11/2026
- Decisão: usar `distbrasil`/Zenodo como matriz estática de distância e duração para os pares do MVP.
- Motivo: cobre o baseline necessário sem subir OSRM próprio.
- Alternativas descartadas: OSRM próprio no caminho crítico.
- Responsável: dados.

## DEC-007 — Routekit como seed/cache, não por request

- Data: 09/11/2026
- Decisão: usar `routekit-mcp` apenas para gerar/cachear geometria, ETA e alternativas das duas rotas, se o spike validar.
- Motivo: a API gratuita tem limite de 50 chamadas/mês e o MVP não pode depender de chamada externa por request.
- Alternativas descartadas: routekit como dependência runtime do dashboard/MCP.
- Responsável: dados/backend.

## DEC-008 — INMET direto como fonte primária de chuva

- Data: 09/11/2026
- Decisão: usar API direta do INMET por geocódigo para previsão de chuva; `brazil_weather_data` fica como fallback/referência, não como primária.
- Motivo: a necessidade do MVP é previsão municipal de 4 dias; o serviço hospedado parece focado em estações e SQL.
- Alternativas descartadas: `brazil_weather_data` como fonte principal.
- Responsável: dados.

## DEC-009 — PRF fora do MVP

- Data: 09/11/2026
- Decisão: não usar `tidyprf`/`prf-api` como dimensão do IAV no MVP.
- Motivo: acidentes e infrações não são uma das cinco dimensões e aumentariam escopo e validação.
- Alternativas descartadas: adicionar risco PRF como sexta dimensão ou substituir qualidade da via.
- Responsável: produto/dados.

## DEC-010 — Orquestração simples com PostgreSQL

- Data: 09/11/2026
- Decisão: usar worker/cron simples e PostgreSQL com `raw_payloads` + `route_condition_facts`; não usar Airflow/Prefect nem DuckDB como banco principal no MVP.
- Motivo: poucas fontes e um fluxo determinístico não justificam plataforma pesada de orquestração.
- Alternativas descartadas: Airflow, Prefect, DuckDB runtime, múltiplos bancos.
- Responsável: backend.

## DEC-011 — ANA como fallback futuro

- Data: 09/11/2026
- Decisão: pedir autorização da ANA HidroWebService no Dia 1, mas manter SACE como fonte primária de nível do rio.
- Motivo: ANA pode ajudar fallback e validação histórica, mas depende de aprovação externa e token.
- Alternativas descartadas: bloquear MVP esperando ANA ou usar ANA sem autorização.
- Responsável: dados.

## DEC-012 — Firecrawl seletivo

- Data: 09/11/2026
- Decisão: usar Firecrawl para SACE e portarias DNIT quando HTTP direto for frágil ou quando schema de extração ajudar; manter HTTP direto como fallback.
- Motivo: fontes não estruturadas/PDF se beneficiam de extração com schema; API/parsing direto reduz custo e dependência.
- Alternativas descartadas: Firecrawl para todas as fontes ou scraping manual sem schema.
- Responsável: dados.

## Pendências que afetam decisões

- Confirmar regulamento do Hack Brasil 2026.
- Confirmar se AIHOT será base após spike.
- Confirmar rota Porto Velho–São Paulo e distância real.
- Confirmar cobertura do `distbrasil` para os pares necessários.
- Validar spike do `routekit-mcp` para as duas rotas.
- Confirmar payload/cobertura da API direta do INMET.
- Confirmar disponibilidade de série SACE 2023 e contagens DNIT.
- Definir caso retrospectivo e “resultado ruim”.
