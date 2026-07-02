# Explicação Técnica dos Slides — VRPTW Multiobjetivo

*Objetivo deste documento: para cada um dos 11 slides da apresentação, uma explicação técnica que parte do fundamento (o conceito básico por trás daquele slide) até o nível avançado (o que exatamente o código faz e por quê) — de forma que alguém sem domínio prévio do assunto consiga estudar este texto e, em seguida, explicar o slide com propriedade. Depois de cada explicação, 7 perguntas e respostas para treino de defesa.*

---

## Slide 1 — Capa: "Roteamento de Veículos com Janelas de Tempo"

**Conteúdo do slide:** INF0415 · UFG · 2026/1 · "Otimização multiobjetivo com GA, NSGA-II e Warm-start" · equipe.

### Explicação técnica (do fundamento ao avançado)

Todo problema de **otimização combinatória** consiste em escolher, dentre um conjunto finito (mas gigantesco) de configurações possíveis, aquela que minimiza ou maximiza uma ou mais medidas de interesse. O VRPTW (Vehicle Routing Problem with Time Windows) é uma instância clássica desse tipo de problema, situada na interseção entre o Problema do Caixeiro-Viajante (visitar pontos com o menor custo) e problemas de escalonamento (respeitar horários).

A capa já entrega a estrutura técnica inteira da apresentação em uma frase: o projeto usa **três técnicas algorítmicas em camadas**. Primeiro, um **Algoritmo Genético (GA)** single-objective, que serve de prova de conceito da modelagem e primeira melhoria sobre uma heurística simples. Segundo, o **NSGA-II**, a versão multiobjetivo do algoritmo genético, capaz de otimizar dois critérios ao mesmo tempo sem reduzi-los a um número só. Terceiro, o **Warm-start**, uma técnica de inicialização inteligente que os autores aplicaram como contribuição própria sobre o NSGA-II padrão. Essas três técnicas correspondem, respectivamente, aos Checkpoints 1, 2 e 3 do cronograma da disciplina — a capa é, na prática, um resumo executivo do roteiro técnico que será detalhado nos slides seguintes.

### Perguntas e Respostas

1. **O que significa a sigla VRPTW?**
   Vehicle Routing Problem with Time Windows — Problema de Roteamento de Veículos com Janelas de Tempo.

2. **Por que a capa já anuncia "GA, NSGA-II e Warm-start"?**
   Porque esses são os três pilares metodológicos do projeto, correspondentes aos Checkpoints 1, 2 e 3 — a capa funciona como roteiro da apresentação inteira.

3. **O que diferencia uma abordagem "single-objective" de uma "multiobjetivo"?**
   Single-objective otimiza um único critério escalar (podendo combinar vários objetivos com pesos fixos); multiobjetivo otimiza vários objetivos simultaneamente sem agregá-los, retornando um conjunto de soluções (a fronteira de Pareto).

4. **Por que esse projeto é identificado como "Problema P1" na proposta da disciplina?**
   É a identificação do problema escolhido pela equipe entre as opções propostas na disciplina de Heurísticas e Modelagem Multiobjetivo.

5. **A instância de dados usada já está decidida neste ponto, mesmo não aparecendo na capa?**
   Sim — o projeto usa a instância Solomon C101 restrita aos 25 primeiros clientes; isso é detalhado no Slide 2, mas já é uma decisão de escopo tomada desde a proposta inicial.

6. **Por que declarar a equipe na capa é relevante academicamente?**
   Estabelece autoria e responsabilidade coletiva pelo trabalho, prática padrão em entregas acadêmicas formais.

7. **Se a banca perguntar o que motivou a escolha desse problema entre as opções disponíveis, como responder?**
   Pela relevância prática (logística real, problema clássico de pesquisa operacional), pelo desafio combinatório (NP-difícil) e pela oportunidade de trabalhar tanto heurísticas construtivas quanto metaheurísticas mono e multiobjetivo dentro do mesmo problema.

---

## Slide 2 — O Problema: "O que é o VRPTW?"

**Conteúdo do slide:** Depósito · Clientes (25, instância C101) · Frota (Q=200) · Janela [eᵢ,lᵢ] · dois objetivos em conflito (f1 distância, f2 veículos).

### Explicação técnica (do fundamento ao avançado)

Formalmente, uma instância de VRPTW é definida por um grafo com um nó **depósito** (índice 0) e n nós **clientes**, cada um com coordenadas (x,y), uma **demanda** qᵢ (quanto "peso" ocupa no veículo) e uma **janela de tempo** [eᵢ, lᵢ]. Existe uma frota de veículos homogêneos, cada um com capacidade Q. O objetivo é particionar os clientes em rotas — cada rota começando e terminando no depósito — de forma que: (1) a soma das demandas de cada rota não exceda Q; (2) cada cliente seja atendido dentro de sua janela, onde chegar antes de eᵢ implica esperar e chegar depois de lᵢ é inadmissível.

No código (`src/instance.py`), esses elementos viram os campos de uma `VRPTWInstance`: `coords`, `demand`, `ready` (eᵢ), `due` (lᵢ), `service` (tempo de atendimento) e uma matriz `dist` pré-computada com distância euclidiana entre todos os pares de nós. A instância usada é a **C101** de Solomon (1987), reduzida a 25 clientes por orçamento computacional — C101 é uma instância "clusterizada" (clientes tendem a estar espacialmente agrupados) com janelas de tempo relativamente amplas, comparada a instâncias da família R (aleatórias) ou RC (mistas).

O ponto central do slide é o **conflito estrutural entre f1 (distância total) e f2 (número de veículos)**: dado um número fixo de clientes, se você reduz o número de rotas, cada rota precisa absorver mais clientes — e mais clientes por rota tendem a significar rotas geometricamente mais longas. Esse não é um efeito acidental do algoritmo; é uma propriedade inerente à forma como o espaço de clientes é particionado, e é exatamente essa tensão que justifica tratar o problema como multiobjetivo em vez de otimizar um único critério.

