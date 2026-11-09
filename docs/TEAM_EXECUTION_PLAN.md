# Plano de execução e divisão da equipe — RotaZFM

Status: rascunho bruto v2  
Data: 09/11/2026  
Fontes: `Plan.md` v3 + `PREMORTEM.md` + `DATA_SOURCES_EVALUATION.md`  
Dia 1 = 05/11/2026; hoje = aproximadamente Dia 5.

## Objetivo do plano

Sair do papel com uma estrutura pequena de tomada de decisão funcionando de ponta a ponta:

```text
fontes públicas → raw_payloads → route_condition_facts → IAV → API/MCP → dashboard → pitch
```

Regra central: **Must antes de tudo**. Nada de Could/Won’t entra antes do checkpoint do Dia 13. O projeto não deve virar um superprojeto de ingestão; deve virar um motor de decisão pequeno, explicável e adaptável.

## Princípio de arquitetura

O MVP não precisa de Airflow, Prefect, OSRM próprio, banco analítico separado ou motor de rotas completo. Precisa de:

- um schema de fato comum;
- coletores pequenos com fallback;
- um motor determinístico;
- uma API/MCP;
- um dashboard simples.

A evolução para outros estados e rotas futuras deve acontecer por **configuração de rotas/fontes**, não por reescrita do motor.

## Papéis sugeridos

São 4 papéis para 4 pessoas. Os nomes ficam em aberto para o time definir.

| Papel | Responsável sugerido | Escopo |
|---|---|---|
| Produto, validação e pitch | Líder/time lead | Regulamento, entrevistas, caso retrospectivo, plataforma, roteiro, vídeo, decisões de corte |
| Dados e fontes | Integrante de dados | SACE, INMET, DNIT, portarias, distbrasil, routekit seed, qualidade da via, fixtures, validação histórica |
| Motor, API e MCP | Integrante backend | Modelo de dados, `route_condition_facts`, IAV, API, MCP, auth, rate limit, testes de contrato |
| Frontend, demo e E2E | Integrante frontend | Dashboard, mapa, justificativa, cliente MCP de demonstração, Playwright, assets visuais |

Se alguém acumular dois papéis, prioridade absoluta:

1. Dados/fontes;
2. Motor/API/MCP;
3. Produto/validação;
4. Frontend/demo.

Frontend só começa quando o contrato do IAV estiver estável.

## Arquitetura de dados mínima

### Tabelas mínimas

| Tabela | Conteúdo |
|---|---|
| `routes` | Corredores candidatos: id, nome, modo, origem, destino, status |
| `route_segments` | Trechos: BR, km inicial/final, município, fonte de restrição |
| `route_geometry` | GeoJSON/polyline opcional, versionado |
| `raw_payloads` | Resposta bruta, origem, data/hora de coleta, hash |
| `route_condition_facts` | Fato normalizado: rota, dimensão, valor, unidade, data de referência, fonte, qualidade |
| `iav_calculations` | Resultado do IAV: dimensões, fontes, datas, alertas, flags |
| `cost_assumptions` | Premissas de custo estimado, somente Should |

### Contrato central

`route_condition_facts` é a fronteira entre coleta, motor e interface. Todo coletor deve gravar primeiro em `raw_payloads` e depois publicar fatos normalizados com qualidade explícita.

Exemplo de fato:

```json
{
  "route_id": "fluvial-manaus-belem",
  "dimension": "river_level",
  "value": 12.4,
  "unit": "m",
  "data_reference_at": "2026-11-09T12:00:00-04:00",
  "collected_at": "2026-11-09T15:10:00-04:00",
  "source": "SACE",
  "source_url": "...",
  "quality": {
    "freshness_hours": 3.2,
    "stale": false,
    "outlier": false
  }
}
```

## Decisões de fontes já tomadas

Detalhamento completo em `docs/DATA_SOURCES_EVALUATION.md`.

| Dimensão/função | Fonte primária | Fallback | Cortado do MVP |
|---|---|---|---|
| Nível do rio | SACE via Firecrawl/HTTP | último valor válido; ANA futura | ANA como primária |
| Chuva | API direta INMET | scraping INMET via Firecrawl | `brazil_weather_data` como primária |
| Tráfego | DNIT/parseforge | tabela estática datada | PRF como dimensão |
| Qualidade da via | índice estático `precviasBR`/`safety-road-mapping` | valor fixo documentado | qualidade dinâmica |
| Restrição de peso | portarias DNIT via Firecrawl/HTTP | tabela estática “não verificada” | assumir PBTC sem fonte |
| Distância/duração | `distbrasil`/Zenodo | OSRM próprio só se necessário | OSRM no caminho crítico |
| Geometria/ETA | `routekit-mcp` seed/cache | geometria simplificada | routekit por request |
| Orquestração | cron/worker simples | — | Airflow/Prefect |

