# O projeto explicado sem jargão

*Como se você estivesse contando para um amigo que nunca programou na vida.*

## Em uma frase

O grupo pegou um problema clássico de logística — **"como organizar os caminhões de entrega para atender todo mundo no horário certo, rodando o mínimo possível e usando o mínimo de caminhões"** — e usou técnicas inspiradas na evolução das espécies para encontrar boas soluções, mesmo sem poder testar todas as possibilidades.

---

## 1. O problema, contado como uma história

Imagine uma empresa de entregas com:

- **Um único depósito**, de onde todos os caminhões saem e para onde todos precisam voltar no fim do dia.
- **25 clientes** espalhados pela cidade, cada um esperando um pacote.
- **Uma frota de caminhões iguais**, cada um com um limite de quanto consegue carregar.
- **Um horário combinado com cada cliente**: se o caminhão chegar cedo demais, ele espera; se chegar tarde demais, já era — não pode entregar fora do combinado.

A pergunta que o projeto responde é: **em que ordem cada caminhão deve visitar seus clientes, e quantos caminhões usar, para gastar o mínimo de quilômetros possível?**

Parece simples, mas repare que dá pra organizar essas entregas de *milhões* de jeitos diferentes — e a maioria deles é ruim (caminhões cruzando a cidade sem necessidade, indo e voltando à toa).

## 2. Por que um computador não "simplesmente calcula" a resposta perfeita

Você pode pensar: "um computador é rápido, por que não testa todas as combinações e escolhe a melhor?"

O problema é que o número de combinações **explode**. Não é como testar 100 ou 1.000 opções — é mais parecido com tentar abrir um cadeado de bicicleta girando *todas* as combinações possíveis de um cadeado com 25 rodinhas: o número de tentativas fica tão absurdamente grande que nem o computador mais rápido do mundo, ligado por anos, conseguiria testar tudo.

Esse tipo de problema é chamado, na área, de **"difícil de resolver com certeza absoluta em tempo razoável"**. Então, em vez de procurar *a* resposta perfeita, o projeto usa estratégias inteligentes para encontrar respostas **muito boas, rapidamente** — mesmo sem a garantia de que são as melhores possíveis do universo.

## 3. O verdadeiro desafio: dois objetivos que puxam em direções opostas

Aqui está o coração do projeto. A empresa quer, ao mesmo tempo:

1. **Rodar o menos possível** (menos quilômetros = menos combustível, menos tempo).
2. **Usar o menor número de caminhões possível** (cada caminhão a mais custa motorista, manutenção, aluguel).

O problema é que essas duas coisas **brigam entre si**: se você usa menos caminhões, cada um precisa visitar mais clientes sozinho — ou seja, roda mais. Se você quer que cada caminhão rode pouco, precisa espalhar os clientes entre mais caminhões.

Não existe uma resposta única que seja "a melhor nos dois quesitos ao mesmo tempo". Existe, na verdade, um **conjunto de boas soluções**, cada uma com um equilíbrio diferente — tipo escolher entre um carro mais barato ou um carro mais rápido: não existe "o melhor carro do mundo", existe um leque de opções boas, cada uma com sua vantagem.

## 4. Como o grupo resolveu isso, passo a passo

### Passo 1 — Um ponto de partida simples (o "baseline")

Primeiro, implementaram uma estratégia clássica e simples, de décadas atrás: comece com um caminhão para cada cliente, e vá **juntando rotas de dois em dois**, sempre que juntá-las economiza distância — desde que isso não estoure a capacidade do caminhão nem os horários combinados.

Isso já dá uma solução razoável, mas longe de ser ótima: nesse caso, precisou de **10 caminhões** e rodou cerca de **556 km** no total. É o "chão" — a referência mínima que qualquer coisa melhor precisa superar.

### Passo 2 — Evolução artificial (Algoritmo Genético)

Depois, o grupo usou uma técnica inspirada em como a natureza melhora espécies ao longo de gerações: **seleção natural simulada em um computador**.

A ideia:
- Crie um monte de soluções aleatórias (como "famílias" de rotas diferentes).
- A cada rodada ("geração"), pegue as melhores soluções e **misture** elas entre si (como cruzar duas "receitas" boas para tentar criar uma ainda melhor), e de vez em quando introduza uma mudança aleatória pequena (uma "mutação").
- Repita esse processo centenas de vezes.
- Sempre guarde a melhor solução encontrada até agora, para nunca perder o progresso.

Depois de 150 "gerações" de evolução, a melhor solução encontrada rodou apenas **~266 km** com **4 caminhões** — quase **45% menos distância** que o ponto de partida simples, e 6 caminhões a menos. Um salto de qualidade enorme.

*Limitação desse passo:* essa técnica só devolve **uma única resposta**, porque ela precisa transformar os dois objetivos (distância e número de caminhões) em um único número artificial para poder comparar soluções. É como decidir de antemão "cada caminhão a mais vale exatamente 50 km" — uma escolha arbitrária que esconde todas as outras opções possíveis de equilíbrio.