### Perguntas e Respostas

1. **Quais são as quatro entidades que definem uma instância do VRPTW?**
   Depósito, clientes (com demanda e coordenadas), frota (capacidade Q) e janelas de tempo [eᵢ, lᵢ] por cliente.

2. **O que acontece se um veículo chega antes de eᵢ?**
   Ele espera até eᵢ para iniciar o atendimento — não é penalizado, apenas aguarda.

3. **O que acontece se o veículo chegaria depois de lᵢ?**
   É considerado infactível; no split decoder do projeto, isso força o fechamento da rota atual e a abertura de uma nova a partir do depósito.

4. **Qual é a capacidade Q usada na instância do projeto?**
   Q = 200 (definida no cabeçalho do arquivo `C101.txt` e lida por `load_solomon_instance` em `instance.py`).

5. **Por que f1 e f2 estão em conflito estrutural, e não apenas empírico?**
   Porque, dado um conjunto fixo de clientes, reduzir o número de veículos obriga a concentrar mais clientes por rota, o que tende a alongar cada rota; é uma relação de compensação inerente a como as rotas particionam os clientes, não um artefato do algoritmo usado.

6. **A instância C101 é "aleatória" ou tem alguma estrutura conhecida?**
   C101 é uma instância clusterizada do benchmark de Solomon — os clientes tendem a estar agrupados espacialmente, e as janelas de tempo são relativamente amplas comparadas a outras famílias (R, RC).

7. **Como a matriz de distâncias é calculada no código?**
   `instance.py` calcula a distância euclidiana entre todos os pares de coordenadas (incluindo o depósito), gerando uma matriz (n+1)×(n+1) usada por todas as funções de avaliação.

---

## Slide 3 — Representação: "Giant-tour + Split"

**Conteúdo do slide:** permutação de clientes · split decoder · exemplo de rotas resultantes · três justificativas de design.

### Explicação técnica (do fundamento ao avançado)

Em metaheurísticas, **representação (ou codificação)** é a forma como uma solução candidata é armazenada e manipulada pelo algoritmo de busca. A formulação matemática "clássica" do VRPTW usa variáveis binárias xᵢⱼₖ (1 se o veículo k viaja diretamente do nó i ao nó j). Essa representação é fiel ao problema, mas dificulta a aplicação de operadores genéticos simples, porque qualquer crossover ingênuo sobre essas variáveis tende a gerar soluções inválidas (rotas desconexas, capacidade violada, clientes visitados duas vezes ou nenhuma).

O projeto opta por uma **representação indireta**: a solução é um **giant-tour**, uma permutação simples dos n clientes, sem nenhum marcador de onde uma rota termina e outra começa. Essa permutação sozinha não representa "rotas" — ela precisa ser **decodificada**. É aí que entra o **split decoder** (`split_into_routes` em `instance.py`): um algoritmo guloso, sequencial e O(n), que percorre a permutação da esquerda para a direita, mantendo carga acumulada e tempo acumulado da rota atual. Para cada cliente da permutação, ele verifica se adicioná-lo violaria a capacidade (`load + q > Q`) ou a janela de tempo (`start_service > due`); se qualquer uma dessas condições for verdadeira, a rota atual é fechada e uma nova rota é iniciada a partir do depósito para aquele cliente.

O resultado prático dessa escolha de design é que **toda permutação, sem exceção, decodifica em um conjunto de rotas 100% viável**. Isso elimina a necessidade de penalizar restrições na função objetivo (uma fonte comum de dificuldade de calibração em GAs) e permite reaproveitar diretamente operadores de crossover e mutação padrão para permutações (como Ordered Crossover), sem nenhum mecanismo de reparo customizado. É uma técnica de decodificação inspirada em trabalhos da literatura de VRP (como o "Split" de Prins), aqui implementada em sua versão mais simples — gulosa e sequencial, não ótima.

### Perguntas e Respostas

1. **O que é uma "permutação" no contexto do giant-tour?**
   Uma ordenação de todos os n clientes sem repetição, sem indicar onde uma rota termina e outra começa — apenas a sequência de visita.

2. **Qual é a complexidade computacional do split decoder para decodificar uma permutação?**
   O(n) — ele percorre a permutação uma única vez, mantendo carga e tempo acumulados, decidindo em tempo constante por cliente se deve fechar a rota.

3. **O split decoder é determinístico?**
   Sim — dada a mesma permutação e a mesma instância, ele sempre produz o mesmo conjunto de rotas, sem aleatoriedade.

4. **Por que a factibilidade "garantida pela decodificação" é vantajosa para os operadores genéticos?**
   Porque o GA/NSGA-II não precisa lidar com indivíduos inválidos nem calibrar pesos de penalização — todo indivíduo gerado por crossover/mutação, ao ser decodificado, já produz uma solução 100% viável.

5. **O split decoder considera a distância de retorno ao depósito ao decidir quando fechar uma rota?**
   Não diretamente — ele checa apenas a viabilidade de adicionar o PRÓXIMO cliente (capacidade e janela); a distância de retorno ao depósito só entra no cálculo de f1 depois, ao somar a rota inteira em `route_distance`.

6. **O que aconteceria se a ordem dos clientes na permutação fosse totalmente aleatória, sem nenhuma otimização?**
   O split decoder ainda produziria rotas viáveis (nunca quebra restrições), mas provavelmente muito ineficientes — mais rotas e mais distância, já que a ordem não teria lógica espacial ou temporal.

