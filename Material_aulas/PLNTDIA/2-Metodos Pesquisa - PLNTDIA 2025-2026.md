# Métodos de Pesquisa e Otimização

**Unidade Curricular:** Planeamento e Tomada de Decisão (PLNTDIA)  
**Mestrado:** Mestrado em Engenharia de Inteligência Artificial (MEIA) — ISEP/IPP  
**Docentes:** Carlos Ramos, Goreti Marreiros (`{csr,mgt}@isep.ipp.pt`)  
**Ano Letivo:** 2025-2026  

---

## 1. Introdução aos Métodos de Pesquisa

Na resolução automática de problemas articulam-se os seguintes conceitos essenciais:
- **Mundo do Problema e Espaço de Estados:** Modelação de estados e transições.
- **Restrições e Objetivos:** Condicionalismos (*hard* e *soft*) e funções de custo/lucro.
- **Complexidade e Otimização:** Explosão combinatória vs. procura do melhor valor.
- **Pesquisa Não Informada vs. Informada:** Uso ou ausência de conhecimento heurístico.

---

## 2. Branch & Bound

### 2.1 Princípios de Funcionamento
- O **Branch and Bound** (Ramificar e Limitar) baseia-se no princípio do *"Primeiro o Melhor"* com avaliação de transições pelo custo acumulado $g(n)$ desde a raiz.
- **Comutação Dinâmica:** Permite comutar a qualquer momento para outro nó folha da árvore de pesquisa que apresente menor custo acumulado, mesmo que não seja descendente do nó acabado de expandir.
- **Avaliação:** Nós candidatos à expansão são comparados com base no valor acumulado do custo (ou ganho) desde a raiz.

### 2.2 Exemplo: Autoestradas da Austrália (Brisbane $\to$ Darwin)

```
        DARWIN
       /      \
 989  /        \ 3848
     /          \
TENNANT_CREEK    PERTH
  |        \      /
  | 2042    \    / 2695
  |          \  /
  |           ADELAIDE
  | 2487     /   |
  |         /    | 727
  |        /     |
BRISBANE  1159  MELBOURNE
  |  \           / 727
  |   \ 1776    /
  |    \       /
 917    CANBERRA
  |      / 286
 SYDNEY /
```
*(Outra ligação: Brisbane $\to$ Cairns: 1682 km)*

#### Passo a Passo da Exploração:
1. **Raiz:** `Brisbane (0)`
   - Expansão gera:
     - `Sydney (917)`
     - `Cairns (1682)`
     - `Melbourne (1776)`
     - `Tennant Creek (2487)`
2. **Menor custo acumulado:** `Sydney (917)`
   - Expansão de `Sydney`:
     - `Canberra (917 + 286 = 1203)`
3. **Menor custo acumulado:** `Canberra (1203)`
   - Expansão de `Canberra`:
     - `Melbourne (1203 + 727 = 1930)`
     - `Adelaide (1203 + 1159 = 2362)`
4. **Menor custo acumulado:** `Cairns (1682)`
   - Não tem ligações não visitadas $\implies$ nó sem saída.
5. **Menor custo acumulado:** `Melbourne (1776)` *(ramo direto de Brisbane)*
   - Expansão de `Melbourne`:
     - `Adelaide (1776 + 727 = 2503)`
     - `Canberra (1776 + 727 = 2503)`
6. **Menor custo acumulado:** `Melbourne (1930)` *(via Sydney-Canberra)*
   - Expansão de `Melbourne`:
     - `Adelaide (1930 + 727 = 2657)`
7. **Menor custo acumulado:** `Adelaide (2362)` *(via Sydney-Canberra)*
   - Expansão de `Adelaide`:
     - `Melbourne (2362 + 727 = 3089)`
     - `Tennant Creek (2362 + 2042 = 4404)`
     - `Perth (2362 + 2695 = 5057)`
