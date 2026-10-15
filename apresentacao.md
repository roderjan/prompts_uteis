# DataCost Architect — cola da apresentação (30 min)

> Reunião com diretoria e lideranças · hoje, 16h30–17h · navegando no outro monitor.
> Números abaixo = o que aparece na tela com o cenário padrão (preços de 15/10/2026, US$ 1 = R$ 5,06).

---

## 0. Antes de começar (5 min antes)

- [ ] Abrir **duas abas**: `D:\TCC\calculator\index.html` (visão técnica) e `D:\TCC\calculator\executive.html` (painel gerencial). Funcionam sem internet.
- [ ] Nas duas: **Idioma = Português (BR)**. Na técnica, **Moeda = BRL** (diretoria pensa em reais).
- [ ] Fechar abas e notificações. Zoom do navegador em 110–125% para a tela compartilhada.
- [ ] **Não clicar em "Atualizar preços"** durante a reunião (baixa ~480 MB da AWS).
- [ ] Se mexer em algum campo e se perder: **recarregar a página (F5)** volta ao cenário padrão.

---

## 1. Roteiro (30 min)

| Tempo | Bloco | Onde |
|---|---|---|
| 0–3 | Abertura: o problema e a ideia em uma frase | sem tela ou na visão técnica |
| 3–6 | O pipeline de exemplo (o "cenário padrão") | aba **Workload & pipeline** |
| 6–16 | Demo: custo, Go/No-Go, frequência, comparação, otimização | abas Resultado → Go/No-Go → Impacto do schedule → Comparar cenários → Otimização |
| 16–20 | Painel gerencial (visão de diretoria) | `executive.html` |
| 20–24 | Como foi construído e de onde vêm os preços + validação | aba **Premissas** |
| 24–27 | Pendências, backlog e próximos passos | — |
| 27–30 | Perguntas | seção 8 desta cola |

---

## 2. Abertura (fale isto)

> "Na nuvem, a gente só descobre quanto um pipeline de dados custa **depois** que ele roda, quando a fatura chega. O DataCost Architect inverte isso: a gente descreve o pipeline — de onde vem o dado, quanto volume, de quanto em quanto tempo roda, quais ferramentas usa — e ele **estima o custo antes de executar**, compara arquiteturas, sugere economias e diz se cabe no orçamento: **GO, REVIEW ou NO-GO**."

> "É o nosso TCC do MBA em Engenharia de Dados na FIAP (eu, Thiago, Mauricio e Murilo). Comparamos com as calculadoras oficiais da AWS, Snowflake e Databricks: **errou no máximo 0,7% por item e 0,1% no total**."

Ideia-chave para repetir: **custo vira requisito de arquitetura, discutido antes — não surpresa na fatura.** Isso é o que o mercado chama de **FinOps**.

---

## 3. Demo passo a passo

### 3.1 Aba "Workload & pipeline" (3 min)

**Mostre** os cards do topo e o fluxo de caixinhas com setas.

**Fale:** "Este é um caso típico: um banco **Oracle** que gera **120 GB de dado novo por dia**. O dado passa por camadas — **Raw → Bronze → Silver → Gold** — que vão limpando e organizando, até chegar no **Snowflake**, onde o BI consulta. Cada caixinha é uma etapa, com sua ferramenta e frequência, e já mostra quanto custa por mês."

- Explique as camadas como uma **cozinha**: Raw = ingrediente como chegou; Bronze = separado e etiquetado; Silver = limpo e cortado; Gold = prato pronto para servir.
- **Não precisa** abrir "Parâmetros avançados". Se perguntarem, cada campo tem um **"i"** que explica o que é.

### 3.2 Aba "Resultado" (2 min)

**Aponte na barra fixa do topo:**
- **Custo mensal ≈ R$ 13.800** (US$ 2.725), com **faixa** de US$ 2.283 a 3.166 ("é uma estimativa, então mostramos a margem").
- **SLA: PASS**, processa em ~5,3 h (limite de 6 h).
- **Confidence 82%**: quanto mais informação damos, mais estreita a faixa.