7. **Essa representação (giant-tour + split) é original do projeto ou vem da literatura?**
   Vem da literatura de VRP — é uma técnica de decodificação inspirada em trabalhos como o "Split algorithm" de Prins; o projeto usa uma versão simplificada (gulosa/sequencial), não a versão ótima.

---

## Slide 4 — Metodologia: "Quatro etapas"

**Conteúdo do slide:** CP1 Modelagem · CP2 Multiobjetivo · CP3 Diferencial · Final Avaliação.

### Explicação técnica (do fundamento ao avançado)

Projetos de pesquisa aplicada frequentemente adotam **entrega incremental**: em vez de implementar o sistema completo de uma vez, cada etapa entrega uma versão funcional e testável, sobre a qual a próxima etapa constrói. Esse projeto segue exatamente essa lógica, estruturada em quatro entregas alinhadas ao cronograma da disciplina.

**CP1 (Modelagem)** implementa a base compartilhada por tudo o que vem depois: a leitura da instância, a representação giant-tour, o split decoder, o cálculo dos objetivos, um baseline construtivo (Clarke-Wright) e um Algoritmo Genético single-objective (DEAP) para validar que a modelagem realmente produz soluções melhores que o baseline. **CP2 (Multiobjetivo)** substitui o GA escalarizado pelo NSGA-II (pymoo), que otimiza f1 e f2 simultaneamente, rodando por apenas 30 gerações — suficiente para demonstrar o conceito de fronteira de Pareto, mas não para competir em qualidade com o GA de 150 gerações do CP1. **CP3 (Diferencial)** introduz o warm-start: a mesma infraestrutura do NSGA-II, agora com 80 gerações e uma população inicial parcialmente semeada com soluções conhecidas. **Final (Avaliação)** não introduz nenhum algoritmo novo — reexecuta a pipeline completa e acrescenta o cálculo do Hypervolume, permitindo comparar objetivamente todas as frentes geradas nas etapas anteriores.

Cada etapa corresponde a um script independente em `scripts/` (`checkpoint1_modelagem.py`, `checkpoint2_multiobjetivo.py`, `checkpoint3_pre_final.py`, `checkpoint_final.py`), cada um salvando seus próprios artefatos (CSVs e figuras) em `results/`, prefixados por `cp1_`, `cp2_`, `cp3_` ou `final_`.

### Perguntas e Respostas

1. **Quantas gerações cada checkpoint rodou o NSGA-II?**
   CP2 rodou 30 gerações (fronteira inicial); CP3 e a entrega final rodaram 80 gerações (com e sem warm-start).

2. **O que muda entre CP1 e os demais em termos de algoritmo, não só de escopo?**
   CP1 usa um GA single-objective (DEAP, fitness escalarizado f1+50×f2); CP2 em diante usa o NSGA-II multiobjetivo (pymoo), que otimiza f1 e f2 simultaneamente sem agregação.

3. **A entrega "Final" adiciona algum algoritmo novo, ou só uma métrica?**
   Só uma métrica — o Hypervolume; a entrega final reexecuta a pipeline (`scripts/checkpoint_final.py`) e calcula HV para todas as frentes já geradas.

4. **O baseline (Clarke-Wright) é usado em quais etapas além do CP1?**
   É reutilizado no CP3/Final como uma das duas soluções-semente do warm-start, além de continuar servindo como referência de comparação em todas as tabelas de resultado.

5. **Por que o warm-start é chamado de "diferencial" e não apenas de mais uma etapa do checkpoint 3?**
   Porque, na estrutura de avaliação da disciplina, "diferenciais" são melhorias opcionais além do escopo mínimo exigido — o warm-start (D2) é uma contribuição extra, não uma exigência básica do checkpoint.

6. **Cada checkpoint gera seus próprios artefatos de forma isolada?**
   Sim — cada script em `scripts/` é independente e salva seus próprios arquivos em `results/`, prefixados por checkpoint.

7. **Por que não pular direto para o warm-start, economizando tempo de desenvolvimento?**
   Porque a validação incremental reduz risco — bugs na modelagem/split/objetivos são muito mais fáceis de detectar com um GA single-objective simples do que escondidos dentro de um NSGA-II com warm-start; a estrutura em camadas também segue o cronograma de entregas da disciplina.

---

## Slide 5 — Baseline: "Clarke & Wright — Savings"

**Conteúdo do slide:** fórmula savings(i,j) · 3 passos do algoritmo · limitação de orientação · resultado (556,5 / 10).

### Explicação técnica (do fundamento ao avançado)

Uma **heurística construtiva** monta uma solução do zero, passo a passo, sem iterar sobre uma população — em contraste com **metaheurísticas** (como GA e NSGA-II), que melhoram iterativamente um conjunto de soluções ao longo de "gerações". O algoritmo de **Clarke & Wright (1964)**, também chamado de "savings" (economias), é uma das heurísticas construtivas mais clássicas para VRP.

A lógica: comece assumindo o pior caso — um veículo dedicado para cada cliente, fazendo a rota trivial depósito→cliente→depósito. Em seguida, calcule, para cada par de clientes (i,j), a economia de fundir suas rotas em uma só: `savings(i,j) = d(0,i) + d(0,j) − d(i,j)`. O raciocínio geométrico por trás da fórmula é direto: `d(0,i)+d(0,j)` é o custo de visitar i e j em viagens separadas (ida e volta ao depósito duas vezes); `d(i,j)` é o custo extra de visitá-los em sequência numa única viagem. A diferença é, literalmente, quanto se economiza ao unir as duas rotas.