## Fases e entregas

### Fase 0 — Fechar decisões e validação inicial

Período: hoje até 09/11 ou imediatamente após.

Entregas:

- Regulamento confirmado: datas, patrocinadores, comprovação de uso.
- Nome padronizado: RotaZFM.
- Rota Porto Velho–São Paulo verificada ou MVP reduzido.
- Teste de acesso a SACE, INMET e DNIT.
- Teste de `distbrasil`: confirmar se a matriz cobre os pares necessários.
- Spike de `routekit-mcp`: validar se gera as duas rotas com waypoints; se sim, salvar saída estática.
- Pedido de autorização ANA enviado, sem bloquear MVP.
- Decisão AIHOT: base, referência ou descarte.
- Três entrevistas ou substituto documentado.
- Caso retrospectivo escolhido ou descartado.
- Definição escrita de “resultado ruim”.

Dono principal: Produto/validação + Dados.

Critério de saída:

- Sabemos quais fontes são viáveis.
- Sabemos qual caso retrospectivo usar.
- Sabemos o que pode ser afirmado no pitch.
- Sabemos se routekit entra como seed estático ou é descartado.

### Fase 1 — Banco, rotas estáticas e primeiros coletores

Período: Dias 6–7 / 10–11/11.

Entregas:

- Repositório com estrutura mínima.
- PostgreSQL ou banco local equivalente.
- Tabelas `raw_payloads`, `route_condition_facts`, `routes`, `route_segments`.
- Configuração estática das duas rotas e segmentos.
- Importação da matriz `distbrasil` para distância/duração base.
- Scripts de coleta para SACE, INMET e DNIT.
- Primeira carga de qualidade da via e restrição de peso.
- Fallbacks mínimos documentados.

Dono principal: Dados + Backend.

Critério de saída:

- Pelo menos três fontes gravam dado bruto.
- Cada registro tem origem, timestamp, data de coleta e payload bruto.
- Cada fato normalizado tem fonte e qualidade.
- Falha de fonte não derruba o sistema; gera alerta/último valor válido.

### Fase 2 — Normalizador e primeiro IAV

Período: Dias 7–9 / 11–13/11.

Entregas:

- Normalizador por dimensão para `route_condition_facts`.
- Cálculo do IAV 0–100.
- Regras de frescor, variação atípica e dados faltantes.
- IAV parcial quando faltar dimensão.
- Restrição de peso como corte duro.
- Script CLI imprime IAV das duas rotas com fontes e datas.

Dono principal: Backend + Dados.

Critério de saída:

- Script equivalente a `npm run iav` imprime resultado determinístico.
- Testes unitários cobrem fórmula, limites, dados ausentes e peso excedido.

### Fase 3 — Motor de rotas e MCP local

Período: Dias 10–13 / 14–17/11.

Entregas:

- Representação dos trechos das duas rotas em `route_segments`.
- Geometria opcional vinda de routekit seed ou simplificada.
- Motor de viabilidade por rota.
- Endpoint/API interna de consulta.
- MCP local com `consultar_viabilidade`.
- API key e resposta 401 sem chave.
- Rate limit implementado, mesmo simples.
- MCP Inspector validando schema.

Dono principal: Backend.

Critério de saída:

- Cliente MCP real ou MCP Inspector recebe IAV e justificativa.
- Entrada inválida é rejeitada.
- Sem chave retorna 401.

### Fase 4 — Deploy MCP Cloudflare

Período: Dias 8–9 do plano original / 12–13/11, se o MCP local estiver pronto.

Entregas:

- Deploy do MCP na Cloudflare.
- API key configurada.
- Endpoint público responde.
- Fallback local com túnel documentado.

Dono principal: Backend + Produto para decisão de go/no-go.

Critério de saída:

- Endpoint público responde em menos de 3s na meta.
- Se falhar, fica o fallback local sem bloquear dashboard.

### Fase 5 — Dashboard

Período: Dias 10–12 do plano original / 14–16/11.

Entregas:

- Tela com origem, destino e data.
- Mapa com duas rotas.
- IAV por rota.
- Dimensões, fontes, datas e alertas de qualidade.
- Estados de loading, erro, IAV parcial e rota inviável.
- Custo estimado somente se ainda for Should e o fluxo estiver estável.