**Na aba, mostre "Custo por categoria":**
- **Storage ≈ 46%**, Warehouse ≈ 30%, Manutenção ≈ 12%, Processamento ≈ 9%.
- **Fale:** "O surpreendente: **guardar** dado custa mais que **processar**. Por isso retenção e formato de arquivo importam tanto."
- **Custo por TB ≈ US$ 765** — "unit economics: custo por unidade, não só o total".

### 3.3 Aba "Go / No-Go" (2 min) — o momento mais forte

**Mostre:** veredito **NO-GO** — o custo (US$ 2.725) passa o orçamento de US$ 2.500 (109%).

**Fale:** "Antes de rodar, ele já avisa que estoura. E não só avisa: mostra **o caminho para caber**." Aponte o passo sugerido: **reduzir o auto-suspend do warehouse para 60 s** → cai para **US$ 2.481**, dentro do orçamento, **SLA mantido**.

> "Em produção, isso poderia **bloquear o job** antes de gastar o dinheiro."

### 3.4 Aba "Impacto do schedule" (2 min)

**Fale:** "Pergunta clássica do negócio: *quanto custa ter o dado de hora em hora em vez de uma vez por dia?*"
- Diário: **US$ 2.725** → De hora em hora: **US$ 2.925** (+7%) → A cada 5 min: **US$ 5.219** (quase o dobro).
- "O custo **não** cresce na proporção das execuções, porque o armazenamento não muda. Agora dá para negociar o prazo com o negócio **sabendo o preço**."

### 3.5 Aba "Comparar cenários" (2 min)

**Fale:** "Ele recalcula o **mesmo** pipeline em 7 arquiteturas diferentes e ranqueia."
- Perfil **Balanced** (padrão): 1º **Servir do lake (Athena), sem DW** — **US$ 1.869/mês** (−31%).
- Troque o perfil para **Performance**: 1º passa a ser **Databricks Photon** (US$ 2.469).
- "Não existe arquitetura 'certa' sem o requisito de negócio: o **perfil de decisão** muda a recomendação."

### 3.6 Aba "Otimização" (1 min)

4 sugestões no cenário padrão, cada uma com o efeito no SLA:
| Sugestão | Economia/mês |
|---|---|
| Auto-suspend do warehouse para 60 s | US$ 243 (8,9%) |
| Migrar Glue para Glue 6.0+ | US$ 192 (7,1%) |
| Reduzir retenção da camada bruta para 90 dias | US$ 190 (7,0%) |
| Mover camada bruta para S3 Standard-IA (mais barato) | US$ 115 (4,2%) |

(Opcional) **Sensibilidade & break-even:** "acima de ~11 TB/mês, outra arquitetura passa a ser a mais barata" — mostra **quando** mudar de arquitetura conforme o dado cresce.

### 3.7 Painel gerencial — `executive.html` (4 min)

**Fale:** "Esta é a visão para diretoria, com o **mesmo motor de cálculo**."
- Cenário: **2,5 TB/dia**, crescendo **5% ao mês**, retenção 12 meses.
- Compara **arquitetura legada** (Glue + Snowflake) × **otimizada** (Databricks Photon): **12% de redução**, **≈ R$ 275 mil economizados no 1º ano**, **≈ R$ 6,1 milhões em 5 anos** (com o crescimento composto).
- Mês 1 otimizado: **≈ R$ 124 mil/mês**; **73% é storage**.
- Card **Viabilidade (SLA)**: "Ele dimensiona as máquinas para cumprir o prazo e **avisa**: com esse crescimento, a partir do **mês 19** nem a arquitetura otimizada cumpre o SLA. É um alerta de planejamento, não só de custo."
- Mexa no slider de **volume** ou **crescimento** ao vivo — os números recalculam na hora.

> Atenção: os R$ do painel **variam com o câmbio** (cotação do Banco Central Europeu do dia da coleta).

---

## 4. Como foi construído (2 min, sem jargão)