O algoritmo então ordena todos os pares por economia decrescente e tenta fundi-los nessa ordem, aceitando a fusão apenas se (a) a carga total da rota resultante não ultrapassar Q, e (b) a rota fundida continuar sendo viável em relação a todas as janelas de tempo dos clientes envolvidos (checado sequencialmente do início ao fim da rota). No código (`baseline.py`), há uma restrição adicional importante: a fusão só é tentada em **uma orientação fixa** — i precisa ser o último cliente da sua rota, e j o primeiro da rota dele (`route_i[-1] != i or route_j[0] != j` descarta a tentativa). O Clarke-Wright "completo" da literatura também testaria a orientação invertida das rotas antes de descartar uma fusão; a ausência dessa segunda tentativa é uma simplificação intencional e documentada, que torna este baseline mais fraco (mais conservador) do que o estado da arte do algoritmo. O resultado final na instância do projeto: 10 rotas, 556,51 de distância total.

### Perguntas e Respostas

1. **Por que savings(i,j) = d(0,i) + d(0,j) − d(i,j) mede uma "economia"?**
   Porque `d(0,i)+d(0,j)` é o custo de visitar i e j em rotas separadas, enquanto `d(i,j)` é o custo extra de visitá-los em sequência numa única rota; a diferença é quanto se economiza ao fundir.

2. **O savings pode ser negativo? O que isso significa?**
   Sim, quando i e j estão muito distantes entre si; um savings negativo significa que fundir as rotas pioraria a distância total, então essas fusões ficam no fim da lista ordenada e tipicamente nunca são aceitas.

3. **Quais são as duas condições que impedem uma fusão de ser aceita?**
   Se a rota fundida ultrapassar a capacidade Q, ou se ela deixar de ser viável em relação às janelas de tempo de qualquer cliente da rota resultante.

4. **Como o código garante que só se tenta fundir "o fim de uma rota com o início de outra"?**
   Em `baseline.py`, a fusão só é aceita se `route_i[-1] == i` e `route_j[0] == j` — ou seja, i precisa ser o último cliente da sua rota e j o primeiro da rota dele.

5. **Quantas rotas o baseline produz na instância usada, e qual é a distância total?**
   10 rotas (veículos) e distância total de aproximadamente 556,51.

6. **O Clarke-Wright é uma heurística construtiva ou uma metaheurística?**
   É uma heurística construtiva — monta uma única solução por um processo guloso determinístico, sem iterar/melhorar uma população de soluções como uma metaheurística faz.

7. **Depois de aceitar uma fusão, o algoritmo reconsidera fusões que dependiam da rota antiga?**
   Não — uma vez fundida, a rota_j deixa de existir e todos os seus clientes passam a apontar para a rota_i; qualquer savings pendente envolvendo extremidades que deixaram de ser extremidade simplesmente falha na checagem e é descartado.

---

## Slide 6 — Checkpoint 1: "Algoritmo Genético"

**Conteúdo do slide:** fitness = f1+50×f2 · OX · shuffle · torneio+elitismo · parâmetros · resultado de 5 sementes.

### Explicação técnica (do fundamento ao avançado)

Um **Algoritmo Genético** mantém uma **população** de soluções candidatas (aqui, permutações/giant-tours) e as evolui por gerações sucessivas usando três operadores inspirados na genética: **seleção** (escolher quem se reproduz, favorecendo os mais aptos), **crossover** (combinar dois pais para gerar filhos) e **mutação** (introduzir variação aleatória). Como este é o Checkpoint 1, o GA é **single-objective**: ele precisa de um único número para comparar soluções, então os dois objetivos reais do problema (f1, f2) são combinados artificialmente em uma **fitness escalarizada**: `fitness = f1 + 50 × f2`. Isso significa, na prática, que o algoritmo trata "um veículo a mais" como equivalente a "50 unidades extras de distância" — um peso de calibração arbitrário, escolhido na proposta do grupo.

Os operadores são escolhidos especificamente por preservarem a validade de uma permutação (nenhum cliente repetido ou faltando). O **crossover OX (Ordered Crossover)** copia um sub-trecho contíguo de um pai diretamente para o filho, e preenche as posições restantes com os genes do outro pai na ordem em que aparecem, pulando os que já foram usados — isso garante que o filho continue sendo uma permutação válida. A **mutação por shuffle de índices** (`mutShuffleIndexes`, `indpb=0.05`) troca aleatoriamente a posição de alguns clientes, com 5% de chance independente por posição, preservando a mesma invariante. A **seleção por torneio (k=3)** sorteia grupos de 3 indivíduos e escolhe o melhor de cada grupo para reproduzir — um meio-termo entre pressão seletiva forte e manutenção de diversidade. O **elitismo** garante que o melhor indivíduo de cada geração seja sempre copiado intacto para a próxima, para que o melhor fitness já encontrado nunca piore ao longo do tempo.

Com população 120 e 150 gerações, o GA foi executado com **5 sementes aleatórias diferentes (1 a 5)** — uma prática de validação que testa se a qualidade da solução é consistente ou depende de sorte na inicialização. A melhor semente (5) atingiu f1=266,08 com 4 veículos; a média das 5 sementes foi 306,0 ± 24,6 em f1, um desvio relativo de ~8%, indicando sensibilidade moderada à aleatoriedade inicial.

### Perguntas e Respostas

1. **Por que a penalização de veículo é multiplicativa (50×f2) e não uma função mais complexa?**
   Porque uma penalização linear simples é fácil de calibrar e suficiente para o propósito do checkpoint — o objetivo era ter um único critério escalar comparável entre indivíduos, não modelar o custo real de frota com precisão.

2. **O que são "cxpb" e "mutpb" nos parâmetros do GA?**
   `cxpb` (0.8) é a probabilidade de aplicar crossover a um par de indivíduos selecionados; `mutpb` (0.2) é a probabilidade de aplicar mutação a um indivíduo da prole.

