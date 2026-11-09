# RotaZFM: documento consolidado

Status: versão 3, substitui as anteriores. Esta é a fonte de verdade do projeto. Itens marcados com [a validar] ainda não foram confirmados em fonte primária e não devem aparecer no pitch como fato.

1. Resumo executivo

O RotaZFM é um apoio à decisão de escoamento para empresas do Polo Industrial de Manaus (PIM). Para uma data de embarque, ele avalia se duas rotas têm condições físicas e regulatórias de funcionar: a fluvial (Manaus–Belém) e a rodoviária (Manaus–BR-319–Porto Velho–São Paulo). A avaliação é um índice determinístico, o IAV (Índice de Antecipação de Viabilidade), calculado sobre cinco dimensões de evidência pública. O resultado aparece em um dashboard para gestores e é exposto a agentes de IA por um servidor MCP hospedado na Cloudflare.

Trilha: Logística & Automação (Hack Brasil 2026).

Frase de posicionamento (primeiros 10 segundos do vídeo):

“O RotaZFM é uma plataforma de logística & automação que avalia a viabilidade do escoamento do Polo Industrial de Manaus com evidências explicáveis, acessível a gestores e a agentes de IA.”

Princípio central: o sistema mede viabilidade, não custo. Custo é uma camada separada, estimada, usada só para ordenar rotas já viáveis.

2. Problema e público-alvo
2.1 Problema

Uma empresa do PIM que precisa escoar carga para o Sudeste ou para o Norte escolhe entre duas rotas principais. A escolha afeta prazo, custo e risco de atraso. Hoje, segundo a hipótese do time, ela se apoia em experiência, contato com transportadoras e consultas pontuais a órgãos públicos. [a validar: nenhum gestor do PIM foi entrevistado ainda.]

Os dados relevantes existem, mas estão espalhados: nível do rio (SGB/CPRM, sistema SACE), previsão de chuva (INMET), contagens de tráfego e portarias de restrição de peso (DNIT). Nenhum deles responde sozinho à pergunta “essa rota vai funcionar na data do embarque?”.

2.2 Público-alvo
Persona	Necessidade	Como o RotaZFM ajuda
Gestor logístico do PIM	Decidir a rota com antecedência	Dashboard com IAV, dimensões e fontes
Agente de IA de uma empresa	Consultar viabilidade de forma padronizada	Ferramenta MCP consultar_viabilidade
Operador logístico ou transportadora	Confirmar se aceita uma carga naquela rota	Mesmo IAV, pelo dashboard ou pelo MCP

O usuário da demo é o gestor. O argumento de arquitetura é o agente. O júri avalia os dois.

3. Escopo do MVP (MoSCoW)
ID	Funcionalidade	Prioridade	Critério de aceitação
F1	Coleta de nível do rio (SACE, via Firecrawl ou HTTP)	Must	Último valor do rio Amazonas no banco, com data e estação
F2	Coleta de previsão de chuva (API do INMET)	Must	Previsão de 4 dias para Manaus e para o trecho da BR-319
F3	Coleta de tráfego (scraper do DNIT, parseforge)	Must	Contagens da BR-319 no banco
F4	Índice de qualidade da via (dado estático)	Must	Valor por trecho, com fonte e data de referência
F5	Restrição de peso (portarias do DNIT)	Must	Limite de PBTC vigente por trecho, com data da portaria
F6	Motor de viabilidade para duas rotas	Must	Para cada rota: viável ou não, e dados que sustentam a decisão
F7	Cálculo do IAV com cinco dimensões	Must	Score de 0 a 100, com dimensões, fontes e datas
F8	Servidor MCP na Cloudflare com consultar_viabilidade	Must	Cliente MCP real (Claude Desktop ou Cursor) recebe o IAV e a justificativa
F9	Autenticação por API Key	Must	Chamada sem chave retorna 401
F10	Dashboard com mapa, IAV e justificativa	Must	Usuário informa origem, destino e data e vê as duas rotas
F11	Camada de custo estimado	Should	Ranking por custo entre rotas viáveis, sempre rotulado como estimativa
F12	Relatório de caso retrospectivo	Should	Documento com a validação histórica e as premissas usadas
F13	Alerta por e-mail via Resend	Could	Disparo quando o IAV de uma rota cai abaixo de um limite
F14	Visualização 3D de cubagem	Could	Substituída por diagrama 2D se necessário
F15	WorkOS AuthKit	Won’t	Cortado; citado como próximo passo de segurança
F16	Or-tools-wasm	Won’t	Substituído pelo motor próprio
F17	Cobrança via AbacatePay	Won’t	Só como modelo de negócio no pitch
F18	Rota de Caracaraí e outras rotas da ZFM	Won’t	Só no pitch