### Passo 3 — Explorando o leque inteiro de opções (NSGA-II)

Para não ter que inventar esse número artificial, o grupo usou uma versão mais sofisticada da mesma ideia evolutiva, chamada **NSGA-II**, que consegue lidar com **os dois objetivos ao mesmo tempo**, sem misturá-los em um só número.

Em vez de devolver uma única resposta, ela devolve **um cardápio de boas soluções** — cada uma com um equilíbrio diferente entre "poucos quilômetros" e "poucos caminhões" — e cabe ao gestor da empresa escolher qual trade-off prefere.

Rodando por pouco tempo (30 "gerações"), o resultado ainda era fraco — esperado, já que é bem mais difícil buscar um cardápio inteiro de boas opções do que buscar uma única resposta.

### Passo 4 — A ideia própria do grupo: dar um "empurrãozinho" inicial (Warm-start)

Aqui está a contribuição original do grupo. Em vez de começar a busca do zero, **totalmente aleatória**, eles pegaram um quarto (25%) das soluções iniciais e já começaram elas a partir de **soluções que já sabiam ser boas** (a do passo 1 e a melhor do passo 2). Os outros 75% continuaram aleatórios, para não perder a capacidade de explorar coisas novas.

É como numa corrida de revezamento: em vez de todo mundo largar do zero, um quarto dos corredores já começa alguns quilômetros mais à frente, usando um "atalho" que já sabemos que funciona — mas o resto da equipe ainda corre a prova inteira, para garantir variedade de estratégias.

O resultado foi impressionante: com o mesmo tempo de busca, a qualidade da fronteira de opções encontrada **mais que dobrou** — um ganho de mais de **100%** em relação a começar do zero.

## 5. Como eles mediram "melhor", de forma justa

Para comparar "cardápios de soluções" diferentes (não só números únicos), o grupo usou uma métrica chamada **Hypervolume**. A ideia é simples de visualizar: imagine um mapa onde um eixo é "quilômetros rodados" e o outro é "número de caminhões". Cada solução encontrada é um ponto nesse mapa. O Hypervolume mede **quanta área desse mapa uma coleção de soluções consegue "conquistar"**, partindo de um canto ruim de referência.

Quanto maior essa área conquistada, melhor o conjunto de soluções — porque significa que ele tem opções mais próximas do ideal **e** bem distribuídas entre os dois objetivos, não só um ponto isolado de sorte.

## 6. Resultado final, resumido

| Estratégia usada | Quilômetros rodados | Caminhões usados |
|---|---|---|
| Ponto de partida simples (Passo 1) | ~556 km | 10 |
| Evolução simples, uma só resposta (Passo 2) | ~266 km | 4 |
| Evolução com dois objetivos, pouco tempo (Passo 3) | ~484–524 km | 7 a 9 |
| Evolução com dois objetivos + empurrãozinho inicial (Passo 4) | ~266 km | 4 |

A última linha é a vencedora: ela chega ao **mesmo resultado numérico** do Passo 2, mas com uma vantagem enorme — em vez de uma única resposta, ela entrega **um leque inteiro de boas alternativas** ao redor desse resultado, para quem for tomar a decisão escolher o equilíbrio que preferir.

## 7. Por que isso importa na vida real

Esse não é um problema acadêmico abstrato — é o mesmo tipo de decisão que empresas de entrega (iFood, Correios, transportadoras, ônibus escolares) tomam todos os dias: **quantos veículos colocar na rua, e em que ordem eles visitam cada endereço**, equilibrando custo de combustível/tempo contra custo de ter mais veículos e motoristas.

O projeto mostra, com números reais, que:
- Um algoritmo simples e clássico já resolve o problema de forma razoável, mas longe do ideal.
- Técnicas inspiradas em evolução conseguem melhorar isso drasticamente.
- Reaproveitar conhecimento de uma etapa para "dar um empurrão" na próxima (o warm-start) é uma forma barata e muito eficaz de acelerar a busca por boas soluções.
- Em vez de forçar uma resposta única e artificial, é mais honesto (e mais útil na prática) entregar um leque de opções e deixar quem decide escolher o trade-off.

## 8. O resumo de 30 segundos, para responder "e aí, o que você fez?"

> "A gente pegou o problema de organizar as rotas de uma frota de entrega — minimizando quilômetros rodados e número de caminhões ao mesmo tempo, mesmo esses dois objetivos brigando entre si. Usamos uma técnica inspirada em evolução das espécies para 'evoluir' rotas cada vez melhores, e criamos um truque próprio de 'dar um empurrãozinho' inicial ao algoritmo usando soluções que já sabíamos ser boas — isso mais que dobrou a qualidade dos resultados, para o mesmo tempo de busca."
