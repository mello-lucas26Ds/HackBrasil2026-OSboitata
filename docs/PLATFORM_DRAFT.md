# Rascunho bruto para plataforma — RotaZFM

Status: rascunho para submissão/ideia  
Data: 09/11/2026  
Fonte: `Plan.md` v3  
Observação: manter `[a validar]` até confirmação em fonte primária. Não apresentar hipóteses como fatos.

## Nome do projeto

**RotaZFM**

Padronizar em todo lugar: RotaZFM. Não usar RouteZFM ou ZFMroute.

## Trilha

Logística & Automação — Hack Brasil 2026

## Frase de posicionamento

O RotaZFM é uma plataforma de logística e automação que avalia a viabilidade do escoamento do Polo Industrial de Manaus com evidências explicáveis, acessível a gestores e a agentes de IA.

## Problema

Empresas do Polo Industrial de Manaus precisam escoar carga para o Sudeste ou Norte e escolhem entre rotas com impactos diferentes em prazo, custo e risco de atraso.

Hoje, segundo hipótese do time `[a validar com entrevistas]`, essa decisão se apoia em experiência, transportadoras e consultas pontuais a órgãos públicos.

Os dados existem, mas estão espalhados: nível do rio, previsão de chuva, tráfego, qualidade da via e restrições de peso. Nenhuma fonte isolada responde: “essa rota vai funcionar na data do embarque?”.

## Solução

O RotaZFM recebe origem, destino e data de embarque e avalia se as rotas candidatas têm condições físicas e regulatórias de operar.

O núcleo é o **IAV — Índice de Antecipação de Viabilidade**, um score determinístico de 0 a 100 calculado sobre cinco dimensões:

1. Nível do rio — 30%
2. Previsão de chuva — 25%
3. Volume de tráfego — 20%
4. Qualidade da via — 15%
5. Restrição de peso — 10%

Cada resultado mostra:

- score por dimensão;
- fonte de cada dado;
- data de coleta;
- alertas de qualidade do dado;
- dimensões ausentes ou parciais;
- regra de corte para inviabilidade.

## Para quem

- Gestor logístico do PIM: decide rota com antecedência.
- Agente de IA de uma empresa: consulta viabilidade por ferramenta MCP.
- Operador/transportadora: confirma se a rota é operável para uma carga.

## Como funciona

1. Coleta dados públicos de fontes oficiais ou fontes derivadas documentadas.
2. Grava dado bruto antes da normalização.
3. Normaliza, valida frescor, faixa e variações atípicas.
4. Motor determinístico calcula IAV por rota.
5. Restrição de peso excedida torna a rota inviável, independente do score.
6. Dashboard mostra rotas, dimensões, fontes e limitações.
7. Servidor MCP expõe `consultar_viabilidade` para agentes de IA.

## Diferenciais

- Não é caixa-preta: cada decisão mostra dimensões, fontes e datas.
- Distingue ausência de dado de aprovação: score parcial é declarado.
- Separa viabilidade de custo: custo é camada estimada e secundária.
- Pensado para agentes: MCP com schema rígido, API key e resposta estruturada.
- Motor determinístico: LLM não calcula score nem escolhe rota.

## MVP

### Must

- Coleta de nível do rio.
- Coleta de previsão de chuva.
- Coleta de tráfego.
- Qualidade da via como dado estático com fonte.
- Restrição de peso/PBTC por trecho.
- Motor de viabilidade para duas rotas.
- IAV 0–100 com cinco dimensões, fontes e datas.
- MCP com `consultar_viabilidade`.
- API key obrigatória; 401 sem chave.
- Dashboard com origem, destino, data, mapa, IAV e justificativa.

### Should, se houver tempo

- Camada de custo estimado com premissas públicas e faixa de valores.
- Relatório de caso retrospectivo.

### Fora do MVP

- WorkOS.
- Cobrança.
- Visualização 3D.
- Outras rotas da ZFM.
- LLM no cálculo.
- Alerta por e-mail.

## Validação prevista

- Três entrevistas com gestores do PIM `[a validar]`.
- Caso retrospectivo com fonte pública verificável `[a validar]`.
- Definição escrita de “resultado ruim” antes de analisar dados.
- Teste de sensibilidade dos pesos.
- Teste de segurança do MCP sem chave, com data inválida e entrada maliciosa.

## Limites que serão declarados

O RotaZFM não prevê:

- disponibilidade de caminhão, motorista ou combustível;
- greves, bloqueios políticos ou crises de abastecimento;
- atraso aduaneiro ou documental;
- probabilidade calibrada estatisticamente;
- custo real de frete;
- decisão final do gestor.

## Stack prevista

- Backend: Node.js/TypeScript ou estrutura equivalente simples.
- Banco: PostgreSQL, com `raw_payloads` e `route_condition_facts` como contrato central.
- Fila/jobs: worker agendado simples; sem Airflow/Prefect no MVP.
- Dashboard: Next.js + MapLibre GL ou Leaflet.
- MCP: Cloudflare Workers + McpAgent, com fallback local.
- Auth MCP: API Key.
- Coleta: HTTP direto e/ou Firecrawl para fontes não estruturadas; fallbacks documentados.
- Fontes: SACE, INMET direto, DNIT/parseforge, portarias DNIT, `distbrasil`/Zenodo e índices estáticos de qualidade da via.
- Rotas: distância/duração base via `distbrasil`; geometria/ETA via `routekit-mcp` somente como seed/cache, não por request.
- Orquestração inicial: estrutura própria simples; AIHOT apenas como referência se spike aprovar.

## Riscos a declarar

- Fontes públicas podem estar indisponíveis ou defasadas.
- Pesos do IAV são decisão de produto, não calibração estatística.
- Dados de qualidade da via podem estar desatualizados.
- Caso retrospectivo depende de existência de série histórica.
- Problema ainda precisa ser validado com usuários reais.

## Texto curto para formulário

O RotaZFM ajuda empresas do Polo Industrial de Manaus a decidir o escoamento de cargas entre rotas fluvial e rodoviária. Ele cruza dados públicos de nível do rio, chuva, tráfego, qualidade da via e restrição de peso para gerar um índice explicável de viabilidade, disponível em dashboard e via MCP para agentes de IA. O sistema declara fontes, datas e dados faltantes, e separa viabilidade de custo estimado.