Regra de corte: se uma funcionalidade não é Must, ela não entra no repositório antes do checkpoint do Dia 13.

4. O que o IAV faz e o que não faz

Faz: verifica se, com os dados disponíveis na data de embarque, a rota tem condições físicas e regulatórias de funcionar. Indica quais dimensões sustentam a decisão e quais dados estão faltando.

Não faz:

Não mede disponibilidade de caminhão, motorista ou combustível.
Não prevê greve, bloqueio político ou crise de abastecimento.
Não calcula custo.
Não é probabilidade calibrada estatisticamente.
Não substitui a decisão do gestor.
5. O índice IAV
5.1 Dimensões e fontes
Dimensão	Peso	Pergunta que responde	Fonte primária	Fallback
Nível do rio	30%	A calha fluvial está navegável na data?	SGB/CPRM (SACE)	Último valor válido; ANA (HidroWebService) como fonte de reserva, com aprovação pendente
Previsão de chuva	25%	Há risco de atoleiro na BR-319 ou de cheia?	API do INMET	Scraping do site do INMET
Volume de tráfego	20%	A rodovia está com fluxo anormal?	Scraper DNIT (parseforge)	Tabela de referência com última contagem
Qualidade da via	15%	O trecho está em condição precária?	Índice estático (precviasBR ou safety-road-mapping)	Valor fixo documentado
Restrição de peso	10%	O limite de PBTC permite a carga?	Portarias do DNIT	Tabela estática com data de revisão

Assoreamento não tem peso. Fica como indicador de confiança da rota fluvial, processado offline no MVP, com fonte em satélite e carta náutica [a validar: viabilidade do processamento no prazo].

5.2 Fórmula
IAV = 0,30 × nivel + 0,25 × chuva + 0,20 × trafego + 0,15 × qualidade + 0,10 × restricao

Cada score fica entre 0 e 100. Restrição de peso excedida torna a rota inviável, independentemente do IAV. É corte duro, não peso.

5.3 Evolução dos pesos
MVP: pesos fixos, documentados, sem aprendizado.
Próxima versão: pesos configuráveis por tipo de carga (perecível, granel, carga geral), guardados no banco e registrados em changelog.
Futuro: calibração com histórico de decisões e resultados reais. Depende de dados que o MVP não coleta.

Os pesos não mudam durante o hackathon. Qualquer alteração exige registro em ADR.

6. Modos de falha
6.1 Falhas fora do escopo do modelo
Falta de caminhão ou motorista (ex.: greve dos caminhoneiros de 2018, em que a rota estava aberta, mas faltava oferta)
Falta de combustível
Bloqueio político, sanitário ou de segurança
Problemas no porto de destino ou em balsas
Atraso aduaneiro ou documental
Acidente isolado sem efeito sobre as variáveis medidas

Esses fatores afetam a decisão de forma real. Devem aparecer declarados como limites do modelo, no dashboard e no pitch.