- **Duas telas** que abrem direto no navegador, **sem servidor e sem internet** — basta o arquivo.
- **O "motor"** (a parte que calcula) foi escrito **duas vezes**, em JavaScript (roda no navegador) e em Python (referência). Testes automáticos garantem que os dois dão **exatamente o mesmo resultado** em 14 cenários. São **129 testes** automáticos no total.
- **Validação:** pegamos 3 pipelines (pequeno, médio, grande), digitamos as mesmas quantidades nas **calculadoras oficiais** de AWS, Snowflake e Databricks e comparamos linha por linha.
- **Na validação descobrimos e corrigimos** dois detalhes de cobrança que o modelo não tinha: (1) o S3 fica **mais barato por GB acima de 50 TB**; (2) um warehouse do Snowflake não pode ficar ligado mais horas do que o mês tem — acima disso precisa de "multi-cluster". Bom exemplo de que a validação **pegou erro real**.

---

## 5. De onde vêm os preços (pergunta provável)

| Provedor | Como coletamos | Em palavras simples |
|---|---|---|
| **AWS** | **AWS Price List** — arquivos públicos oficiais de preço | A própria AWS publica a tabela completa; um programa baixa e extrai só o que usamos |
| **Azure** | **Azure Retail Prices API** — consulta oficial e aberta | A Microsoft responde "quanto custa X" via internet |
| **Snowflake e Databricks** | Tabela **transcrita** das páginas oficiais e **conferida** nas calculadoras deles | Eles **não publicam** API de preço aberta |
| **Câmbio** | Taxas de referência do **Banco Central Europeu** (serviço Frankfurter) | Só para mostrar em R$/€; o cálculo é sempre em dólar |

- Sem login, sem senha, sem conta: tudo **público**.
- Cada preço fica registrado com **fonte e data** (aba **Premissas** → tabela com etiquetas `api` / `curated`). 27 dos 38 preços vêm direto de API.
- O botão **"Atualizar preços"** roda essa coleta de novo (precisa de um programa local ligado; leva alguns minutos).
- É **preço de lista** (tabela cheia). Desconto negociado da empresa pode ser simulado no campo **"Desconto contratual"**.

---

## 6. Números para citar

- **Erro vs. calculadoras oficiais:** 0,0% a 0,7% por item; **0,1% no total** de cada pipeline; erro médio 0,5% em 19 comparações.
- **Velocidade:** compara 7 arquiteturas + todas as análises em **menos de 10 milissegundos** no navegador.
- **Economia encontrada:** 1 a 4 otimizações por pipeline, **até 24% de economia combinada** sem perder o prazo (SLA).
- **SLA:** classificou certo **168 de 168** cenários de teste.

---

## 7. Pendências, backlog e próximas features

**O que falta no TCC (honesto):**
1. **Medir a velocidade real das ferramentas.** Hoje o tempo de processamento usa uma velocidade *estimada* por ferramenta. Já temos o kit pronto, inclusive para rodar de graça no trial do Snowflake. Se a velocidade real for metade da estimada, o custo de um pipeline pequeno sobe até ~50% — por isso é prioridade.
2. **Validação por especialistas:** profissionais vão dizer, às cegas, qual arquitetura escolheriam, para medir se a recomendação da ferramenta bate.

**Backlog / próximas features (em ordem de valor):**
1. **Comparar com a fatura real** (estimado × realizado) — o passo mais valioso para uma empresa.
2. **Preço da própria conta** (com descontos negociados), não só preço de lista.
3. **Mais regiões** e **Google Cloud**.
4. **Concorrência de usuários** e auto-scaling (hoje não modelados).
5. **Salvar e compartilhar cenários**, multiusuário.
6. Montar o pipeline **arrastando caixinhas** (visual).
7. Custos indiretos: orquestração, monitoramento, governança.

**Se perguntarem "dá para usar aqui na empresa?":**
> "Como protótipo para discutir arquitetura antes de construir, já dá. Para virar ferramenta de decisão oficial, os próximos passos seriam: usar a **região** e os **preços com desconto** da nossa conta, **medir a velocidade** com jobs reais nossos e **comparar com a fatura** de alguns pipelines existentes."

---

## 8. Perguntas prováveis — respostas curtas

**"Quão preciso é?"** — Contra as calculadoras oficiais, 0,1% no total. Mas é **preço de lista** e o tempo de execução ainda é estimado; não substitui a fatura.

