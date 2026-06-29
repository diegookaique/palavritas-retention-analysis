# Palavritas — O que determina a retenção no jogo de palavras diário?

**Case técnico — Analista de Dados (Produto & Growth) @ the news**

> O the news é a maior newsletter de notícias do Brasil, com mais de 2 milhões de assinantes. O Palavritas é o jogo diário de palavras (estilo Wordle) dentro do app do the news. Este case analisa dados de comportamento dos usuários para responder à pergunta do Head de Produto: **o que determina se um usuário volta a jogar — e o que podemos fazer para aumentar isso?**

---

## Dataset

Três arquivos cobrindo ~5 meses de atividade:

| Arquivo | Linhas | Descrição |
|---|---|---|
| `palavritas_sessions.csv` | ~41.000 | Uma linha por sessão de jogo (resultado, tentativas, dispositivo, sinalizadores de retenção) |
| `palavritas_attempts.csv` | ~149.000 | Uma linha por tentativa dentro de uma sessão |
| `user_profile.csv` | 800 | Dados demográficos e comportamentais de uma pesquisa com parte dos usuários |

## Abordagem

A análise seguiu as três entregas solicitadas no case:

1. **Limpeza e diagnóstico dos dados** — auditei as três tabelas antes de qualquer análise, já que vários problemas distorceriam os resultados se não fossem tratados.
2. **Análise exploratória e estatística** — testei quais variáveis se correlacionam com a retenção em 30 dias (`active_d30`) e o retorno no dia seguinte (`played_next_day`): horário de jogo, palavra do dia, perfil do usuário (setor, salário, dispositivo), frequência de food delivery e relação com a newsletter.
3. **Proposta de ação** — traduzi o achado mais robusto em uma hipótese testável, uma ação concreta e um critério de sucesso mensurável.

## Principais achados de qualidade dos dados

- **~1.200 linhas de sessão 100% duplicadas** (idênticas em todas as colunas) — removidas.
- **Formato de data misturado** na coluna `word_date` (`YYYY-MM-DD` e `DD/MM/YYYY` na mesma coluna) — exigiu parsing unificado antes de qualquer análise temporal.
- **A coluna `attempts` não é confiável**: não existe nenhuma sessão em todo o dataset com vitória na 4ª ou 5ª tentativa, e toda derrota aparece com o mesmo valor padrão (6), independente de quantas tentativas reais foram usadas. Confirmado ao cruzar com a tabela `attempts`, que mostrou 1.195 divergências entre o valor declarado e a contagem real.
- **Falhas de integridade referencial**: 200 `session_id` órfãos em `attempts` sem sessão correspondente, e 90 sessões sem nenhum registro de tentativa.
- **Cobertura limitada de perfil**: apenas 800 dos ~1.200 usuários únicos (66,7%) têm dados demográficos/comportamentais — qualquer análise cruzando comportamento com perfil cobre dois terços da base e pode ter viés de quem respondeu à pesquisa.
- Ajustes menores: inconsistência de capitalização em `device` e `job_role` (ex.: `iOS`/`ios`/`IOS`), tipos mistos em `orders_food_delivery` (`True`/`False` vs. `"sim"`/`"não"`), e uma pequena quantidade de valores impossíveis (tempo de conclusão negativo, tentativas fora do intervalo válido de 1–6).

## Principais achados da análise

- **Abrir a newsletter antes de jogar é o preditor mais forte e estatisticamente robusto da retenção em 30 dias.** Usuários que abrem a newsletter antes de jogar no mesmo dia atingem 37,8% de retenção em D30, contra 30,5% entre quem não abre (χ² = 151,4, p ≈ 8,7e-35).
- **Esse não é um sinal independente — é a mesma população dos "jogadores matinais".** 100% das sessões em que a newsletter foi aberta antes ocorreram entre 6h e 8h da manhã. Tratado como um único padrão de comportamento (uma rotina matinal), não como dois achados separados.
- Uma regressão logística controlando resultado do jogo, dispositivo e sequência de dias confirma que abrir a newsletter antes de jogar é o único preditor estatisticamente significativo (odds ratio = 1,37, p < 0,001). A sequência de dias (streak) parecia promissora numa tabela simples, mas **não** se confirmou significativa quando testada com mais rigor (p = 0,757) — uma verificação importante contra conclusões baseadas em ruído de amostras pequenas em sequências altas.
- Dispositivo, setor, faixa salarial, porte da empresa e frequência de food delivery não mostraram relação relevante com a retenção.
- A análise exploratória no nível de palavra sugeriu que palavras mais difíceis (mais tentativas, menor taxa de vitória) associam-se a retenção levemente maior — mas isso é baseado em apenas 30 palavras únicas e está marcado como de baixa confiança; não foi usado para sustentar a proposta abaixo.

## Proposta

| | |
|---|---|
| **Hipótese** | A correlação observada pode refletir um segmento de usuários já mais engajado — não necessariamente um efeito causal da própria newsletter sobre a retenção. |
| **Ação** | Antes de mudar o produto, rodar um teste controlado pequeno: selecionar aleatoriamente um grupo de usuários que normalmente não abrem a newsletter antes de jogar, incentivá-los a fazer isso por duas semanas, e comparar a retenção em D30 com um grupo controle equivalente. |
| **Critério de sucesso** | Uma diferença estatisticamente significativa de retenção a favor do grupo incentivado confirma um efeito causal e justifica investimento de produto (ex.: link direto da newsletter para o jogo). Se não houver diferença significativa, o achado original é melhor explicado como um marcador de engajamento pré-existente, e o esforço de retenção deve ser direcionado para outras frentes. |

## Conteúdo do repositório

- [`analise_palavritas.ipynb`](./analise_palavritas.ipynb) — limpeza de dados, EDA e análise estatística completas (Python)
- [`Documento_Analise_Palavritas.docx`](./Documento_Analise_Palavritas.docx) — documento não-técnico para o Head de Produto (diagnóstico, achados, proposta)
- [`dashboard_palavritas.csv`](./dashboard_palavritas.csv) — dataset limpo no nível de sessão usado para construir o dashboard
- **Dashboard interativo:** https://datastudio.google.com/s/gORGWT-gBDg

## Ferramentas

Python (pandas, NumPy, SciPy, statsmodels, Matplotlib) · Jupyter Notebook · Google Looker Studio (Data Studio)