6.2 Falhas que o modelo detecta mal
Dado defasado: boletim do SACE publicado com atraso ou de estação que não representa o trecho exato da rota.
Previsão errada: INMET prevê chuva fraca, e chove forte na BR-319.
Fonte fora do ar: scraper quebra e o sistema usa último valor válido, que pode ser antigo.
Pesos arbitrários: 30/25/20/15/10 são decisão de produto, não calibração. Outro analista chegaria a outro resultado.
Dimensões correlacionadas: nível do rio e chuva andam juntos. Tratá-los como independentes superestima a evidência.
Qualidade da via estática: índice de um ano atrás não captura um trecho que cedeu na semana passada.
6.3 Falhas de apresentação
Score com casas decimais, como se fosse medição precisa.
Score alto sem indicar qual dado está faltando.
Valor de último registro exibido como se fosse atual.
6.4 Como o sistema detecta e sinaliza

Regra geral: nunca mostrar um score sem mostrar a qualidade do dado que o gerou. Ausência de dado nunca vira aprovação.

Situação	Ação do sistema	O que aparece
Dado de rio com mais de 24 h	Usa último valor válido	“Dado de rio com X horas de atraso”
Variação do nível acima de 2 m em 1 h	Descarta o valor	“Valor descartado por variação atípica”
Chuva prevista acima de 100 mm/dia	Marca evento crítico e pede revisão humana	“Evento crítico, IAV não atualizado sem revisão”
Dimensão sem dado	Score indisponível, IAV parcial	“IAV parcial: 4 de 5 dimensões”
Score fora de 0 a 100	Não publica	Erro interno
Scraper quebrado	Usa último valor e registra	“Fonte SACE indisponível desde DD/MM”
Restrição de peso não confirmada	Trata como não verificada, não como liberada	“Restrição de peso não verificada”
7. Custo: camada separada e estimada
7.1 Por que separado

O PRD original falava em “menor custo total”. O IAV não mede custo. Misturar os dois contamina a viabilidade com uma estimativa de preço frágil. Por isso, a decisão tem dois passos:

Viabilidade: IAV, com as cinco dimensões.
Ordenação: entre as rotas viáveis, qual é mais barata, com custo estimado.
7.2 Fontes de premissas

Não existe API pública de tarifa de frete por rota. As fontes disponíveis são:

Fonte	Uso	Limitação
Pisos mínimos de frete rodoviário (ANTT)	Linha de base por km e eixo para o trecho rodoviário	Piso regulatório, não preço de mercado
Dissertações da UFAM sobre frete fluvial (Manaus–Porto Velho, Manaus–Belém, 2015 a 2017)	Estrutura de custo e valores de referência para o trecho fluvial	Dados antigos, de uma empresa; precisam de atualização
Cotações reais de usuários ou parceiros	Calibração futura	Não disponível no MVP
7.3 Regras
Tabela premissas_custo, com: modal, custo por t·km, consumo, velocidade média, fonte, data e responsável.
Custo sempre rotulado como “estimado”, com fonte visível.
Apresentado como faixa, não como número único.
No pitch: “custo estimado com premissas públicas, calibrável com cotações reais”. Nunca “custo real”.
8. Arquitetura e stack
8.1 Diagrama
COLETA (worker agendado)
  SACE (Firecrawl ou HTTP) ───┐
  INMET (API) ────────────────┤
  DNIT (parseforge) ──────────┼──▶ normalizador ──▶ PostgreSQL (dado bruto + tratado)
  Portarias DNIT ─────────────┤
  Qualidade da via (estático)─┘
          │
          ▼
MOTOR DE DECISÃO (determinístico)
  Rotas candidatas (Dijkstra/A* sobre grafo simples)
  IAV com cinco dimensões e guardrails
  Camada de custo estimado (só para rotas viáveis)
  Justificativa a partir do JSON calculado (template; LLM opcional)
          │
          ▼
SERVIDOR MCP (Cloudflare Workers + McpAgent)
  consultar_viabilidade(origem, destino, data_embarque)
  Auth: API Key no header Authorization
          │
     ┌────┴─────┐
     ▼          ▼
  Dashboard   Agentes (Claude Desktop, Cursor)
  (Next.js)