3. **Como funciona o crossover OX na prática?**
   Escolhe um sub-trecho contíguo de um dos pais e o copia diretamente para o filho; o restante das posições é preenchido com os genes do outro pai, na ordem em que aparecem, pulando os já copiados — garantindo que nenhum cliente se repita.

4. **Por que a mutação usa "shuffle de índices" em vez de substituir um cliente por outro aleatório?**
   Porque trocar (swap) duas posições preserva a propriedade de permutação; substituir um gene por um valor aleatório poderia duplicar um cliente e quebrar essa invariante.

5. **Qual é a taxa de mutação por gene (indpb), e o que ela significa?**
   `indpb=0.05` — cada posição da permutação tem 5% de chance independente de participar de uma troca durante a mutação de um indivíduo selecionado para mutar.

6. **Como o elitismo é implementado no laço principal do GA?**
   A cada geração, `toolbox.select` escolhe `pop_size-1` indivíduos (abrindo uma vaga), e ao final o melhor indivíduo da geração anterior é anexado de volta à nova população, garantindo que ele nunca se perca.

7. **A semente 3 e a semente 4 tiveram f2=5 enquanto as demais tiveram f2=4 — isso é esperado?**
   Sim — como f2 entra apenas como penalização (não como restrição rígida), diferentes execuções podem convergir para ótimos locais distintos com números de veículos ligeiramente diferentes; é justamente esse tipo de variação que a análise de 5 sementes busca capturar.

---

## Slide 7 — Checkpoint 2: "NSGA-II multiobjetivo"

**Conteúdo do slide:** otimiza f1,f2 sem agregar · dominância · fronts · crowding · resultado 30 ger. (HV=1.280,7).

### Explicação técnica (do fundamento ao avançado)

O NSGA-II (Non-dominated Sorting Genetic Algorithm II) é a adaptação de um algoritmo genético para lidar com **múltiplos objetivos simultaneamente**, sem reduzi-los a um número artificial. O conceito central é a **dominância de Pareto**: uma solução A domina uma solução B se A é igual ou melhor que B em todos os objetivos, e estritamente melhor em pelo menos um. Uma solução é **não-dominada** se nenhuma outra solução da população a domina. O conjunto de todas as soluções não-dominadas é a **fronteira de Pareto** — representa todos os trade-offs "competitivos" entre os objetivos.

O NSGA-II usa dois mecanismos complementares para conduzir a busca. O primeiro é o **fast non-dominated sorting**: para cada solução, conta-se quantas outras a dominam; as com contagem zero formam o "front 1" (a melhor camada); removendo-as e repetindo o processo nas remanescentes, forma-se o "front 2", e assim sucessivamente. Isso controla a **convergência** — indivíduos de fronts melhores têm prioridade na sobrevivência e reprodução. O segundo mecanismo é a **crowding distance**: dentro de um mesmo front, mede o quão "espaçada" uma solução está de suas vizinhas mais próximas em cada objetivo (soluções nos extremos do front recebem distância infinita, para serem sempre preservadas). Isso controla a **diversidade** — evita que a população colapse em um único aglomerado da fronteira, garantindo que ela permaneça bem distribuída ao longo de todo o espectro de trade-off.

No código (`nsga2.py`), o problema é definido como uma classe `Problem` do pymoo com `n_obj=2` (f1, f2), usando os operadores nativos de permutação da biblioteca: `OrderCrossover` (equivalente conceitual ao OX do DEAP) e `InversionMutation` (inverte um sub-trecho contíguo, ao invés de trocar posições isoladas). Rodando por apenas 30 gerações neste checkpoint, a fronteira resultante (3 soluções não-dominadas, HV=1.280,7) ainda é claramente inferior ao ponto único que o GA encontrou com 150 gerações — um resultado esperado e honestamente reportado, já que o orçamento de gerações do NSGA-II aqui é 5 vezes menor, e ele precisa descobrir E espalhar uma fronteira inteira, não apenas convergir para um único ponto.

### Perguntas e Respostas

1. **Dado A=(f1=300,f2=5) e B=(f1=310,f2=5), A domina B?**
   Sim — A é igual a B em f2 e estritamente melhor em f1, então A domina B (B é descartável).

2. **Dado A=(f1=300,f2=6) e B=(f1=310,f2=5), A domina B?**
   Não — A é melhor em f1 mas pior em f2; nenhum domina o outro, ambos pertencem à mesma fronteira não-dominada (se não houver uma terceira solução que domine ambos).

3. **Como o "fast non-dominated sorting" atribui os fronts?**
   Para cada solução, conta-se quantas outras a dominam; as com contagem zero vão para o front 1; remove-se o front 1 e repete-se o processo nas remanescentes para formar o front 2, e assim sucessivamente.

4. **O que exatamente a crowding distance soma para cada solução?**
   Para cada objetivo, a distância (normalizada pelo intervalo do objetivo naquele front) entre os dois vizinhos mais próximos da solução; soma-se essa distância através de todos os objetivos.

5. **Soluções nas extremidades de um front recebem qual valor de crowding distance, e por quê?**
   Distância infinita, para serem sempre preservadas prioritariamente — isso mantém os extremos da fronteira (soluções mais especializadas em um único objetivo) na população.

6. **Quais operadores de permutação o pymoo usa no NSGA-II deste projeto?**
   `OrderCrossover` (equivalente ao OX do DEAP) e `InversionMutation` (inverte um sub-trecho contíguo da permutação) — diferente do shuffle de índices do GA, mas com o mesmo objetivo de preservar a validade da permutação.

7. **Por que "eliminate_duplicates=True" é relevante para o NSGA-II neste problema?**
   Sem essa opção, indivíduos idênticos (mesma permutação) poderiam se acumular na população, desperdiçando espaço populacional e reduzindo a diversidade efetiva de busca — a opção remove/substitui duplicatas.

---

## Slide 8 — Diferencial D2: "Warm-start"

