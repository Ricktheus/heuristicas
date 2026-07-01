# Conteúdo dos slides — VRPTW Multiobjetivo (para colar no Gamma)

> **Como usar:** este arquivo reproduz, slide a slide, o conteúdo exato da
> apresentação `VRPTWMultiobjetivo.pptx`. Cada slide é separado por uma linha
> `---`, que o Gamma reconhece como quebra de card ao colar texto ("Colar
> texto" → "Importar"). O título de cada slide está em `#`, os sub-blocos em
> `##`. As **Notas do apresentador** ficam ao final de cada slide, entre `>` —
> cole-as no campo de notas do Gamma ou apague, como preferir.
>
> Projeto: **INF0415 · UFG · 2026/1** — Roteamento de Veículos com Janelas de
> Tempo (VRPTW), otimização multiobjetivo com GA, NSGA-II e Warm-start.
> Instância: Solomon C101, 25 primeiros clientes.

---

# Roteamento de Veículos com Janelas de Tempo

INF0415 · UFG · 2026/1

Otimização multiobjetivo com GA, NSGA-II e Warm-start

**Equipe:** Henrique · Eduardo · Vitor · Antonio · João Henrique

> Notas do apresentador: Projeto sobre VRPTW. Vamos mostrar modelagem, algoritmos e resultados.

---

# O problema — O que é o VRPTW?

`02`

**Definição do cenário**

- **Depósito** — 1 ponto central, origem de toda a frota
- **Clientes** — 25 pontos a atender — instância Solomon C101
- **Frota** — veículos homogêneos, capacidade Q = 200
- **Janela** — cada cliente só é atendido em [eᵢ, lᵢ] — chegar cedo espera, chegar tarde é infactível

**Dois objetivos em conflito**

- **f1 — minimizar:** Distância total
- **f2 — minimizar:** Número de veículos

> Menos veículos → rotas mais longas → maior distância. Os dois puxam em direções opostas.

> Notas do apresentador: Depósito, 25 clientes, frota com capacidade. Cada cliente tem janela de tempo. Minimizar distância E número de veículos — objetivos que conflitam.

---

# Representação — Giant-tour + Split

`03`

**Permutação de clientes**

`3 · 7 · 1 · 12 · 5 · 9 · 22 · 4 · …`

↓ **Split decoder** — quebra a rota ao violar capacidade ou janela

- Rota 1: ⌂ → 3 → 7 → 1 → ⌂
- Rota 2: ⌂ → 12 → 5 → 9 → ⌂
- Rota 3: ⌂ → 22 → 4 → … → ⌂

**Por que essa escolha?**

1. Não é preciso evoluir o número de rotas explicitamente
2. Crossover e mutação operam sobre permutações simples
3. Factibilidade garantida pela decodificação — sem penalização de restrições

> Notas do apresentador: Giant-tour: uma lista com a ordem de todos os clientes. O split decoder quebra em rotas quando violaria capacidade ou janela. Simplifica os operadores.

---

# Metodologia — Quatro etapas

`04`

- **CP1 · Modelagem** — Baseline Clarke-Wright + Algoritmo Genético
- **CP2 · Multiobjetivo** — NSGA-II, 30 gerações — fronteira de Pareto inicial
- **CP3 · Diferencial** — Warm-start no NSGA-II, 80 gerações
- **FINAL · Avaliação** — Hypervolume como métrica de qualidade

> Notas do apresentador: Três checkpoints: modelagem + baseline + GA; depois NSGA-II; depois warm-start. Nesta entrega final acrescentamos o hypervolume.

---

# Baseline — Clarke & Wright (Savings)

`05`

**Economia ao unir rotas de i e j:**

`savings(i,j) = d(0,i) + d(0,j) − d(i,j)`

1. Uma rota por cliente: ⌂ → Cᵢ → ⌂
2. Ordena as economias do maior para o menor
3. Funde se não violar capacidade nem janela

> Baseline simples e intencional — fusão testada em uma só orientação. Não é o estado da arte.

**Resultado**

- **556,5** — f1 · distância total
- **10** — f2 · veículos

> Notas do apresentador: Clarke & Wright, heurística clássica dos anos 60. Começa com um caminhão por cliente e funde rotas pela maior economia. 10 veículos, distância 556 — nossa referência mínima.

---

# Checkpoint 1 — Algoritmo Genético

`06`

**Minimizamos:** `fitness = f1 + 50 × f2`

- **Crossover** — OX — preserva blocos de adjacência
- **Mutação** — Shuffle de índices (indpb 5%)
- **Seleção** — Torneio k=3 + elitismo
- **Parâmetros** — pop 120 · 150 ger · cxpb 0.8 · mutpb 0.2

**Resultados · 5 sementes**

| Semente | f1 | f2 |
|---|---|---|
| 1 | 343,57 | 4 |
| 2 | 303,02 | 4 |
| 3 | 309,50 | 5 |
| 4 | 307,89 | 5 |
| 5 · melhor | 266,08 | 4 |
| **média** | **306,0 ±24,6** | **4,4** |

> −45% em distância e −6 veículos vs. baseline.

> Notas do apresentador: População de 120, 150 gerações. Single-objective: fitness = f1 + 50·f2. OX preserva ordem relativa. 5 sementes; melhor foi f1=266 com 4 veículos, -45% vs baseline.

---

# Checkpoint 2 — NSGA-II multiobjetivo

`07`

Otimiza f1 e f2 simultaneamente, sem agregá-los. Devolve uma fronteira de Pareto — soluções não-dominadas.

**A domina B** se é melhor ou igual em tudo e estritamente melhor em ao menos um objetivo. Não-dominada = nenhuma outra a supera em tudo.

- **Não-dominância** — classifica em fronts (front 1 = melhor)
- **Crowding** — preserva diversidade dentro do front

**Fronteira · 30 gerações**
*(inserir gráfico: `results/final_pareto_comparison.png`)*

> HV = 1.280,7 — ainda fraca; 30 gerações é pouco.

> Notas do apresentador: NSGA-II não soma os objetivos — otimiza em paralelo e devolve uma fronteira de Pareto de soluções não-dominadas. Com só 30 gerações a fronteira ainda é fraca, mas mostra o trade-off.

---

# Diferencial · D2 — Warm-start

`08`

Em vez de partir de 100% aleatório, semeamos a população inicial com soluções que já sabemos boas.

- **25%** — sementes boas
- **75%** — aleatório — mantém diversidade genética

As sementes boas:
- → Rota do baseline Clarke-Wright
- → Melhor solução do GA (seed 5, f1 = 266)

**Evolução de f1 · 80 gerações**
*(inserir gráfico: `results/final_convergence.png` / `cp3_warmstart_convergence.png`)*

> Notas do apresentador: Semeamos 25% da população inicial com soluções boas — a rota Clarke-Wright e a melhor do GA. 75% aleatório para manter diversidade. O aleatório começa em 686 e chega a 421; o warm-start começa em 266 e mantém.

---

# Métrica — Hypervolume

`09`

Área dominada pela fronteira, limitada por r = (750, 12). Quanto maior, melhor.

- **30 gen · sem WS** — 1.280,7
- **80 gen · sem WS** — 1.901,8
- **80 gen · com WS** — 3.871,4

**+104%** — ganho do warm-start em HV

> Notas do apresentador: Hypervolume: área do espaço de objetivos dominada pela fronteira até um ponto de referência (750,12). Quanto maior, melhor. O warm-start mais que dobrou: de 1.902 para 3.871, +104%.

---

# Resultados — Comparação final

`10`

| Método | f1 · distância | f2 · veículos | HV |
|---|---|---|---|
| Baseline C&W | 556,5 | 10 | — |
| GA · média (5 seeds) | 306,0 ±24,6 | 4,4 | — |
| GA · melhor (seed 5) | 266,1 | 4 | — |
| NSGA-II · 30 gen | 483,7–524,0 | 7–9 | 1.281 |
| NSGA-II · 80 gen | 420,5 | 6 | 1.902 |
| NSGA-II · 80 gen + warm-start | 266,1 | 4 | 3.871 |

> Mesmo melhor f1 do GA, porém com fronteira de Pareto completa — o decisor escolhe o trade-off.

> Notas do apresentador: Resumo. O GA já reduziu distância em 45%. O NSGA-II com warm-start iguala o melhor f1 do GA MAS entrega uma fronteira completa — o tomador de decisão tem opções. HV 3.871 vs 1.902.

---

# Conclusões — Contribuições

`11`

1. Modelagem completa do VRPTW — giant-tour + split decoder
2. Comparação rigorosa de 3 abordagens: heurística, GA e NSGA-II
3. Warm-start com ganho comprovado: +104% em hypervolume
4. Métrica quantitativa (Hypervolume) para comparação objetiva

**Trade-off**

Menos veículos implica rotas mais longas — e o NSGA-II mapeia esse trade-off inteiro.

> Notas do apresentador: Solução completa: baseline, GA e NSGA-II. Warm-start: alto impacto, baixo custo — mais que dobrou a fronteira. Hypervolume quantifica a melhoria.