8.2 Stack e fallbacks
Componente	Primária	Fallback	Decisão até
Orquestração	AIHOT.news (MIT), como esqueleto de worker, fila, API e MCP	Estrutura própria simples	Dia 3 (5/11 = Dia 1)
Coleta de nível	Firecrawl no SACE	HTTP direto com headers de navegador	Dia 3
Coleta de chuva	API do INMET	Scraping do site	Dia 3
Coleta de tráfego	parseforge/dnit-brazil-traffic-scraper	Tabela de referência	Dia 4
Motor de rotas	Dijkstra/A* próprio	Or-tools (só se o próprio falhar)	Dia 6
MCP	Cloudflare Workers + McpAgent	MCP local com SDK + túnel	Dia 8
Auth MCP	API Key	Nenhum, com risco documentado	Dia 8
Dashboard	Next.js + MapLibre GL	Leaflet	Dia 12
Justificativa	Template determinístico	LLM (ver 8.3)	Dia 10
Relatório	JSON; PDF se houver tempo	Só JSON	Dia 16
8.3 Decisão sobre o LLM

Recomendação: começar com template determinístico. A justificativa é montada a partir dos números calculados, sem LLM. Isso elimina risco de alucinação e de dependência.

O Gemini 2.5 Flash é tecnicamente viável, mas a documentação do Google o marca como acesso restrito a quem já usava o modelo. Para projeto novo, o recomendado é um modelo atual da família. Só adicionar LLM se sobrar tempo, e sempre fora do cálculo.

8.4 Princípios
O motor é determinístico. Nenhum modelo de linguagem calcula score ou escolhe rota.
Frontend conversa só com a API, nunca com o banco.
Todo dado bruto é gravado antes da normalização, para permitir reprocessamento.
9. Repositórios e ferramentas de patrocinadores
9.1 Ferramentas de patrocinadores
Ferramenta	Papel	Peso no projeto
Cloudflare (Workers + McpAgent)	Exposição do MCP	Central
Firecrawl	Coleta do SACE e das portarias	Média, com fallback
Resend	Alerta por e-mail	Baixa, Could
AbacatePay	Cobrança	Nula no MVP
WorkOS	Cortado	Nenhuma

[a validar]: confirmar no regulamento quais ferramentas contam como patrocinadoras e como se comprova o uso.

9.2 Repositórios abertos
Repositório	Uso	Tipo
AIHOT.news (MIT)	Esqueleto de orquestração	Base de código, se a adaptação couber em dois dias
parseforge/dnit-brazil-traffic-scraper	Tráfego da BR-319	Dependência
precviasBR ou safety-road-mapping	Qualidade da via	Dado estático
cargofy/ATLAS	Referência de estrutura de ferramentas MCP	Referência
MCP-Brasil	Referência de exposição de dados públicos via MCP	Referência

ATLAS e MCP-Brasil só contam como “usados” se alguma linha de código for reaproveitada.

9.3 Riscos das dependências
AIHOT é recente e pode mudar a API. Congelar um commit no Dia 2.
Nome e logo do AIHOT não podem ser usados.
O módulo de scoring por LLM e o de clustering devem ser removidos. Se a adaptação consumir mais de dois dias, abandonar o AIHOT sem culpa.
10. Segurança e testes
10.1 Ameaças
Dado falso ou corrompido na coleta.
Chamadas abusivas ao MCP, sem chave ou em volume alto.
Entrada maliciosa nos campos de origem e destino.
Decisão errada por agente que recebe IAV alto para rota inviável.
10.2 Defesas

Ingestão: validação de faixa e de variação temporal; dado bruto com origem, data e hash; quarentena de valor suspeito.

Motor: IAV determinístico; teste unitário com valores conhecidos e casos de borda; saída sempre entre 0 e 100.

MCP: schema de entrada rígido, com rejeição de campos extras; API Key obrigatória (401 sem chave); limite de requisições por chave, implementado e não apenas documentado (meta de 60 por minuto [a validar: número]); nenhum dado pessoal real nas demos.