**Conteúdo do slide:** 25% sementes / 75% aleatório · fontes das sementes · evolução de f1 em 80 gerações.

### Explicação técnica (do fundamento ao avançado)

**Warm-start** é uma técnica de inicialização: em vez de começar a busca a partir de uma população 100% aleatória, parte dela é "semeada" com soluções que já se sabe serem boas, na esperança de que o algoritmo converja mais rápido por já começar de uma posição vantajosa no espaço de busca. É uma técnica geral de otimização (aplicável a qualquer metaheurística populacional), e aqui é a contribuição própria do grupo sobre o NSGA-II padrão.

A implementação vive na classe `SeededPermutationSampling` (`nsga2.py`), que sobrescreve a etapa de amostragem inicial do pymoo. Ela recebe uma lista de soluções-semente (aqui, duas: a rota do baseline Clarke-Wright e a melhor solução do GA, seed 5, f1=266) e uma fração `seed_frac=0.25`. Ao gerar a população inicial, primeiro cria `n_samples` indivíduos totalmente aleatórios (via `PermutationRandomSampling`), depois substitui os primeiros `n_seed = int(n_samples * seed_frac)` deles pelas soluções-semente, alternando entre elas por índice módulo (`i % len(seed_perms0)`). O restante (75%) permanece aleatório, preservando diversidade genética suficiente para a busca continuar explorando regiões não cobertas pelas sementes.

Um ponto técnico importante: as soluções-semente **não recebem tratamento especial após a geração 0** — elas são reavaliadas normalmente pela função objetivo, competem pela sobrevivência através dos mesmos critérios de dominância/crowding, e podem ser substituídas por descendentes melhores nas gerações seguintes, exatamente como qualquer outro indivíduo. O efeito observado é dramático: enquanto a população aleatória começa a busca com f1≈686 (pior indivíduo aleatório encontrado), a população com warm-start já começa em f1≈266 — o mesmo valor da melhor semente — e ambas rodam pelas mesmas 80 gerações, isolando o efeito da inicialização como variável experimental controlada.

### Perguntas e Respostas

1. **Como o código decide quais indivíduos da população recebem uma solução-semente?**
   Em `SeededPermutationSampling._do()`, os primeiros `n_seed = int(n_samples * seed_frac)` índices da matriz de população recebem uma das soluções-semente (alternando entre elas por módulo), e os demais mantêm a amostragem aleatória.

2. **Se há duas soluções-semente e n_seed=30, quantos indivíduos recebem cada semente?**
   Elas se alternam pelo índice módulo o número de sementes (`i % len(seed_perms0)`) — com 2 sementes e 30 posições, aproximadamente 15 indivíduos recebem cada uma.

3. **As soluções-semente entram com f1/f2 já calculados, ou são reavaliadas pelo NSGA-II?**
   São reavaliadas normalmente — o NSGA-II roda `evaluate()` sobre cada indivíduo da população inicial (semeado ou aleatório) da mesma forma, sem atalhos.

4. **O warm-start altera os operadores de crossover/mutação do NSGA-II?**
   Não — ele altera apenas a amostragem da população inicial (geração 0); a partir da geração 1, os mesmos operadores (`OrderCrossover`, `InversionMutation`) atuam sobre toda a população, sementes incluídas.

5. **Depois de várias gerações, as soluções originais de semente ainda existem na população?**
   Elas competem normalmente pela sobrevivência via seleção NSGA-II; podem ser preservadas (se ainda estiverem no front 1) ou substituídas por descendentes melhores — não há proteção especial além da qualidade delas mesmas.

6. **Por que a rota do baseline (mais fraca, f1=556) ainda é útil como semente, já que a do GA (f1=266) é muito melhor?**
   Porque ela oferece um ponto de partida estruturalmente diferente (mais veículos, rotas mais curtas), ajudando a diversificar a região inicial da busca em vez de concentrar tudo perto de uma única solução, o que poderia prejudicar a crowding distance.

7. **O ganho de desempenho do warm-start seria o mesmo em uma instância com 200 clientes em vez de 25?**
   Não necessariamente — não foi testado (limitação documentada). Em instâncias maiores, o espaço de busca cresce muito mais, e o benefício relativo de semear 25% da população com apenas 2 soluções pode diminuir ou exigir ajuste dessa fração.

---

## Slide 9 — Métrica: "Hypervolume"

**Conteúdo do slide:** área dominada até r=(750,12) · HV em 30/80 gen sem WS / 80 gen com WS · +104%.

### Explicação técnica (do fundamento ao avançado)

Comparar duas **fronteiras** de Pareto (conjuntos de soluções, não pontos únicos) é mais difícil do que comparar dois números — uma fronteira pode ter mais soluções mas ser pior em qualidade, ou menos soluções mas mais próxima do ótimo. O **Hypervolume (HV)** resolve esse problema resumindo toda a fronteira em um único número: é a medida (em 2D, uma área; em mais dimensões, um volume) da região do espaço de objetivos que é **dominada** pela fronteira, delimitada por um **ponto de referência** fixo, escolhido para ser pior que qualquer solução observada nos experimentos.

Formalmente, HV(F, r) = medida da união das "caixas" formadas entre cada ponto da fronteira F e o ponto de referência r. Quanto mais próximas do ótimo E mais bem distribuídas as soluções da fronteira, maior a área capturada — por isso o HV consegue capturar **simultaneamente** convergência (proximidade do ótimo) e diversidade (espalhamento), diferente de métricas mais simples. O projeto usa `pymoo.indicators.hv.HV`, a mesma biblioteca do NSGA-II, com ponto de referência **r=(750, 12)** — escolhido por ser pior que o pior f1 observado (~686) e o pior f2 observado (10) em qualquer execução. Um ponto de referência mal escolhido (dominado por alguma solução real) invalidaria o cálculo; um ponto de referência consistente entre execuções é o que torna os valores de HV diretamente comparáveis entre si.

