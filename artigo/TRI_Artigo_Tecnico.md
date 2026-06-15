# Teoria de Resposta ao Item na Prática: Calibração Bayesiana, Validação e Diagnóstico

**Gilson Machado Monteiro**  
Especialização em Estatística Aplicada à Ciência de Dados — PUC Minas | 2025  
[github.com/gilsonmm6/tri-irt-bayesian](https://github.com/gilsonmm6)

---

## Resumo

Aplicamos Teoria de Resposta ao Item (TRI) a um instrumento dicotômico com 239 respondentes e 60 itens, comparando os modelos Rasch (1PL) e 2PL via inferência bayesiana completa em PyMC. O modelo 2PL superou o Rasch em poder preditivo (Δelpd = 86,97; peso ≈ 1,0) e apresentou ajuste local excelente (0/60 itens fora do envelope preditivo de 90%). Contudo, o comportamento agregado do score total — média e desvio padrão observados ambos fora do envelope PPC — aliado à ausência de fator dominante na PCA e dependência local pontual (2 pares com Q3 ≥ 0,20), sugere incompatibilidade com estrita unidimensionalidade. Os resultados apontam para extensões como MIRT ou modelo bifator como próximos passos.

**Palavras-chave:** TRI, Rasch, 2PL, inferência bayesiana, PyMC, posterior predictive check, Yen's Q3, psicometria.

---

## 1. Introdução

Em avaliações educacionais, o escore bruto (contagem de acertos) ignora diferenças importantes entre itens: um item difícil e um fácil contribuem igualmente para o total, o que distorce a estimativa de habilidade. A **Teoria de Resposta ao Item (TRI)** supera essa limitação modelando a probabilidade de acerto como função do traço latente θ (habilidade do respondente) e dos parâmetros do item.

O presente estudo aplica TRI bayesiana a um instrumento real, comparando o modelo de Rasch — que assume discriminação uniforme entre itens — com o modelo 2PL, que estima discriminação item a item. A inferência é conduzida em PyMC com validação rigorosa via Posterior Predictive Checks (PPC) e diagnósticos de pressupostos.

---

## 2. Metodologia

### 2.1 Dados

- **N:** 239 respondentes
- **J:** 60 itens dicotômicos (0 = erro, 1 = acerto)
- **Taxa média de acerto:** 0,422 (dificuldade moderada)
- **Estrutura:** matriz de respostas Y ∈ {0,1}^(239×60)

### 2.2 Modelos

**Rasch (1PL):**

$$P(Y_{pi}=1 \mid \theta_p, b_i) = \sigma(\theta_p - b_i)$$

Assume discriminação constante entre itens (a = 1 para todos). Prior: θ ~ Normal(0,1); b ~ Normal(0,1) centralizado.

**2PL:**

$$P(Y_{pi}=1 \mid \theta_p, a_i, b_i) = \sigma(a_i(\theta_p - b_i))$$

Estima discriminação item a item. Prior: a ~ LogNormal(0; 0,30) — prior firme que evita inflação de a e estabiliza a amostragem.

### 2.3 Inferência

- **Amostrador:** NUTS (No-U-Turn Sampler) via PyMC 5
- **Rasch:** 2 chains × 1.000 draws (tune=1.000)
- **2PL:** 2 chains × 1.500 draws (tune=1.500), target_accept=0,92
- **Log-likelihood:** calculado durante a amostragem (`idata_kwargs={"log_likelihood": True}`)

### 2.4 Avaliação

| Critério | Método | O que mede |
|---|---|---|
| Comparação de modelos | LOO (BB-pseudo-BMA) | Poder preditivo fora da amostra |
| Precisão do teste | TIF e SEM | Faixa de habilidade mais informativa |
| Ajuste local | PPC por item | Calibração item a item |
| Ajuste global | PPC do score total | Compatibilidade com unidimensionalidade |
| Pressupostos | PCA + Yen's Q3 | Estrutura latente e dependência local |

---

## 3. Resultados

### 3.1 Comparação de modelos (LOO)

| Modelo | elpd_loo | p_loo | Δelpd | Peso |
|---|---|---|---|---|
| **2PL** | -8.908,98 | 306,39 | 0,00 | **1,000** |
| Rasch | -8.995,95 | 282,76 | 86,97 | ≈0,000 |

A diferença de Δelpd = 86,97 (com dse = 9,16) é substancialmente maior que o limiar convencional de 4 pontos, indicando superioridade clara do 2PL. O peso de 1,0 significa que, em média preditiva ponderada, nenhuma massa recai sobre o Rasch.

### 3.2 Precisão do teste (TIF/SEM)

O pico de informação ocorre em **θ ≈ −0,37** (TIF ≈ 12,35; SEM ≈ 0,285), indicando que o instrumento discrimina melhor respondentes na faixa de **habilidade baixa a média**. Em θ = 0 (habilidade média-alta), a informação já decai para ≈ 10, e nas extremidades (|θ| > 2) a precisão cai consideravelmente.

**Implicação prática:** o instrumento é mais adequado para triagem de respondentes abaixo da média do que para discriminação nos extremos superiores.

### 3.3 Posterior Predictive Check — nível do item

Dos 60 itens, **nenhum ficou fora do envelope preditivo de 90%**. Isso indica que o 2PL captura adequadamente a proporção de acertos de cada item individualmente — evidência de bom ajuste local.

### 3.4 Posterior Predictive Check — nível global

| Estatística | Observado | Envelope PPC 90% | Dentro? |
|---|---|---|---|
| Média do score | 25,30 | [25,47 – 26,21] | ❌ Não |
| DP do score | 6,74 | [6,97 – 7,62] | ❌ Não |

O score total observado apresenta média e variabilidade **abaixo** do predito pelo modelo. O modelo 2PL subestima levemente os scores médios e superestima a dispersão esperada. Isso é característico de instrumentos com estrutura **multidimensional**: quando múltiplos traços contribuem para as respostas, um modelo unidimensional tende a subestimar a variância explicada.

### 3.5 Diagnóstico de pressupostos

**PCA:** nenhum fator único explica proporção dominante da variância — indício de estrutura multidimensional ou alta heterogeneidade entre itens.

**Yen's Q3:** dos pares de itens analisados, 2 apresentaram Q3_adj ≥ 0,20. Esse nível de dependência local é **pontual** e não suficiente, por si só, para explicar o desalinhamento global do PPC — reforçando a hipótese de multidimensionalidade como causa principal.

---

## 4. Discussão

Os resultados apresentam um padrão coerente internamente:

> O 2PL ajusta bem **localmente** (cada item individualmente) mas falha **globalmente** (comportamento agregado dos scores).

Esse padrão é típico de instrumentos que possuem um traço dominante (fator g, ou habilidade geral) acompanhado de subfatores específicos por domínio de conteúdo. Nesse cenário, o modelo unidimensional captura o traço dominante mas não a variância residual estruturada dos subfatores — o que se manifesta como subestimação da dispersão do score total.

**Implicações metodológicas:**
- O instrumento pode ser usado para estimativa de habilidade geral com ressalvas, especialmente nos extremos.
- Uma análise confirmatória com **MIRT** (Multidimensional IRT) permitiria separar os traços e verificar se há subfatores interpretáveis.
- O modelo **bifator** seria adequado se a hipótese for de um traço geral + domínios específicos ortogonais.

---

## 5. Conclusão

Este estudo demonstra que a aplicação de TRI bayesiana vai além da simples calibração de itens: a combinação de LOO, PPC por item, PPC global, PCA e Q3 forma uma cadeia diagnóstica que permite identificar não apenas se o modelo ajusta, mas **onde e por que** ele pode falhar.

O principal achado — bom ajuste local com desalinhamento global — é um resultado substantivo, não apenas técnico: aponta para a necessidade de modelos mais flexíveis e levanta questões sobre a estrutura latente do instrumento que só uma análise multidimensional poderia responder.

---

## Referências

- Rasch, G. (1960). *Probabilistic models for some intelligence and attainment tests*. Copenhagen: Nielsen & Lydiche.
- Birnbaum, A. (1968). Some latent trait models. In F. Lord & M. Novick, *Statistical theories of mental test scores*. Addison-Wesley.
- Gelman, A. et al. (2014). *Bayesian Data Analysis* (3rd ed.). CRC Press.
- Yen, W. M. (1984). Effects of local item dependence on the fit and equating performance of the three-parameter logistic model. *Applied Psychological Measurement*, 8(2), 125–145.
- Vehtari, A. et al. (2017). Practical Bayesian model evaluation using leave-one-out cross-validation and WAIC. *Statistics and Computing*, 27(5), 1413–1432.
- Salvatier, J. et al. (2016). Probabilistic programming in Python using PyMC3. *PeerJ Computer Science*.

---

*Código e dados disponíveis em: [github.com/gilsonmm6/tri-irt-bayesian](https://github.com/gilsonmm6)*