8. **Menor custo acumulado:** `Tennant Creek (2487)` *(direto de Brisbane)*
   - Expansão de `Tennant Creek`:
     - **`Darwin (2487 + 989 = 3476)`** $\to$ **Solução encontrada!**
     - `Adelaide (2487 + 2042 = 4529)`

> **Condição de Paragem para Garantir a Solução Ótima:**  
> Atingir uma solução não é suficiente para parar. **Só podemos garantir que temos a melhor solução quando todos os nós folha por expandir tiverem um custo acumulado igual ou superior ao da melhor solução encontrada até ao momento** (ou seja, $\ge 3476$).  
> Como todos os restantes nós folha têm custo $> 3476$, confirma-se que $3476\text{ km}$ é a solução ótima.

---

## 3. Algoritmo A\*

### 3.1 Fundamentos
O algoritmo **A\*** combina:
1. O uso de **custos acumulados conhecidos** do *Branch and Bound* ($g(n)$);
2. A estimativa heurística de distância ao objetivo do *Primeiro o Melhor* ($h'(n)$);
3. A capacidade de comutar entre ramos em aberto na fronteira de pesquisa.

### 3.2 Função de Avaliação
$$f'(n) = g(n) + h'(n)$$
- **$g(n)$:** Custo real despendido desde o estado inicial até ao nó atual $n$ (conhecido com exatidão).
- **$h'(n)$:** Estimativa do custo restante do nó $n$ até à meta (estimativa sujeita a erro).
  - À medida que nos aproximamos da meta, $h'(n) \to 0$. Na meta, $h'(\text{meta}) = 0$.

### 3.3 Admissibilidade da Heurística
- Para garantir que o A\* encontra a solução ótima em problemas de minimização de custos, $h'(n)$ deve ser um **minorante** do custo real $h(n)$ ($h'(n) \le h(n)$, dita heurística **admissível**).
- Se estivermos a maximizar lucros, $h'(n)$ deve ser um **majorante**.
- *Caso particular:* Se $h'(n) = 0$ para todos os nós, o A\* degenera exatamente no algoritmo de Dijkstra / Branch and Bound.

---

### 3.4 Exemplo Prático A\*: Brisbane $\to$ Perth