Os três números-chave do slide: HV=1.280,7 (NSGA-II, 30 gerações, sem warm-start — CP2); HV=1.901,8 (NSGA-II, 80 gerações, sem warm-start); HV=3.871,4 (NSGA-II, 80 gerações, **com** warm-start). A comparação isolando corretamente o efeito do warm-start é entre as duas últimas (mesmo número de gerações, 80), resultando no ganho de **+103,6%** (arredondado para "+104%") citado no slide — o warm-start mais que dobrou a área de objetivos capturada pela fronteira, para exatamente o mesmo orçamento computacional de gerações.

### Perguntas e Respostas

1. **Hypervolume é calculado sobre quantas dimensões neste projeto?**
   Duas — f1 (distância) e f2 (número de veículos), já que `use_f3_ethics` permanece desativado por padrão.

2. **Por que HV=0 seria o pior resultado possível?**
   Significaria que nenhuma solução da fronteira domina nenhuma região do espaço de objetivos em relação ao ponto de referência — na prática, ocorreria apenas se a fronteira estivesse vazia.

3. **A biblioteca usada para calcular o HV é a mesma usada para o NSGA-II?**
   Sim — `pymoo.indicators.hv.HV`, da mesma biblioteca pymoo usada para o algoritmo NSGA-II.

4. **O ponto de referência (750,12) é o mesmo em todas as comparações (CP2, CP3 cold, CP3 warm)?**
   Sim — usar o mesmo ponto de referência em todas as execuções é o que torna os valores de HV diretamente comparáveis entre elas.

5. **O HV cresce sempre que se roda mais gerações?**
   Na prática tende a crescer ou estabilizar (nunca diminui em teoria, pois o NSGA-II preserva as melhores soluções via elitismo), mas o ganho marginal por geração normalmente diminui conforme a fronteira se aproxima do ótimo.

6. **Dois HVs iguais implicam fronteiras idênticas?**
   Não necessariamente — fronteiras com formatos ou números de soluções diferentes podem dominar a mesma área total; HV resume qualidade, não identifica unicamente a forma da fronteira.

7. **Por que a comparação de HV entre CP2 (30 ger.) e o NSGA-II final com warm-start (80 ger.) não isola o efeito do warm-start?**
   Porque, além do warm-start, também mudou o número de gerações; para isolar especificamente o efeito do warm-start, a comparação controlada correta é entre os dois resultados de 80 gerações (com e sem warm-start: 3.871,4 vs. 1.901,8), que é o que gera o +103,6% reportado.

---

## Slide 10 — Resultados: "Comparação final"

**Conteúdo do slide:** tabela consolidada (baseline, GA médio/melhor, NSGA-II 30/80 gen, NSGA-II+warm-start).

### Explicação técnica (do fundamento ao avançado)

Este slide consolida, numa única tabela, todas as abordagens implementadas no projeto — funcionando como o resumo quantitativo de toda a apresentação. É importante entender o que cada coluna realmente representa: **f1** e **f2** são os valores objetivos (mínimos, no caso das fronteiras) de cada método; **HV** só existe para métodos que produzem uma fronteira de soluções (o baseline e o GA produzem cada um uma única solução/ponto, então HV não se aplica a eles da mesma forma).

Um detalhe sutil, presente em `results/summary.json` mas não explícito no slide: a fronteira "NSGA-II 80 gen sem warm-start" tem **2 soluções não-dominadas** (f1_min=420,54, f2_min=6), enquanto a fronteira "NSGA-II 80 gen + warm-start" tem **apenas 1 solução não-dominada** (f1=266,08, f2=4) — coincidentemente (ou não, já que é uma das sementes) o mesmo ponto que o GA já havia encontrado. Isso é relevante para interpretar corretamente a frase final do slide ("fronteira de Pareto completa — o decisor escolhe o trade-off"): conceitualmente, essa é a promessa do método (NSGA-II devolve um conjunto, não um ponto), mas, nesta execução específica com warm-start, a fronteira final colapsou para um único ponto ótimo — uma limitação real que vale reconhecer proativamente se questionada.

A leitura mais defensável da tabela é: (1) cada camada de sofisticação algorítmica melhorou a solução sobre a anterior (baseline → GA → NSGA-II); (2) o NSGA-II com warm-start alcançou o mesmo ponto ótimo do GA (f1=266,1, f2=4), mas chegou lá através de uma metodologia que, em princípio, também mapeia trade-offs alternativos — mesmo que, nesta rodada específica, a fronteira tenha convergido para um único ponto em vez de se manter espalhada.

### Perguntas e Respostas

1. **Por que a linha do baseline e as do GA não têm valor de HV na tabela?**
   Porque HV é uma métrica de qualidade de FRONTEIRA; baseline e GA produzem cada um uma única solução/ponto, não uma fronteira, então HV não se aplica diretamente a eles da mesma forma.

2. **A faixa "483,7–524,0" do NSGA-II 30 gerações representa o quê exatamente?**
   É o intervalo de f1 entre as soluções da fronteira de 30 gerações — a menor e a maior distância total entre as 3 soluções não-dominadas encontradas nesse checkpoint.

3. **Os números de f1 e f2 do "NSGA-II 80 gen + warm-start" (266,1 / 4) são idênticos aos do "GA melhor" — isso é coincidência?**
   Não — a melhor solução do GA foi literalmente usada como uma das soluções-semente do warm-start, então é esperado que ela reapareça como parte ou totalidade da fronteira final.

