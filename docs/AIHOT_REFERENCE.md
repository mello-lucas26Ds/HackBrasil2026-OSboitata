# Referência — AIHOT.news / KKKKhazix/AIHOT

Status: rascunho bruto de pesquisa  
Data: 09/11/2026  
Repositório: https://github.com/KKKKhazix/AIHOT  
Commit inspecionado: `ad3e934d376e4399ce350d981b4b5eb7a61f860e`

## O que é

AIHOT é um framework open source, licença MIT, para criar sites de tópicos quentes/curadoria por indústria. Ele coleta fontes, filtra com LLM, pontua conteúdos, agrupa eventos, calcula popularidade e publica newsletters/digest.

Stack observada:

- Node.js 24
- TypeScript
- npm workspaces
- Fastify API
- worker com pg-boss
- React Router SSR web
- PostgreSQL 17
- Docker Compose
- MCP e API pública

## Estrutura observada

Arquitetura em três processos:

| Processo | Caminho | Papel |
|---|---|---|
| api | `apps/api/` | Fastify: site API, public API, RSS, MCP, admin, imagens, share images |
| worker | `apps/worker/` | pg-boss: coleta, modelos, agrupamento, heat, newsletters, alertas, limpeza |
| web | `apps/web/` | React Router SSR; só fala HTTP com api |

Pacotes e áreas relevantes:

| Caminho | Conteúdo |
|---|---|
| `packages/backend/src/sources/` | leitores de fontes RSS/web/JSON/X/WeChat/external |
| `packages/backend/src/content/` | ingestão, deduplicação, extração/limpeza de conteúdo |
| `packages/backend/src/editorial/` | análise editorial: prefilter, scoring, títulos, resumos, estrutura |
| `packages/backend/src/events/` | agrupamento de eventos, heat, story digest |
| `packages/backend/src/publication/` | camada única de leitura pública |
| `packages/backend/src/reports/` | daily/weekly/monthly |
| `packages/backend/src/providers/` | providers pagos, receipts, budget circuit breaker |
| `apps/web/app/routes/` | uma rota por página |
| `site/` | identidade do site, modelos, marca, páginas, arquivos públicos |
| `industry/` | taxonomia, tópicos, fontes, prompts, thresholds |
| `modules/` | funcionalidades específicas do site como módulos isolados |
| `database/migrations/` | migrations versionadas |

## Pontos fortes para referência

- Separação clara entre API, worker e web.
- Banco PostgreSQL + fila de tarefas.
- Camada pública única para web/RSS/API/MCP.
- MCP já aparece como superfície pública.
- Módulos isolados permitem adicionar funcionalidade sem mexer no núcleo.
- Docker Compose facilita subir ambiente local.
- Regras de segurança operacional e testes são explícitas.

## Incompatibilidades com RotaZFM

O AIHOT resolve um problema diferente: curadoria e escrita de notícias com LLM. O RotaZFM precisa de:

- coleta de dados públicos estruturados;
- normalização e validação de séries temporais;
- motor determinístico de viabilidade;
- grafo/segmentos de rota;
- cálculo de IAV com dimensões, fontes, datas e guardrails;
- MCP de consulta operacional;
- dashboard de decisão.

Portanto, não devemos copiar prompts, scoring editorial, clustering, reports de notícias ou fluxo de conteúdo. O próprio `Plan.md` já determina que LLM não calcula score nem escolhe rota.

## Veredito

**AIHOT é válido como referência de estrutura, não como base recomendada.**

Recomendação:

1. Usar AIHOT como mapa mental de arquitetura.
2. Fazer um spike de até 2 dias somente se o time quiser testar reuso real.
3. Se o spike mostrar que é preciso remover scoring LLM, clustering, reports e prompts para chegar ao RotaZFM, abandonar a base e criar estrutura própria mínima.
4. Se for usado, congelar commit e não depender de atualização upstream durante o hackathon.

## Estrutura mínima sugerida para RotaZFM

Se não usarmos AIHOT como base, o repositório pode seguir uma versão simplificada:

```text
apps/
  api/          # Fastify ou equivalente: REST + MCP HTTP
  worker/       # cron/jobs de coleta e recálculo
  web/          # dashboard Next.js
packages/
  domain/       # regras do IAV, rotas, validações
  data/         # modelos, queries, fixtures
  collectors/   # SACE, INMET, DNIT, portarias, qualidade via
infra/
  docker-compose.yml
  migrations/
docs/
tests/
```

Essa estrutura é menor e mais alinhada ao MVP do que importar o framework inteiro.