Saída para agente: IAV, dimensões, fontes, datas de coleta e alertas de qualidade. A responsabilidade pela decisão final é declarada na documentação da ferramenta.

10.3 Testes
Nível	O que testar	Ferramenta
Unitário	Normalização, fórmula, validação de schema	Vitest ou Jest
Integração	Coletor grava no banco; motor lê e calcula; MCP responde com e sem chave	Banco local
Contrato	Saída do MCP segue o schema documentado	MCP Inspector
Ponta a ponta	Dashboard recebe entrada e mostra IAV	Playwright
Diário	Coleta, motor e MCP rodam; IAV dentro de 0 a 100	Script agendado
Sensibilidade	Pesos ±5 pontos; a recomendação muda?	Script
11. Validação
11.1 Caso retrospectivo

Caso principal: seca de 2023 no rio Amazonas. O IAV teria sinalizado risco de navegação na via fluvial com antecedência? Só vale se houver série de nível do SACE com granularidade diária para o período [a validar].

Caso secundário: interdição ou atoleiro da BR-319 com data e local em fonte pública [a validar].

Greve dos caminhoneiros de 2018 não serve como validação do IAV. A rota estava fisicamente aberta; o que faltou foi oferta. Usá-la como teste faria o modelo parecer errado. Deve ser citada como exemplo de falha fora de escopo.

11.2 Regras da validação
Definir “resultado ruim” antes de olhar os dados. Exemplo: interrupção ou atraso acima de X dias, documentado em fonte pública.
Registrar a definição em documento com data.
Se a série de dados não existir, trocar o caso antes de gastar tempo com custo.
11.3 Outras validações
Teste de ataque interno: tentar chamar o MCP sem chave, com data no passado e com texto injetado.
Teste de sensibilidade dos pesos.
Três entrevistas com gestores do PIM nos Dias 1 a 5. Sem elas, o argumento de problema real não se sustenta.
11.4 Custo perdido no caso retrospectivo

Forma honesta: “com as premissas declaradas, o custo evitado por carga de referência ficaria entre R$ A e R$ B”. A faixa é calculada com a tabela de premissas, e as premissas são mostradas. Não é medição.

12. Cronograma

Dia 1 = 5/11/2026. Submissão até 28/11, com margem antes do prazo informado de 30/11 [a validar no regulamento].

Dia	Data	Meta	Critério de aceitação
1	5/11	Decisões e repositório	Repositório público criado após 4/11 às 21h; pesos travados; decisão sobre AIHOT
2	6/11	Banco e primeiros coletores	Três scripts gravam dado bruto (SACE, INMET, DNIT)
3 a 5	7 a 9/11	Normalizador e primeiro IAV	Script imprime IAV das duas rotas com dimensões e fontes
6 a 7	10 a 11/11	Motor de rotas e MCP local	MCP local responde no MCP Inspector
8 a 9	12 a 13/11	Deploy do MCP na Cloudflare, com API Key	Endpoint público responde
10 a 12	14 a 16/11	Dashboard	Mapa com duas rotas e IAV
13	17/11	Checkpoint de ponta a ponta	Fluxo completo; decisão sobre 3D
14 a 16	18 a 20/11	Relatório e polimento	Relatório gerado; estados de erro tratados
17	21/11	Testes de carga e validação retrospectiva	10 chamadas simultâneas sem erro; caso testado
18	22/11	Roteiro do vídeo	Cinco blocos com tempos
19	23/11	Gravação PT e EN	Dois takes completos
20	24/11	Edição	Vídeo de 3 minutos
21	25/11	Deploy final e teste público	Demo acessível fora da rede local
22	26/11	README e documentação	Arquitetura, como rodar, fontes e limites
23 a 24	27 a 28/11	Buffer e submissão	Submissão feita em 28/11

Regras de corte:

Checkpoint do Dia 8 sem MCP no ar: adiar o dashboard.
Checkpoint do Dia 13 sem fluxo completo: cortar a BR-319 e ficar com a rota fluvial.
Dia 24: nenhuma feature nova.
13. KPIs
13.1 Produto
Métrica	Como medir	Meta
Frescor do dado de rio	Idade do último registro válido	Menos de 24 h
Dimensões com dado real	Quantas das cinco têm dado no ciclo	5 de 5
Tempo de resposta do dashboard	Entrada até IAV exibido	Menos de 2 s
Latência do MCP	Chamada de consultar_viabilidade	Menos de 3 s
Disponibilidade na janela de avaliação	Uptime	Acima de 95%
Justificativa rastreável	Decisões com fontes e datas	100%
13.2 Hackathon
Critério	Indicador	Meta
Execução técnica	Fluxo ponta a ponta sem intervenção manual	Dia 13
Implementação	Ferramentas de patrocinadores em uso real	Pelo menos duas, confirmadas no regulamento
Implementação	Repositórios abertos com código reaproveitado	Pelo menos dois
Impacto	Caso retrospectivo com faixa de custo	Sim, com premissas declaradas
Comunicação	Vídeo entregue	Até 24/11
Comunicação	“Logística” nos primeiros 10 segundos	Sim
13.3 Time
Métrica	Meta
Algo novo funcionando de ponta a ponta por dia, a partir do Dia 5	Sim
Peças funcionando isoladas e não integradas	Máximo de uma
Features Could ou Won’t tentadas	Zero
Decisões pendentes há mais de 24 horas	Zero
14. Riscos e crítica do júri
14.1 Riscos priorizados
Risco	Probabilidade	Impacto	Mitigação
Escopo grande para 4 semanas	Alta	Alto	Cortes nos Dias 6, 8 e 13
Fonte principal (SACE) quebra	Média	Alto	Fallback HTTP e último valor válido com sinalização
MCP não funciona na Cloudflare	Média	Alto	Fallback local com túnel
Vídeo tardio ou amador	Média	Alto	Roteiro no Dia 18, gravação no Dia 19
Caso retrospectivo sem dado	Média	Alto	Trocar caso antes de investir
Posicionamento lido como meio ambiente	Média	Médio	Frase de abertura travada
Júri questiona pesos	Alta	Médio	Declarar como decisão de produto, com sensibilidade testada
Regras de patrocinadores mal entendidas	Média	Médio	Confirmar regulamento
14.2 Onde pecamos
Problema não validado com usuário. O argumento de impacto parte de uma hipótese. O júri pode perguntar “quem já pediu isso?”.
Valores de impacto são exemplos, não resultados. R$ 180, R$ 20 mil e 10% de gain-share foram ilustrações da conversa. Se entrarem no pitch como fato, o projeto perde credibilidade no primeiro questionamento.
Probabilidades de final (55%, 70 a 80%) são estimativas minhas. Não devem aparecer no pitch.
Versões conflitantes do material. Pesos, dias do MCP e número de fontes mudaram ao longo das conversas. Este documento é a única fonte de verdade a partir de agora.
Cronograma inicial estava errado. Submissão no Dia 28 caiu depois de 30/11 em uma das versões. Corrigido aqui.
Custo tratado como se fosse parte do IAV. Corrigido: custo é camada separada.
Ferramentas de patrocinadores tratadas como garantidas. Confirmar regulamento.
Nome inconsistente: RotaZFM, RouteZFM e ZFMroute. Escolher RotaZFM e usar em tudo, inclusive no repositório.
Rota Porto Velho–São Paulo não verificada. Confirmar com mapa oficial e distância real.
Muitas dependências novas e recentes (AIHOT, Firecrawl, Cloudflare Agents) com curva de aprendizado que pode consumir os primeiros dias.
14.3 Onde o projeto é forte
Problema regional com dado público real.
Explicabilidade por dimensão, com falhas declaradas.
Exposição via MCP, alinhada à tese agêntica do evento.
Separação clara entre viabilidade e custo.
Escopo que pode ser cortado sem quebrar o fluxo principal.
14.4 Veredito