4. **A linha "NSGA-II 80 gen" (sem warm-start) tem HV=1.902 com quantas soluções na fronteira?**
   2 soluções, segundo o `summary.json` (n_solutions=2), com f1 mínimo de 420,54 e f2 mínimo de 6.

5. **Que conclusão prática se tira comparando a linha do baseline com a última linha da tabela?**
   Que a combinação de metaheurística + warm-start reduziu a distância total em quase 52% (de 556,5 para 266,1) e o número de veículos de 10 para 4, mantendo viabilidade total das rotas.

6. **A frase final do slide ("o decisor escolhe o trade-off") se sustenta totalmente pelos dados desta tabela específica?**
   Parcialmente — é a mensagem pretendida do método, mas, como a fronteira final com warm-start colapsou para 1 única solução, essa execução específica não demonstra múltiplas opções de trade-off; a afirmação é conceitualmente correta sobre o NSGA-II em geral, mas os dados desta rodada são uma exceção que merece nota.

7. **Se pedirem para comparar o custo computacional das diferentes abordagens, o que responder?**
   O baseline Clarke-Wright é o mais barato (heurística construtiva, roda em milissegundos); o GA (150 ger., pop 120) e o NSGA-II (80 ger., pop 120, 2 objetivos por indivíduo) têm custo comparável ou o NSGA-II um pouco maior por geração — o relatório não reporta tempos de execução exatos, o que é uma limitação a reconhecer se perguntada com precisão.

---

## Slide 11 — Conclusões: "Contribuições"

**Conteúdo do slide:** 4 contribuições listadas · trade-off final.

### Explicação técnica (do fundamento ao avançado)

O slide de conclusões resume o projeto em quatro afirmações, e é útil, do ponto de vista de defesa, separar o que é **aplicação rigorosa de técnica conhecida** do que é **contribuição própria do grupo**. A modelagem via giant-tour + split decoder é uma técnica estabelecida na literatura de VRP (não inventada pelo grupo, mas corretamente implementada e validada). A comparação entre heurística construtiva, GA e NSGA-II é um exercício comparativo rigoroso, mas metodologicamente descritivo — não há testes estatísticos formais de significância, apenas médias e desvios-padrão sobre 5 sementes (só para o GA; o NSGA-II não teve a mesma análise de variância entre sementes). A métrica de Hypervolume é uma ferramenta padrão da área, aplicada corretamente com um ponto de referência consistente. O item genuinamente autoral do grupo é o **warm-start**: a estratégia específica de misturar 25% de soluções-semente (Clarke-Wright + melhor GA) com 75% de população aleatória como inicialização do NSGA-II — uma ideia simples, de baixo custo de implementação (~15 linhas de código adicionais), mas com o maior impacto mensurável de todo o projeto (+103,6% em HV).

A frase final ("menos veículos implica rotas mais longas — e o NSGA-II mapeia esse trade-off inteiro") reafirma a tese central: é uma tendência geral e esperada em VRP, fundamentada na geometria do problema, mas a formulação "os dados confirmam" refere-se especificamente a esta instância (C101, 25 clientes) — generalização para outras instâncias não foi testada, e essa é uma das limitações explicitamente reconhecidas no relatório técnico (junto com o Clarke-Wright simplificado, o uso de uma única instância, e a ausência de hibridização com busca local).

### Perguntas e Respostas

1. **Das quatro contribuições listadas, qual tem base mais direta na literatura clássica?**
   A "modelagem completa via giant-tour + split decoder" — é uma técnica de representação conhecida na literatura de VRP (ex: Split de Prins), aplicada e implementada pelo grupo, mas não inventada por eles.

2. **A "comparação rigorosa de 3 abordagens" inclui alguma análise estatística formal?**
   Não — a análise é descritiva (médias, desvios-padrão em 5 sementes para o GA), sem testes estatísticos formais de significância; é uma limitação a reconhecer se questionada.

3. **O ganho de "+104% em hypervolume" citado nas conclusões é o mesmo número calculado no Slide 9?**
   Sim, é o mesmo resultado (103,6%, arredondado), comparando NSGA-II 80 gerações com e sem warm-start.

4. **A conclusão do trade-off é uma lei geral do VRPTW ou uma observação empírica deste experimento?**
   É uma tendência geral e esperada em VRP (fundamentada na estrutura geométrica do problema), mas a formulação apresentada refere-se especificamente às observações empíricas desta instância (C101, 25 clientes) — não foi provada formalmente nem testada em outras instâncias.

5. **As limitações mencionadas no relatório técnico incluem quais pontos?**
   Clarke-Wright simplificado (uma orientação só), única instância testada (C101/25 clientes), f3 implementado mas não usado nos resultados principais, e ausência de hibridização memética (NSGA-II + busca local), mencionada na proposta original mas não implementada.

6. **O projeto testa a robustez do NSGA-II (com e sem warm-start) em múltiplas sementes, como fez com o GA?**
   Não da mesma forma extensiva — os experimentos-chave do NSGA-II usam sementes fixas por execução (1 e 10), sem uma análise de variância entre múltiplas sementes equivalente às 5 sementes do GA; é uma lacuna metodológica a reconhecer.

7. **O que o grupo faria de diferente se recomeçasse o projeto agora?**
   Testar em mais instâncias Solomon (tamanhos e famílias R/RC/C diferentes), rodar múltiplas sementes para o NSGA-II também, implementar a versão completa do Clarke-Wright (ambas orientações), e explorar a hibridização com busca local (2-opt) mencionada na proposta original, mas não implementada por restrição de tempo.

---

*Total: 11 slides × 7 perguntas = 77 perguntas e respostas de treino. Recomendação de uso: leia a explicação técnica de um slide, feche o documento, e tente responder as 7 perguntas de memória antes de conferir — é o mesmo princípio de active recall usado no material de estudo interativo (`docs/estudo_interativo.html`).*