**"Por que não perguntar ao ChatGPT?"** — Aqui cada preço tem **fonte oficial e data**, o cálculo é **sempre igual** para a mesma entrada, foi **testado** contra as calculadoras oficiais e mostra todas as **premissas**. Uma IA responde com base em dados de treino de data incerta.

**"Os preços ficam desatualizados?"** — Um clique em "Atualizar preços" recoleta tudo das fontes oficiais. A tela avisa quando os preços têm mais de 7 dias.

**"Considera o desconto que temos com a AWS?"** — Não automaticamente: dá para simular no campo "Desconto contratual". Há suporte técnico para ler o preço real da conta Snowflake/Databricks, mas exige acesso à conta.

**"Algum dado da empresa sai daqui?"** — Não. Roda **local**, no navegador. Só acessa a internet para buscar tabelas **públicas** de preço, quando alguém pede.

**"Por que essas ferramentas (AWS, Snowflake, Databricks)?"** — São as mais usadas em plataformas de dados; a Azure entrou como alternativa. Google Cloud está no backlog.

**"Quanto custou construir?"** — Praticamente zero em nuvem: tudo usa fontes públicas e roda local.

**"Por que storage pesa tanto?"** — O dado se acumula: cada dia soma mais, e várias camadas guardam cópias por 1 a 3 anos. Processar é pontual; guardar é contínuo.

**"O que é o GO / REVIEW / NO-GO?"** — GO: cabe no orçamento mesmo no pior caso da estimativa. REVIEW: cabe no valor esperado, mas o pior caso estoura. NO-GO: não cabe.

**Não sabe a resposta?** — "Boa pergunta; anoto e te respondo depois." (Não invente.)

---

## 9. Glossário (para você não travar)

### Nuvem e FinOps
| Termo | O que é, em uma linha |
|---|---|
| **Cloud / nuvem** | Alugar computadores e armazenamento de um provedor e pagar pelo uso, como conta de luz |
| **AWS / Azure / GCP** | Os três maiores provedores de nuvem: Amazon, Microsoft e Google |
| **FinOps** | Prática de gerir o custo da nuvem juntando TI, finanças e negócio |
| **Preço de lista / on-demand** | Preço cheio de tabela, sem desconto |
| **Savings Plans / compromisso** | Desconto em troca de prometer um consumo por 1 ou 3 anos |
| **Unit economics** | Custo por unidade (por TB, por consulta, por cliente), não só o total |
| **Região** (us-east-1) | Local físico do data center; preço varia por região. Usamos Virgínia (EUA) |
| **Egress** | Taxa para tirar dado da nuvem ou mandar para outra região |

### Dados e arquitetura
| Termo | O que é |
|---|---|
| **Pipeline de dados** | A "esteira" que leva o dado da origem até o relatório |
| **ETL / ELT** | Extrair, Transformar, Carregar — a ordem muda onde a transformação acontece |
| **Data lake** | Depósito barato de arquivos de dados (no nosso caso, o S3) |
| **Data warehouse (DW)** | Banco otimizado para relatórios e BI (no nosso caso, o Snowflake) |
| **Lakehouse** | Lake com recursos de warehouse, sem precisar copiar o dado |
| **Raw / Bronze / Silver / Gold** | Camadas: bruto → organizado → limpo → pronto para consumo ("arquitetura medalhão") |
| **BI** | Painéis e relatórios (Power BI, Tableau…) |
| **Full load / Incremental / CDC** | Copiar tudo toda vez / copiar só o que mudou / capturar cada alteração em tempo real |
| **Schedule / frequência** | De quanto em quanto tempo o pipeline roda |
| **Orquestração / DAG** | O "maestro" que dispara as etapas na ordem certa |
| **SLA** | Prazo máximo combinado para o processamento terminar (aqui: 6 h) |
| **Freshness** | "Idade" máxima aceitável do dado quando chega ao usuário (aqui: 30 h) |
| **Latência** | Tempo entre o dado nascer na origem e estar disponível |
| **Parquet / ORC / CSV / JSON** | Formatos de arquivo; **Parquet** é compactado e por colunas = menor e mais rápido de ler |
| **Iceberg / Delta / Hudi** | "Formatos de tabela": uma camada de controle sobre os arquivos (histórico, transações); ocupa um pouco mais, mas consulta lê menos dado |
| **Compaction / manutenção** | Juntar arquivos pequenos e apagar versões antigas — custa processamento |
| **Retenção** | Quantos dias o dado fica guardado |

