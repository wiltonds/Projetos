# Will Costa — Portfólio técnico

Este repositório reúne quatro projetos que mostram o mesmo padrão de trabalho em
contextos diferentes: sistemas de IA em produção real, arquitetura desenhada ponta
a ponta com validação humana, e inteligência de dados aplicada a decisão de negócio.

**[→ Landing page do portfólio](./index.html)** — visão geral, trajetória e
ferramentas. Publicado via GitHub Pages.

## Índice

1. [Radar de Oportunidades — sistema multiagente de inteligência comercial](#projeto-1--radar-de-oportunidades) — *em produção*
2. [Plataforma de acompanhamento de saúde com IA supervisionada](#projeto-2--plataforma-de-saúde) — *arquitetura + protótipo funcional*
3. [Motor de busca semântica curso ↔ mercado de trabalho](#projeto-3--busca-semântica) — *concluído*
4. [Sprout — coach educacional com fit por desafio real](#projeto-4--sprout) — *em desenvolvimento, 9+ meses*

---

## Projeto 1 — Radar de Oportunidades

**Status: em produção, Sistema FIEA/SENAI-AL**
**Ferramentas: n8n · Flowise · Google Apps Script · SQL Server (DW institucional) · Moskit CRM · CAGED/RAIS/CNPJ**

Sistema multiagente de inteligência comercial construído do zero para a área de
mercado do SENAI-AL. Resolve um problema concreto: a equipe comercial não tinha
visibilidade sistemática de onde estavam as oportunidades de negócio em Alagoas —
novos empreendimentos, vagas em alta, empresas sem cobertura recente.

**Pipeline ponta a ponta:**
`coleta (notícias, dados públicos) → classificação por IA (relevância geográfica/setorial) → enriquecimento (CAGED/RAIS/CNPJ) → briefing visual por e-mail → qualificação humana → CRM`

- **Base analítica:** dashboard Power BI unificado consolidando 27 indicadores de
  8+ sistemas institucionais, sobre um modelo de dados em star schema construído
  do zero.
- **Segmentação comercial:** modelo RFM aplicado a 2.461 empresas via dados do
  Moskit CRM, gerando um workbook de inteligência comercial por tier.
- **Modelo preditivo:** XGBoost + SHAP para risco de evasão escolar (segmento NEM),
  com calculadora de ROI financeiro para iniciativas de retenção.
- **Automação:** validado primeiro como MVP em planilha (Google Apps Script +
  OpenRouter, por restrição de acesso/orçamento), com a fase de produção definida
  para subir a qualificação de oportunidades para dentro do CRM institucional —
  decisão tomada em conjunto com a área comercial para evitar retrabalho na
  implementação.
- **Framework de governança:** todo o trabalho segue o FIE (Framework Inteligência
  e Evidência) como metodologia analítica.

*Nota de transparência: como este é um sistema institucional em construção ativa,
o repositório não inclui código-fonte proprietário — a documentação aqui descreve
arquitetura e decisões, não implementação licenciada à instituição.*

---

## Projeto 2 — Plataforma de saúde

Estudo de caso técnico: desenho e protótipo funcional de uma plataforma de coaching
de saúde (nutrição + treino) por assinatura, com agentes de IA supervisionados por
profissionais de saúde — arquitetura pensada para produção, não apenas demonstração.

**[→ Protótipo interativo](./prototipo-coaching-saude.html)** — roda no navegador,
chama a API da Anthropic diretamente, sem backend. Abra o arquivo local ou publique
via GitHub Pages.

---

## O problema que o projeto resolve

Ligar CRM, WhatsApp, IA de voz, geração de planos personalizados, check-ins semanais,
automações e aplicação de entrega ao cliente — mantendo um humano licenciado
responsável por cada decisão que chega ao usuário final, dentro de uma economia
unitária que sustenta uma assinatura de baixo custo (o desenho abaixo usa R$ 79,90/mês
como restrição de projeto, não R$ 400–1.800/mês do mercado tradicional de nutricionista
e personal trainer).

Essa restrição de preço é o que dirige toda decisão de arquitetura documentada aqui:
cada minuto de revisão humana tem custo, então o sistema existe para tornar esse
minuto o mais eficiente e seguro possível — nunca para eliminá-lo.

## Como o projeto responde a cada requisito

| Requisito da vaga | O que foi desenhado |
|---|---|
| **LLMs e agentes de IA** | Motor de composição restrito a catálogo (o LLM nunca gera item livre — só combina e escala itens pré-validados); classificação em arquétipos (nutrição + treino gerados a partir do mesmo objeto, nunca dessincronizados); agente de escalonamento que detecta sinal clínico fora do escopo e interrompe a automação |
| **Automações e workflows em produção** | Orquestração via n8n: evento → composição do plano → notificação push ao profissional → fila de revisão → aprovação → geração de PDF assinado → entrega; workflow paralelo de pagamento instantâneo (Pix) disparado na aprovação, com auditoria por amostragem que não bloqueia o pagamento mas afeta prioridade futura |
| **Integração de APIs de terceiros** | Avaliação comparativa de WhatsApp Cloud API/BSP (janela de 24h, categorias de template, mudança de cobrança de out/2026), gateways de Pix instantâneo (Asaas/Iugu/Celcoin), e plataformas de IA de voz (LiveKit self-host vs. Vapi/Retell gerenciado) |
| **CRM e WhatsApp Business** | Separação explícita entre CRM (estado comercial: assinatura, cobrança) e perfil clínico (estado de saúde) como fontes de verdade distintas — decisão que evita, por desenho, que uma automação comercial atinja um cliente com sinal clínico aberto |
| **Plataformas de IA de voz** | Análise de orçamento de latência (cascata STT→LLM→TTS vs. speech-to-speech), endpointing/barge-in, e trade-off compliance (HIPAA/SOC2/GDPR) vs. custo de operação própria |
| **Arquitetura técnica ponta a ponta** | Do canal de entrada (WhatsApp/voz/app) até a entrega final, passando por perfil, geração de plano, agendador de check-in e CRM — com decisão de dado documentada em cada camada |
| **Sistemas com validação humana** | Fila de revisão com visão lado a lado (dado bruto do cliente × ficha técnica gerada pela IA), diff entre versões do plano, validador programático que confere se toda saída do modelo respeita o catálogo, assinatura eletrônica registrada por aprovação (quem, quando, o quê) |

## Decisões de arquitetura (e por que)

**O plano é um objeto versionado, não um texto.** O que circula entre os sistemas é
uma estrutura de dados (`plano_id`, `versao`, `derivado_de`, `aprovado_por`,
`blocos`) — o texto em WhatsApp, o PDF e o roteiro de voz são renderizações
diferentes do mesmo objeto. Isso é o que permite diff entre versões (revisão em
minutos, não do zero) e rastreabilidade de quem aprovou o quê.

**O catálogo é a fonte de verdade, não o prompt.** Restrição alimentar, tags de
equipamento e limites clínicos são filtros aplicados *antes* do modelo ver as
opções — não instruções que o modelo pode, em tese, ignorar. Um validador roda
depois da geração e confere programaticamente se toda saída cita apenas itens
do catálogo; qualquer suspeita de item fora da lista é sinalizada para revisão
humana antes de aprovar, nunca corrigida silenciosamente pelo próprio modelo.

**Personalização é classificação, não geração livre.** Em vez de o LLM decidir
o programa de cada pessoa do zero, o cliente é classificado num pequeno conjunto
de arquétipos (objetivo × nível × restrição de equipamento) previamente validados
pelos profissionais. A individualização por cliente é escala de porção/carga a
partir de dados antropométricos — matemática determinística, não julgamento
clínico repetido a cada chamada de API. É essa divisão que torna a economia
unitária sustentável em R$ 79,90/mês.

**Nutrição e treino nascem cientes um do outro.** O plano de treino é gerado
recebendo o plano nutricional já aprovado como contexto (e vice-versa), evitando
o problema comum de mercado de dois profissionais prescrevendo sem sinergia para
o mesmo cliente.

**Pagamento instantâneo por aprovação, com guardrail de qualidade desacoplado.**
Pagar o profissional via Pix no momento da aprovação (padrão de app de
mobilidade) melhora fluxo de caixa e reforça autonomia profissional — mas cria
incentivo a aprovar rápido demais. A mitigação não atrasa o pagamento: uma
amostra de casos aprovados entra em auditoria em paralelo, cujo resultado afeta
prioridade e taxa futuras, não desfaz o que já foi pago.

## O que este protótipo deliberadamente não resolve

Documentado aqui porque um arquiteto sênior nomeia os limites do próprio trabalho
em vez de escondê-los:

- **Persistência.** O estado vive em memória no navegador; produção exige banco
  real com o objeto de plano versionado descrito acima.
- **WhatsApp real.** A interface simula a conversa; a integração real de Cloud
  API/BSP, janela de 24h e templates aprovados é a próxima camada.
- **Assinatura eletrônica de nível avançado.** O protótipo registra autoria e
  timestamp (suficiente para assinatura eletrônica simples); um requisito de
  ICP-Brasil do conselho de classe exigiria provedor dedicado (Clicksign/D4Sign).
- **Casos fora do arquétipo.** Comorbidade, lesão prévia ou objetivo atípico são
  desenhados para escalonamento a atendimento humano completo — o sistema
  reconhece o próprio limite em vez de forçar encaixe.

## Rodando o protótipo

Abra `prototipo-coaching-saude.html` em qualquer navegador. Ele chama
`https://api.anthropic.com/v1/messages` diretamente do cliente (sem chave
exposta no código — pensado para rodar dentro do ambiente de artifacts da
Anthropic). Para uma demo pública via GitHub Pages, adapte a chamada para
passar por um backend leve que injete a credencial.

---

---

## Projeto 3 — Busca semântica

**Status: concluído — Programa Inova Talentos (Itaú)**
**Ferramentas: Python · FAISS · Embeddings · TF-IDF · Word2Vec**

Dois entregáveis de fellowship com foco em NLP aplicado:

- **Motor de correspondência semântica** entre o catálogo de cursos do SENAI e a
  demanda real de mercado de trabalho, via embeddings indexados em FAISS —
  substitui busca por palavra-chave exata por correspondência de significado,
  relevante para o mesmo domínio de dado de emprego do Projeto 1.
- **Pipeline de análise de sentimento** sobre base de reviews da Amazon,
  comparando representação por TF-IDF contra Word2Vec, com avaliação de
  trade-off entre as duas abordagens.

---

## Projeto 4 — Sprout

**Status: em desenvolvimento — 9+ meses**
**Padrão: classificação por arquétipo · banco de desafios validado · revisão humana**

Plataforma de educação digital "student-first", voltada para quem está começando
ou migrando de carreira em tecnologia — inicialmente com foco em desenvolvedores
sem direção clara de mercado, incluindo caminhos para trabalho no exterior.

Combina três frentes: trilha técnica + inglês para sair do zero e começar a
gerar renda, parcerias B2B com empresas, e um serviço de headhunting — em vez
de indicar candidato por currículo e palavra-chave, a IA mapeia o fit com base
em como o candidato resolve **desafios reais que as empresas parceiras já
enfrentam**, com validação humana antes de qualquer indicação seguir adiante.

A espinha dorsal técnica reaproveita o mesmo padrão do Projeto 2: um banco de
desafios validado (equivalente ao catálogo), classificação do candidato num
conjunto fechado de perfis (equivalente ao arquétipo), e fila de revisão humana
antes da indicação sair — nenhum match é automático.

Validação em andamento com abordagem lean: já com o primeiro professor no time
e MVP definido em torno da trilha de ferramentas + inglês.