O projeto é executável no prazo se o escopo for mantido. O risco maior não é técnico: é a combinação de escopo inflado e validação de problema ausente. Se o time fizer três entrevistas nos primeiros cinco dias, escolher um caso retrospectivo com dado real e aceitar os cortes, a proposta fica sólida. Sem isso, o pitch depende de premissas não verificadas.

15. Pitch e vídeo
15.1 O que pode ser afirmado
O IAV sinaliza riscos físicos e regulatórios das rotas com dados públicos.
Cada decisão mostra fontes, datas de coleta e dados faltantes.
No caso retrospectivo, o sistema teria sinalizado o risco com X dias de antecedência (se a validação confirmar).
Custo é estimado, com premissas declaradas.
A arquitetura é agnóstica à fonte: trocar as fontes permite outro corredor.
15.2 O que não pode ser afirmado
Que o IAV prevê disponibilidade de caminhão ou combustível.
Que o score é probabilidade calibrada.
Que a economia gerada é medida.
Que o sistema substitui o gestor.
Qualquer número de impacto ilustrativo apresentado como resultado.
15.3 Estrutura de três minutos
Bloco	Tempo	Conteúdo	Tela
1. Problema	0:00 a 0:30	Frase de posicionamento e a decisão de embarque	Planilha e decisão atual, identificadas como ilustrativas
2. Solução	0:30 a 1:00	Origem, destino e data; IAV das duas rotas	Dashboard real
3. Como funciona	1:00 a 1:45	Cinco dimensões, fontes e alertas de qualidade	Diagrama e dado real
4. Demo ao vivo	1:45 a 2:30	Mudança de dado, recálculo e resposta do agente via MCP	Dashboard e cliente MCP
5. Caso e visão	2:30 a 3:00	Caso retrospectivo com faixa de custo; limites declarados; visão de futuro	Relatório e slide

Regras: “logística” no primeiro bloco; todo número rotulado como real ou ilustrativo; plano B gravado para a demo ao vivo; versão em inglês.

16. Decisões pendentes
#	Decisão	Responsável sugerido	Prazo
1	Confirmar regulamento: datas, patrocinadores, comprovação	Melo	Dia 1
2	Padronizar o nome como RotaZFM	Time	Dia 1
3	Confirmar rota Porto Velho–São Paulo e distância	Dados	Dia 2
4	Verificar série do SACE para 2023 e contagens do DNIT	Dados	Dia 3
5	Documentar caso de interdição ou restrição da BR-319 com data	Dados	Dia 4
6	Aceitar ou descartar AIHOT como base	Backend	Dia 3
7	Definir “resultado ruim” da validação, por escrito	Time	Dia 5
8	Realizar três entrevistas com gestores do PIM	Quem tiver contato	Dia 5
9	Definir LLM ou template determinístico	Backend	Dia 10
10	Definir quem grava e edita o vídeo	Time	Dia 10
11	Confirmar presença na final, se selecionado	Melo	4/12
17. Visão de futuro (só no pitch)
Calibração de pesos com histórico de decisões e resultados.
Pesos configuráveis por tipo de carga.
Cotações reais de transportadoras para custo.
Mais fontes: ANA, ANTAQ, satélite para assoreamento.
Disponibilidade de veículo e combustível, quando houver dado.
Outros corredores e estados, trocando as fontes e mantendo o motor e o MCP.
Autenticação com WorkOS e múltiplos usuários.
Cobrança e gain-share via AbacatePay.
Visualização 3D de cubagem.
Fechamento

O RotaZFM tem tese clara, dados públicos reais e uma arquitetura executável. A parte mais frágil não é técnica. É a validação do problema com usuário e a escolha de um caso retrospectivo com dado verificável. Se essas duas coisas forem resolvidas nos primeiros cinco dias, o restante é execução com cortes disciplinados.