Tabela de estimativas em linha reta a **Perth** ($h'$):
- Brisbane: $3606$
- Cairns: $3441$
- Melbourne: $2721$
- Sydney: $3291$
- Tennant Creek: $2283$
- Canberra: $3086$
- Adelaide: $2131$
- Darwin: $3848$
- Perth: $0$

#### Traço de Execução:
1. **Raiz:** `Brisbane` $\implies g=0, h'=3606 \implies f'=3606$
   - Sucessores:
     - `Sydney`: $g=917, h'=3291 \implies f'=4208$
     - `Melbourne`: $g=1776, h'=2721 \implies f'=4497$
     - `Tennant Creek`: $g=2487, h'=2283 \implies f'=4770$
     - `Cairns`: $g=1682, h'=3441 \implies f'=5123$
2. **Menor $f'$:** `Sydney (4208)`
   - Expansão de `Sydney`:
     - `Canberra`: $g=917+286=1203, h'=3086 \implies f'=4289$
3. **Menor $f'$:** `Canberra (4289)`
   - Expansão de `Canberra`:
     - `Adelaide`: $g=1203+1159=2362, h'=2131 \implies f'=4493$
     - `Melbourne`: $g=1203+727=1930, h'=2721 \implies f'=4651$
4. **Menor $f'$:** `Melbourne (4497)` *(via Brisbane)*
   - `Adelaide`: $g=1776+727=2503, h'=2131 \implies f'=4634$
   - `Canberra`: $g=1776+727=2503, h'=3086 \implies f'=5589$
5. **Menor $f'$:** `Adelaide (4493)` *(via Brisbane-Sydney-Canberra)*
   - `Perth`: $g=2362+2695=5057, h'=0 \implies f'=5057$ $\to$ **Solução encontrada!**
   - `Tennant Creek`: $g=2362+2042=4404, h'=2283 \implies f'=6687$

> **Verificação de Otimalidade no A\*:**  
> Como $h'$ é admissível (minorante), a pesquisa só termina quando não existirem nós em aberto na lista com $f'(n) < f'(\text{solução}) = 5057$.  
> Após expandir nós restantes com $f' < 5057$ (`Adelaide 4634`, `Melbourne 4651`), todos os nós folha excedem $5057$.  
> Logo, o trajeto **Brisbane $\to$ Sydney $\to$ Canberra $\to$ Adelaide $\to$ Perth** com **5057 km** é garantidamente a solução ótima.

### 3.5 Variantes Notáveis do A\*
- **IDA\*** (*Iterative Deepening A\**): A\* iterativo com níveis de corte de custo.
- **SMA\*** (*Simplified Memory-bounded A\**): A\* com limite estrito de memória.
- **RTA\*** (*Real-Time Adaptive A\**): A\* adaptativo para ambientes de tempo real.
- **AlphA\***: Extensão com cortes $\alpha$-$\beta$.
- **Bidirectional A\***: Pesquisa A\* simultânea a partir da origem e do destino.
- **LPA\*** (*Lifelong Planning A\*) e **D\* Lite**: Algoritmos dinâmicos para ambientes com variações de grafo durante a execução (robótica móvel).

---

## 4. IDA\* (A\* com Aprofundamento Iterativo e Níveis de Corte)

- **Motivação:** No A\*, a tabela de nós em memória pode crescer exponencialmente. Além disso, se $h'$ subestimar gravemente o custo de um nó, o algoritmo pode explorar caminhos desnecessários.
- **Mecanismo:** Define-se um **patamar de corte (*threshold*)** baseado em $f' = g + h'$.
  - Nós com $f'(n) > \text{patamar}$ são podados na iteração corrente.
  - Se nenhuma solução for encontrada dentro do patamar, este é incrementado para o menor valor de $f'$ podado, reiniciando-se a busca.
- **Vantagem:** Consumo de memória linear $O(b \cdot d)$ com garantia de encontrar a solução ótima se $h'$ for admissível.

---

## 5. Subida da Colina (Hill Climbing)

- **Princípio:** Método local e ganancioso (*greedy*). A partir do ponto atual $x$, avaliam-se os vizinhos na distância $\Delta$ ($x \pm \Delta x$):
  - Se algum vizinho tiver melhor valor que o ponto atual, avança-se para esse vizinho.
  - Se nenhum vizinho for melhor, o algoritmo termina.
- **Limitações Críticas:**
  - **Máximos/Mínimos Locais:** O algoritmo fica retido e não alcança o ótimo global.
  - **Planaltos (*Plateaus*):** Zonas planas onde todos os vizinhos têm o mesmo valor de avaliação, impedindo a orientação do declive.
  - **Cordilheiras (*Ridges*):** Cristas estreitas com declives acentuados dos lados onde passos axiais não conseguem progredir eficientemente.
- **Solução:** **Hill Climbing Multi-Start** (múltiplos arranques a partir de posições iniciais distintas geradas aleatoriamente).

---

## 6. Gradiente Ascendente e Descendente

- **Princípio:** Determina analiticamente a melhor direção de subida (ascendente) ou descida (descendente) calculando as **derivadas parciais** (vetor gradiente $\nabla f$).
- **Analogia:** Soltar um berlinde numa superfície tridimensional: ele desce seguindo a linha de maior declive local.
- **Ligação à Aprendizagem Automática:** O **Gradiente Descendente** é o núcleo do treino de redes neuronais durante o processo de *backpropagation*, minimizando a função de perda (*loss function*).

---

## 7. Pesquisa Local, Pesquisa Local Iterada e Aleatorizada

- **Pesquisa Local Básica:** Começa numa solução inicial arbitrária (ou construtiva) e move-se continuamente para vizinhos melhores até convergir num ótimo local.
- **Pesquisa Local Iterada (ILS):** Aplica perturbações calibradas quando atinge um ótimo local, permitindo saltar para outra bacia de atração.
- **Pesquisa Local Aleatorizada:** Executa $N$ vezes a Pesquisa Local partindo de pontos gerados aleatoriamente ou pseudo-aleatoriamente.

---

## 8. Problemas de Sequenciamento e Estrutura de Vizinhança

### 8.1 Formulação do Problema (Escalonamento de 5 Tarefas)
Dadas 5 tarefas $\{t_1, t_2, t_3, t_4, t_5\}$ numa máquina:
- $p_j$: Tempo de processamento.
- $d_j$: Prazo limite (*deadline*).
- $w_j$: Peso de penalização por unidade de atraso.
- $T_j = \max(0, C_j - d_j)$: Atraso da tarefa $j$ ($C_j$ é o instante de conclusão).
- **Função Objetivo:** $\text{Minimizar } \sum_{j=1}^5 w_j T_j$.

#### Dados das Tarefas:

| Tarefa | Processamento ($p_j$) | Conclusão Máxima ($d_j$) | Peso de Atraso ($w_j$) |
| :---: | :---: | :---: | :---: |
| **$t_1$** | 2 | 5 | 1 |
| **$t_2$** | 4 | 7 | 6 |
| **$t_3$** | 1 | 11 | 2 |
| **$t_4$** | 3 | 9 | 3 |
| **$t_5$** | 3 | 8 | 2 |

---

### 8.2 Definição de Vizinhança
Para a sequência inicial $S_0 = [t_3, t_1, t_2, t_5, t_4]$:
- **Vizinhos de Ordem 1 (Troca de 2 elementos adjacentes):**
  1. Trocar posições 1 e 2: $[t_1, t_3, t_2, t_5, t_4]$
  2. Trocar posições 2 e 3: $[t_3, t_2, t_1, t_5, t_4]$
  3. Trocar posições 3 e 4: $[t_3, t_1, t_5, t_2, t_4]$
  4. Trocar posições 4 e 5: $[t_3, t_1, t_2, t_4, t_5]$
- **Vizinhos de Ordem 2 (Troca a 2 posições de distância):**
  Ex.: $[t_2, t_1, t_3, t_5, t_4]$, $[t_3, t_5, t_2, t_1, t_4]$, $[t_3, t_1, t_4, t_5, t_2]$.

---

### 8.3 Execução da Pesquisa Local

Calculando o custo ponderado dos atrasos $\sum w_j T_j$ para a sequência inicial e seus vizinhos de ordem 1:

1. **$S_0 = [t_3, t_1, t_2, t_5, t_4]$:**
   - Conclusões: $C_3 = 1$, $C_1 = 3$, $C_2 = 7$, $C_5 = 10$, $C_4 = 13$
   - Atrasos: $T_3=0$, $T_1=0$, $T_2=0$, $T_5 = 10 - 8 = 2$, $T_4 = 13 - 9 = 4$
   - Custo: $0 + 0 + 0 + (2 \times 2) + (4 \times 3) = 4 + 12 = \mathbf{16}$
2. **Vizinho 1:** $[t_1, t_3, t_2, t_5, t_4] \implies \text{Custo} = \mathbf{16}$
3. **Vizinho 2:** $[t_3, t_2, t_1, t_5, t_4] \implies \text{Custo} = (7-5)\times 1 + (13-9)\times 3 + (10-8)\times 2 = 2 + 12 + 4 = \mathbf{18}$
4. **Vizinho 3:** $[t_3, t_1, t_5, t_2, t_4] \implies \text{Custo} = (10-7)\times 6 + (13-9)\times 3 = 18 + 12 = \mathbf{30}$
5. **Vizinho 4:** $[t_3, t_1, t_2, t_4, t_5]$:
   - Conclusões: $C_3=1, C_1=3, C_2=7, C_4=10, C_5=13$
   - Atrasos: $T_4 = 10-9=1$, $T_5 = 13-8=5$
   - Custo: $(1 \times 3) + (5 \times 2) = 3 + 10 = \mathbf{13}$ $\implies$ **Melhoria!**

A Pesquisa Local transita para $[t_3, t_1, t_2, t_4, t_5]$ (custo 13).  
Prosseguindo as iterações:
$$[t_3,t_1,t_2,t_5,t_4]_{16} \to [t_3,t_1,t_2,t_4,t_5]_{13} \to [t_1,t_3,t_2,t_4,t_5]_{13} \to [t_1,t_2,t_3,t_4,t_5]_{13} \to [t_1,t_2,t_4,t_3,t_5]_{\mathbf{10}}$$
A solução ótima atingida é $[t_1, t_2, t_4, t_3, t_5]$ com **custo 10**.

---

## 9. Pesquisa Tabu (Tabu Search)

- **Conceito:** Meta-heurística de pesquisa local dotada de **memória de curto prazo** (*Lista Tabu*) para evitar regressar a soluções recentes e escapar de ótimos locais.
- **Aceitação de Degradação:** Ao contrário da Pesquisa Local simples, se todos os vizinhos forem piores que o estado atual, a Pesquisa Tabu **aceita o melhor vizinho disponível**, mesmo que piore a função objetivo.
- **Funcionamento da Lista Tabu:**
  - Funciona com política **FIFO** e dimensão fixa (ex.: $k=3$).
  - Armazena as trocas realizadas recentemente (ex.: proibir trocar $(t_5, t_4)$).
  - Após $k$ iterações, a restrição mais antiga expira da lista, libertando o movimento.

---

## 10. Simulated Annealing (Arrefecimento Simulado)

### 10.1 Origem e Analogia
Inspirado na termodinâmica e metalurgia: um metal fundido arrefecido lentamente atinge uma estrutura cristalina regular de energia mínima. Se arrefecido bruscamente (*quenching*), cristaliza com defeitos num estado de alta energia.

| Termodinâmica | Simulated Annealing (Otimização) |
| :--- | :--- |
| **Estados do Sistema** | Soluções do Problema |
| **Energia** | Custo da Solução |
| **Mudança de Estado** | Transição para Solução Vizinha |
| **Temperatura ($T$)** | Parâmetro de Controlo $T$ |
| **Estado Frio (Equilíbrio)** | Solução Final Obtida |

### 10.2 Critério de Aceitação de Metropolis
Dada a solução atual $x_{\text{now}}$ e uma solução vizinha $x'$ com variação de custo $\Delta C = C(x') - C(x_{\text{now}})$:
- Se $\Delta C \le 0$ (a solução melhora): **Aceite incondicionalmente** ($x_{\text{now}} \leftarrow x'$).
- Se $\Delta C > 0$ (a solução piora): **Aceite com probabilidade:**
  $$P(\text{aceitação}) = e^{-\frac{\Delta C}{T}}$$
  Gera-se $q \sim U(0, 1)$; se $q < e^{-\frac{\Delta C}{T}}$, a solução pior é aceite.
  - A temperaturas altas ($T$ elevado): Quase todos os movimentos são aceites (exploração ampla).
  - A temperaturas baixas ($T \to 0$): O algoritmo torna-se estritamente ganancioso (*greedy*).

### 10.3 Esquema de Arrefecimento
- **Redução da Temperatura:** $T_{k+1} = k \cdot T_k$, onde a constante de arrefecimento $k \in [0.8, 0.99]$ (frequentemente próximo de $0.99$).
- **Pseudocódigo Estruturado:**

```text
Construir solução inicial x0; x_now = x0
Definir temperatura inicial T = TI
Repetir para cada nível de temperatura:
    Repetir TL iterações:
        Gerar vizinho aleatório x' na vizinhança N(x_now)
        DeltaC = C(x') - C(x_now)
        Se DeltaC <= 0 então:
            x_now = x'
        Senão:
            Gerar q = random(0, 1)
            Se q < exp(-DeltaC / T) então:
                x_now = x'
    T = k * T
Até critério de paragem satisfeito
Retornar melhor solução encontrada
```

---

## 11. Métodos Baseados na Vida e em Populações

- **Computação Bio-Inspirada:** Imitação de processos evolutivos e comportamentos coletivos biológicos na resolução de problemas.
- **Computação Baseada em Populações:** Em vez de manter um único estado, gere uma população de indivíduos que colaboram e competem, proporcionando uma cobertura distribuída do espaço de pesquisa.
- **Divisão das Abordagens:**
  1. **Inteligência de Enxame (*Swarm Intelligence*):** *Particle Swarm Optimization* (PSO), *Ant Colony Optimization* (ACO).
  2. **Computação Evolucionária:** Algoritmos Genéticos (GA), Algoritmos Meméticos (MA), Programação Genética.

---

## 12. Particle Swarm Optimization (PSO)

Inspirado no comportamento coordenado de bandos de pássaros (*flocking*) e cardumes de peixes: sem líder central, o grupo organiza-se por regras locais simples.

### 12.1 As 4 Regras Fundamentais de Reynolds
1. **Separação:** Evitar colisões com vizinhos próximos.
2. **Alinhamento:** Seguir a direção média dos vizinhos.
3. **Coesão:** Manter a proximidade ao centro de massa do grupo.
4. **Desvio:** Desviar de obstáculos à frente.

### 12.2 Modelo Matemático de Atualização
Para a partícula $i$ na iteração $t$:

$$\vec{v}_i(t+1) = \underbrace{w \cdot \vec{v}_i(t)}_{\text{Inércia}} + \underbrace{c_1 \cdot r_1 \cdot (\vec{x}_{i,\text{pbest}} - \vec{x}_i(t))}_{\text{Componente Cognitiva (Auto-experiência)}} + \underbrace{c_2 \cdot r_2 \cdot (\vec{x}_{g,\text{gbest}} - \vec{x}_i(t))}_{\text{Componente Social (Enxame)}}$$

$$\vec{x}_i(t+1) = \vec{x}_i(t) + \vec{v}_i(t+1)$$

- $\vec{x}_i(t)$: Posição atual da partícula.
- $\vec{v}_i(t)$: Velocidade (vetor de deslocamento).
- $w$: Peso de inércia (controla o equilíbrio entre exploração global e refinação local; pode decair linearmente ao longo do tempo).
- $c_1, c_2$: Coeficientes de aceleração (tipicamente $c_1 + c_2 \approx 4$).
- $r_1, r_2 \sim U(0, 1)$: Variáveis aleatórias uniformes.
- $\vec{x}_{i,\text{pbest}}$: Melhor posição individual histórica da partícula $i$.
- $\vec{x}_{g,\text{gbest}}$: Melhor posição encontrada na vizinhança da partícula (global ou social).

### 12.3 Topologias de Vizinhança
- **Global (*Star Topology*):** Todas as partículas partilham a mesma melhor posição global $G_{\text{best}}$. Convergência rápida, mas propensa a mínimos locais.
- **Geográfica / Social (*Ring / Grid Topology*):** Cada partícula só partilha informação com vizinhos próximos. Promove maior diversidade genética no enxame.

---

## 13. Colónia de Formigas (Ant Colony Optimization - ACO)

### 13.1 Princípio Biológico e Estigmergia
- **Estigmergia:** Comunicação indireta mediada pela alteração física do meio através da deposição de **feromonas**.
- Ao regressarem da comida ao formigueiro, as formigas depositam feromona no solo.
- Formigas em caminhos mais curtos completam mais viagens por unidade de tempo, reforçando a feromona mais rapidamente.
- **Evaporação:** A feromona dissipa-se com o tempo, permitindo ao sistema esquecer gradualmente rotas subótimas e evitar saturação precoce.

### 13.2 ACO Aplicado a Grafos
- Arcos do grafo têm associados níveis de feromona $\tau_{ij}$ e informação heurística $\eta_{ij} = \frac{1}{d_{ij}}$.
- A probabilidade de escolher o próximo nó depende do rácio de feromona e da distância.
- As formigas mantêm uma **lista de memória (tabu)** para evitar ciclos no trajeto.

---

## 14. Algoritmos Genéticos (GA)

Desenvolvidos por **John Holland** na década de 1970, simulam a seleção natural e a genética mendeliana.

### 14.1 Vocabulário: Biologia vs. Algoritmos Genéticos

| Conceito Biológico | Equivalente nos Algoritmos Genéticos |
| :--- | :--- |
| **Indivíduo / Organismo** | Solução candidata ao problema |
| **Gene** | Parâmetro ou variável individual de uma solução |
| **Genótipo / Cromossoma** | Vetor ou sequência codificada de uma solução |
| **População** | Conjunto de soluções mantidas numa dada iteração |
| **Geração** | Ciclo/Iteração de evolução |
| **Reprodução / Cruzamento** | Operador de recombinação (*Crossover*) gerando novos indivíduos |
| **Mutação** | Perturbação estocástica de genes em descendentes |
| **Aptidão (*Fitness*)** | Qualidade da solução medida pela função objetivo |

---

### 14.2 Operadores de Cruzamento (*Crossover*)

#### 1. Cruzamento num Ponto de Corte (1-Point Crossover)
Seleciona-se uma posição aleatória $k$. O primeiro filho herda os genes de $P_1$ até $k$ e de $P_2$ após $k$. O segundo herda o inverso:
- $P_1 = [10110 \mid 010]$
- $P_2 = [11010 \mid 110]$
- Corte após a 5ª posição:
  - $D_1 = [10110 \mid 110] = 10110110_2$
  - $D_2 = [11010 \mid 010] = 11010010_2$

#### 2. Cruzamento em Dois Pontos de Corte (2-Point Crossover)
Selecionam-se duas posições de corte dividindo os cromossomas em 3 segmentos:
- Cortes entre posições 2-3 e 5-6:
  - $P_1 = [10 \mid 110 \mid 010]$
  - $P_2 = [11 \mid 010 \mid 110]$
  - $D_1 = [10 \mid 010 \mid 010] = 10010010_2$
  - $D_2 = [11 \mid 110 \mid 110] = 11110110_2$

#### 3. Cruzamento Uniforme (com Máscara Binária)
Gera-se uma máscara $M$ aleatória de bits do tamanho do cromossoma: se $M_j = 1$ troca os genes dos progenitores; se $M_j = 0$ mantém:
- $P_1 = 10110010$, $P_2 = 11010110$, $M = 01110110$
  - $D_1 = 11010110_2$
  - $D_2 = 10110010_2$

#### 4. Order Crossover (OX) para Problemas de Permutação
Em sequenciamento/rotas (TSP), o cruzamento convencional geraria elementos duplicados e omissões inválidas. O **OX** preserva a ordem relativa e a validade da permutação:
1. Mantém-se o segmento central do progenitor no filho nas posições originais.
2. A partir do segundo ponto de corte, percorre-se ciclicamente o outro progenitor, copiando os elementos que ainda não existam no segmento central para as posições vagas.

---

### 14.3 Operadores de Mutação
- **Mutação Binária:** Inverte um bit escolhido aleatoriamente ($0 \to 1$ ou $1 \to 0$).
- **Mutação por Troca (*Swap Mutation*):** Para permutações, troca a posição de dois genes no cromossoma, equivalente a gerar um vizinho na pesquisa local.

---

### 14.4 Exemplo 1: Maximização de $f(x) = x^2$ em $[0, 63]$ (Codificação de 6 Bits)

População inicial ($N=4$):

| Elemento | Geração 1 | $x$ (decimal) | $f(x) = x^2$ |
| :---: | :---: | :---: | :---: |
| **1** | `100101` | 37 | 1369 |
| **2** | `001001` | 9 | 81 |
| **3** | `010011` | 19 | 361 |
| **4** | `001100` | 12 | 144 |

Cruzamento Uniforme com máscara $M = 010101$:
- Cruzamento $(1, 2) \implies D_5 = \text{`100001`} (x=33, f=1089)$, $D_6 = \text{`001101`} (x=13, f=169)$
- Cruzamento $(3, 4) \implies D_7 = \text{`000110`} (x=6, f=36)$, $D_8 = \text{`011001`} (x=25, f=625)$

Seleção para a Geração 2 (elementos mais aptos: 1, 3, 5, 8):
- `100101` ($f=1369$), `010011` ($f=361$), `100001` ($f=1089$), `011001` ($f=625$)

Segundo ciclo de cruzamento com a mesma máscara gera indivíduos com aptidão até $f=2401$ ($x=49$, `110001`).

> **A Importância Vital da Mutação:**  
> A solução ótima teórica é $x = 63$ (`111111`, $f=3969$). Note-se que na Geração 3 **todos os indivíduos possuem o bit `0` na 5ª posição**.  
> Sem mutação, nenhum operador de cruzamento consegue introduzir um `1` nessa posição, impedindo a população de alguma vez alcançar o ótimo global. A mutação é a garantia de diversidade alélica contra a convergência prematura.

---

### 14.5 Parametrização e Regras Práticas

1. **Dimensão da População:**
   - Muito pequena: fraca diversidade, estagnação rápida em máximos locais.
   - Muito grande: excelente cobertura, mas com custo computacional desproporcionado.
2. **Taxa de Cruzamento ($P_c$):**
   - Tipicamente entre **$60\%$ e $80\%$**.
3. **Taxa de Mutação ($P_m$):**
   - Deve ser baixa, tipicamente em torno de **$1\%$**. Taxas altas transformam o algoritmo numa busca puramente aleatória.
4. **Taxa de Substituição:**
   - Define a percentagem da população substituída entre gerações. Deve ser articulada com a taxa de cruzamento e evitar elitismo cego.
5. **Critérios de Paragem Comuns:**
   - Número máximo fixo de gerações.
   - Atingimento de um limiar satisfatório de aptidão (*fitness* alvo).
   - Limite de tempo de computação.
   - Estagnação da população (sem melhorias durante $K$ gerações consecutivas).

---

### 14.6 Métodos de Seleção

```
                            Operadores de Seleção
                                     │
           ┌─────────────────────────┴────────────────────────┐
           ▼                                                  ▼
     Proporcionais                                   Por Classificação
  (Baseados no valor de f)                       (Baseados na ordem/ranking)
     • Método da Roleta                              • Seleção por Torneio
     • Seleção Universal Estocástica (SUS)
```

1. **Método da Roleta (*Roulette Wheel Selection*):**
   A probabilidade de seleção de cada indivíduo é estritamente proporcional ao seu valor de aptidão:
   $$P_s(i) = \frac{f_i}{\sum_{j=1}^N f_j}$$
   Cada rotação simula a escolha de um progenitor. Indivíduos com melhor avaliação têm fatias maiores da roleta.
2. **Seleção Universal Estocástica (*Stochastic Universal Sampling* - SUS):**
   Gira a roleta uma única vez com $N$ ponteiros equidistantes ($\Delta = \frac{1}{N}$), eliminando flutuações e viés estatístico da amostragem simples.
3. **Seleção por Torneio (*Tournament Selection*):**
   Selecionam-se aleatoriamente $k$ indivíduos da população (ex.: $k=2$ ou $k=3$). O que tiver maior aptidão vence o torneio e é selecionado como progenitor. Muito eficiente e imune a escalas ou *offsets* aberrantes de valores de fitness.

---

## 15. Algoritmos Meméticos (MA)

- **Conceito:** Hibridização de **Algoritmos Genéticos** (exploração global em larga escala) com procedimentos de **Pesquisa Local** (refinamento e exploração intensiva local em cada indivíduo descendente).
- Também conhecidos por *Algoritmos Evolucionários Baldwinianos*, *Algoritmos Lamarckianos* ou *Pesquisa Local Genética*.
- **Vantagem:** Evitam a dispersão ineficiente em torno de ótimos locais, acelerando dramaticamente a convergência para soluções de excelência em problemas complexos de otimização combinatória.