Dono principal: Frontend + Backend para contrato.

Critério de saída:

- Usuário vê resultado em menos de 2s na meta.
- Nenhum score aparece sem fonte/data ou alerta de qualidade.

### Fase 6 — Checkpoint ponta a ponta

Período: Dia 13 do plano original / 17/11.

Entregas:

- Fluxo completo funcionando.
- Teste com 10 chamadas simultâneas.
- Decisão sobre 3D: cortar.
- Decisão sobre BR-319: manter ou cortar conforme checklist.
- Relatório retrospectivo iniciado.

Dono principal: Todos, com Product decidindo cortes.

### Fase 7 — Relatório, testes e vídeo

Período: Dias 14–20 do plano original / 18–24/11.

Entregas:

- Relatório JSON ou PDF simples.
- Caso retrospectivo com premissas declaradas.
- Roteiro de vídeo de 3 minutos.
- Gravação PT e EN.
- Edição.
- Plano B da demo gravado.

Dono principal: Produto/Pitch + Frontend para evidências visuais.

### Fase 8 — Deploy final, docs e submissão

Período: Dias 21–24 do plano original / 25–28/11.

Entregas:

- Deploy final acessível fora da rede local.
- README com arquitetura, como rodar, fontes e limites.
- Submissão na plataforma até 28/11.
- Buffer para imprevistos.

## Rotinas do time

### Diário

- 10 minutos de alinhamento.
- Cada pessoa responde: o que integrou ontem, o que integra hoje, qual bloqueio.
- Um único quadro de tarefas com colunas: Must, Should, Fora do escopo.

### Decisões

Toda decisão técnica ou de escopo entra em `docs/DECISIONS.md` com:

- data;
- decisão;
- motivo;
- alternativa descartada;
- responsável.

### Regras de corte

- Dia 8 sem MCP: adiar dashboard.
- Dia 13 sem fluxo completo: cortar BR-319 e ficar com rota fluvial.
- Dia 24: nenhuma feature nova.
- Could/Won’t não entra antes do Dia 13.
- LLM não entra no cálculo.
- Airflow/Prefect não entra no MVP.
- PRF não entra como dimensão do IAV.
- Routekit não é chamado por request; entra como seed/cache se aprovado.

## Divisão de validação

| Validação | Dono | Evidência esperada |
|---|---|---|
| Regulamento e prazos | Produto | Captura/screenshot do regulamento e datas |
| Entrevistas PIM | Produto | Notas ou gravação autorizada, com consentimento |
| Caso retrospectivo | Dados | Fonte primária, série de dados, definição de resultado ruim |
| SACE/INMET/DNIT | Dados | Script rodando + fixture + fallback |
| Distbrasil | Dados | Pares necessários extraídos e versionados |
| Routekit seed | Dados/Backend | GeoJSON/ETA salvo ou descarte documentado |
| ANA | Dados | Pedido enviado e prazo estimado |
| Qualidade da via e portarias | Dados | Tabela com fonte e data de referência |
| IAV e fórmulas | Backend | Testes unitários e exemplos calculados |
| MCP/auth/rate limit | Backend | MCP Inspector + teste 401 + teste de rate limit |
| Dashboard/mapa | Frontend | E2E Playwright + screenshots |
| Pitch sem claims inválidos | Produto | Revisão final do `PLATFORM_DRAFT.md` |

## Primeira semana prática

Se hoje é Dia 5, a prioridade é fechar a Fase 0 hoje ou amanhã:

1. Confirmar regulamento.
2. Testar manualmente SACE, INMET e DNIT.
3. Verificar rota Porto Velho–São Paulo.
4. Verificar cobertura do `distbrasil` para os pares necessários.
5. Fazer spike de `routekit-mcp` para gerar/cachear geometria das duas rotas.
6. Marcar entrevistas.
7. Pedir autorização ANA, sem bloquear MVP.
8. Decidir AIHOT em até 2 dias; se não houver prova clara de reuso, usar estrutura própria.
9. Criar repositório com estrutura mínima, schema de fatos e README.

## O que não fazer

- Não começar pelo dashboard.
- Não começar pelo vídeo.
- Não adotar AIHOT inteiro por entusiasmo.
- Não adicionar LLM ao cálculo.
- Não usar Airflow/Prefect no MVP.
- Não usar PRF como dimensão do IAV.
- Não chamar routekit por request sem cache.
- Não usar números ilustrativos como impacto real.
- Não mostrar score sem fonte, data e qualidade do dado.
- Não deixar decisão pendente por mais de 24h.
