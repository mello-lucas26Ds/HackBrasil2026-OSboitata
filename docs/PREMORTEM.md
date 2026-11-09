# Pré-mortem — RotaZFM

Status: rascunho bruto de planejamento  
Data: 09/11/2026  
Fonte: `Plan.md` versão 3

## Veredito

O RotaZFM é **tecnicamente viável**, mas **condicionalmente viável como projeto de hackathon**.

A ideia tem uma tese forte: apoio à decisão de escoamento com dados públicos, explicabilidade por dimensão e exposição via MCP. O maior risco não é escrever código; é fechar, nos primeiros dias, três incógnitas:

1. O problema é real o suficiente para o júri?
2. As fontes públicas entregam dados suficientes, atuais e históricos?
3. O escopo fica restrito ao fluxo determinístico essencial?

Se essas três respostas forem “sim” ou tiverem fallback honesto, o projeto deve seguir. Se forem “não”, o projeto ainda pode existir, mas precisa reduzir a promessa: de “plataforma que resolve a decisão logística” para “protótipo de apoio à decisão com evidências públicas e limites declarados”.

## Premissa do pré-mortem

Suponha que, em 28/11/2026, o projeto foi submetido e fracassou. O fracasso mais provável não veio de um bug isolado. Veio de uma combinação de:

- validação de problema ausente ou tardia;
- fonte pública instável ou sem histórico para o caso retrospectivo;
- escopo inchado por causa do AIHOT, dashboard, MCP, custo, relatório e vídeo ao mesmo tempo;
- rota rodoviária mal definida ou não verificada;
- demo dependente de internet/fontes externas;
- pitch com número ilustrativo tratado como resultado.

## Falhas mais prováveis

| # | Falha | Probabilidade | Impacto | Sinal precoce | Mitigação |
|---|---|---:|---:|---|---|
| 1 | Problema não validado com gestor do PIM | Alta | Alto | Ninguém confirma que a decisão de rota é manual ou difícil | Fazer 3 entrevistas rápidas nos Dias 1–5; se impossível, buscar evidência pública e declarar hipótese no pitch |
| 2 | SACE/INMET/DNIT não entregam dado utilizável | Média/Alta | Alto | Scripts de coleta falham ou retornam HTML/erro | Definir fallback por fonte; dado estático com data; fixture local para demo |
| 3 | Caso retrospectivo sem série histórica | Média/Alta | Alto | Não há dado diário do SACE para 2023 ou contagens DNIT úteis | Trocar caso antes de investir; documentar bloqueio; usar fonte pública alternativa |
| 4 | Rota Manaus–BR-319–Porto Velho–São Paulo mal definida | Média | Alto | Distância, trechos, portarias e transições não fecham | Validar com mapa oficial; modelar por trechos; se não fechar, reduzir para corredor viável |
| 5 | Escopo cresce além dos Must | Alta | Alto | Entram custo, 3D, Resend, WorkOS, relatórios extras | Regra dura: nada que não seja Must entra antes do checkpoint Dia 13 |
| 6 | AIHOT consome mais de 2 dias | Média/Alta | Médio/Alto | Adaptação exige remover LLM, clustering, notícias, admin e prompts | Usar AIHOT como referência estrutural; se spike não provar reuso, seguir estrutura própria simples |
| 7 | MCP na Cloudflare falha por auth, cold start ou compatibilidade | Média | Alto | Endpoint não responde ou cliente MCP não conecta | Fallback local com túnel; endpoint HTTP simples; vídeo com plano B gravado |
| 8 | IAV parece preciso demais ou arbitrário | Média | Médio | Score com muitas casas, sem dados faltantes, pesos sem contexto | Mostrar dimensões, fontes, datas, IAV parcial e teste de sensibilidade |
| 9 | Equipe integra tarde | Média | Alto | Cada um tem peça pronta, mas fluxo não fecha | Vertical slice desde o Dia 3; demo interno a cada fase; um dono por integração |
| 10 | Pitch usa número ilustrativo como fato | Média | Alto | R$ 180, R$ 20 mil, 10% de gain-share aparecem sem fonte | Bloquear números ilustrativos; todo valor com fonte, data e rótulo “estimado” |
| 11 | Regulamento de patrocinadores é interpretado errado | Média | Médio | Cloudflare/Firecrawl/Resend não contam como esperado | Confirmar regulamento no Dia 1; preparar uso real mínimo de pelo menos duas ferramentas |
| 12 | Vídeo é deixado para o fim | Média | Alto | Roteiro só aparece no Dia 18 | Roteiro e plano B de demo desde o Dia 13; gravar blocos conforme funcionalidades ficam prontas |

## Situação no dia 09/11/2026

Considerando `Dia 1 = 05/11/2026`, hoje é aproximadamente o **Dia 5**. Isso significa que a janela de validação inicial está acabando.

As decisões abaixo precisam ser fechadas imediatamente:

- nome oficial: **RotaZFM**;
- confirmação do regulamento, datas e patrocinadores;
- rota Porto Velho–São Paulo verificada;
- existência de série SACE 2023 e contagens DNIT;
- caso retrospectivo escolhido ou descartado;
- definição escrita de “resultado ruim”;
- 3 entrevistas ou substituto honesto;
- decisão AIHOT: usar como base, referência ou abandonar.

## Gates de decisão

### Gate 0 — Hoje/Dia 5

Bloqueia o projeto se falhar:

- Não sabemos o que a plataforma exige.
- Não temos dono para validação.
- Não sabemos se existe dado histórico para o caso retrospectivo.

Ação:

- Confirmar regulamento.
- Marcar entrevistas.
- Pedir ao responsável por dados para testar SACE, INMET e DNIT hoje.
- Congelar nomes e rotas.

### Gate 1 — Dia 6/7

Bloqueia a base técnica se falhar:

- Pelo menos três fontes gravam dado bruto ou entram em fallback aprovado.
- Banco de dados e estrutura de dados brutos existem.
- Primeira versão do IAV roda em script, mesmo parcial.

### Gate 2 — Dia 8/9

Bloqueia o produto se falhar:

- MCP local responde com schema válido.
- Auth/API Key retorna 401 sem chave.
- Motor calcula IAV das duas rotas ou declara IAV parcial com dimensões faltantes.
- Dashboard ainda não é prioridade se MCP não estiver no ar.

### Gate 3 — Dia 13

Checkpoint de ponta a ponta:

- Usuário informa origem, destino e data.
- Sistema mostra IAV, dimensões, fontes, datas e alertas de qualidade.
- MCP responde para cliente real.
- Se falhar, cortar BR-319 e manter rota fluvial.

### Gate 4 — Dia 17

Validação final:

- Caso retrospectivo testado com dados reais ou substituído.
- 10 chamadas simultâneas sem erro.
- Sensibilidade dos pesos executada.
- Pitch sem números ilustrativos.

## Recomendação sobre AIHOT.news

O AIHOT.news é um projeto open source válido como **referência estrutural**, mas **não recomendo adotá-lo como base** sem um spike de no máximo 2 dias.

Motivos:

- Ele é um motor de curadoria de notícias com LLM, clustering, eventos, relatórios e prompts de indústria.
- O RotaZFM precisa de motor determinístico, dados públicos estruturados, grafo de rotas e ferramenta MCP de consulta.
- Adotar AIHOT pode trazer complexidade desnecessária: modelos, orçamento de chamadas, fontes RSS/web, eventos, ranking, relatórios diários/semanais/mensais, admin e prompts.
- O próprio plano já prevê remover scoring por LLM e clustering; se for preciso remover muito, o reuso perde valor.

O que aproveitar do AIHOT:

- separação `apps/api`, `apps/worker`, `apps/web`;
- PostgreSQL + fila de tarefas;
- padrão de módulo isolado;
- camada pública única;
- MCP já implementado como referência;
- Docker Compose para subir ambiente;
- disciplina de testes e documentação operacional.

O que não aproveitar:

- seleção editorial por LLM;
- clustering de notícias;
- prompts de conteúdo;
- relatórios de notícias;
- marca e identidade visual;
- fontes de demonstração de IA.

## Como proceder com o produto

Prossiga, mas mude a ordem mental: primeiro **provar os dados e a decisão**, depois **escalar a interface**.

### Fase A — Provar a tese

Entregar primeiro um script que responde:

> “Para a data X, a rota fluvial e/ou rodoviária têm condições físicas e regulatórias para operar?”

Mesmo que a resposta seja “IAV parcial: 3 de 5 dimensões”. Isso vale mais do que uma arquitetura bonita.

### Fase B — Fechar o fluxo mínimo

Fluxo mínimo do MVP:

```text
fontes públicas → dado bruto → normalização → IAV determinístico → API/MCP → dashboard simples
```

Não incluir nesta fase:

- LLM;
- custo real;
- 3D;
- múltiplas rotas além das duas do MVP;
- WorkOS;
- cobrança;
- alerta por e-mail;
- otimização avançada de rota.

### Fase C — Provar para humanos e agentes

O mesmo resultado precisa aparecer em duas superfícies:

- dashboard para gestor;
- MCP para agente.

A justificativa deve ser determinística: números, fontes, datas, dimensões faltantes e regras de corte.

## Sinais de pivotar

Pivotar não significa abandonar. Significa reduzir a promessa.

- Se não houver entrevistas: chamar o produto de **protótipo de apoio à decisão** e validar com caso retrospectivo + fontes públicas.
- Se SACE 2023 não existir: escolher outro evento com dado disponível ou validar só com dado atual e documentar limitação.
- Se BR-319 não for verificável: reduzir o MVP para Manaus–Belém fluvial e Manaus–Porto Velho–São Paulo rodoviário, com trechos explícitos.
- Se MCP na Cloudflare falhar: manter MCP local + túnel e documentar fallback; não perder o dashboard por causa disso.
- Se AIHOT atrasar: abandonar base e construir estrutura própria mínima.

## Atualização com avaliação de fontes

Em 09/11/2026, as fontes e ferramentas foram avaliadas em `docs/DATA_SOURCES_EVALUATION.md`. A decisão é manter o MVP pequeno:

- `distbrasil` cobre distância/duração base;
- `routekit-mcp` entra só como seed/cache, não por request;
- INMET direto é a fonte primária de chuva;
- `brazil_weather_data` fica como fallback/referência;
- PRF sai do MVP;
- ANA fica como fallback futuro após autorização;
- Airflow/Prefect e DuckDB não entram no caminho crítico.

Isso reduz o risco de dependências, mas não elimina a necessidade de provar que SACE, INMET e DNIT entregam dados suficientes para o IAV.

## Conclusão

O plano está bem detalhado, mas o detalhe não elimina risco. O projeto é viável se o time aceitar cortes disciplinares e tratar validação de dados/problema como entregável dos Dias 1–5, não como enfeite do pitch.

O maior erro possível seria começar pelo framework, pelo dashboard ou pelo vídeo antes de provar que as cinco dimensões têm dado suficiente para sustentar uma recomendação explicável.