### Serviços (o que é cada um)
| Serviço | O que faz | Como cobra |
|---|---|---|
| **Amazon S3** | Armazena arquivos (o lake) | Por GB guardado por mês + por acesso |
| **S3 Standard-IA / Glacier** | Versões mais baratas do S3 para dado pouco acessado | Mais barato guardar, paga ao ler |
| **AWS Glue** | Roda processamento de dados sem gerenciar servidor | Por **DPU-hora** (DPU = unidade de máquina da AWS) |
| **Glue 6.0+** | Versão nova do Glue | DPU-hora ~30% mais barata |
| **Amazon EMR** | Cluster de máquinas para processamento, mais controle | Hora de máquina (EC2) + taxa do EMR |
| **Amazon EC2** | Máquina virtual alugada por hora | Por hora ligada |
| **AWS DMS** | Replica banco de dados continuamente (CDC) | Máquina ligada 24h |
| **Amazon Athena** | Consulta SQL direto nos arquivos do S3 | Por **TB lido** |
| **Databricks** | Plataforma de processamento de dados (Spark) | Por **DBU** (unidade deles) + a máquina |
| **Photon** | "Turbo" do Databricks: gasta ~2× DBU, mas termina bem mais rápido | — |
| **Databricks Serverless / SQL** | Versões sem gerenciar máquina / para consultas de BI | Por DBU |
| **Snowflake** | Data warehouse na nuvem | Por **crédito** (US$ 2) × horas ligadas |
| **Warehouse Snowflake (XS, S, M, L)** | "Motor" de processamento do Snowflake; cada tamanho dobra velocidade e preço | Créditos por hora |
| **Auto-suspend** | Desligar o warehouse sozinho após X segundos parado | Parado e desligado = não cobra |
| **Multi-cluster** | Vários warehouses em paralelo para muita gente consultando | Cobra cada um ligado |
| **Snowpipe** | Carga contínua de arquivos no Snowflake | Por volume carregado |
| **Gen 1 / Gen 2** | Gerações de warehouse do Snowflake; Gen 2 é mais rápida e ~35% mais cara por hora | — |
| **Azure ADLS** | O "S3 da Microsoft" | Por GB/mês |
| **Spark / PySpark** | O motor de processamento distribuído que Glue, EMR e Databricks usam por baixo | — |

### Termos da ferramenta
| Termo | O que é |
|---|---|
| **Worker / nó** | Cada máquina que processa em paralelo; mais workers = mais rápido e mais caro por hora |
| **Throughput** | Velocidade: quantos GB uma máquina processa por minuto (hoje é **estimado** — a pendência nº 1) |
| **Confidence** | Quanto confiamos na estimativa; define a largura da faixa |
| **Go / No-Go** | Veredito se o pipeline cabe no orçamento |
| **Perfil de decisão** | Pesos do ranking: priorizar custo, equilíbrio ou desempenho |
| **Sensibilidade** | Como o custo muda se o volume crescer |
| **Break-even** | Ponto de volume em que outra arquitetura passa a ser mais barata |
| **Calculadora oficial** | Site de cada provedor para estimar preço (nossa referência de validação) |
| **API** | Forma de um programa pedir informação a outro pela internet |

---

## 10. Se algo der errado

- **Tela estranha / perdi o cenário:** F5 (volta ao padrão).
- **Ficou em inglês:** seletor **Idioma** no topo.
- **Números em dólar:** seletor **Moeda** → BRL.
- **Pediram um cenário específico:** aba Workload — mude **"Volume alterado por dia"** e **"Orçamento mensal"**; tudo recalcula na hora.
- **Pergunta técnica profunda:** "Está documentado no repositório; te mando o detalhe depois."

**Fechamento (fale isto):**
> "Resumo: em vez de descobrir o custo na fatura, a gente estima antes, com erro de 0,1% contra as calculadoras oficiais, compara arquiteturas e sabe se cabe no orçamento. O próximo passo natural para uma empresa é ligar isso aos **preços e faturas reais da nossa conta**."
