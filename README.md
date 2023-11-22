# Pricing_Exotic_Options

Opções asiáticas são um tipo de derivativo financeiro em que o prêmio é definido pela média do preço do ativo subjacente ao longo de um período específico, em vez do seu preço em um único ponto no tempo. Essas opções são comumente utilizadas para reduzir o impacto da volatilidade de curto prazo e são populares em mercados com alta volatilidade. 

Matematicamente, uma Call (opção de compra) com preço de exercício fixo, tem seu payoff expresso a seguir: 

$$ \text{max}\left(\frac{1}{T - t_0}\int_{t_0}^{T}S_t dt - K, 0 \right) $$ 

Seu valor é determinado no vencimento $T$ pela diferença entre média aritmética do preço de um ativo subjacente $S_t$ e preço de exercício $K$.

Utilizando esta fórmula, podemos determinar o valor da opção asiática em qualquer instante de tempo $t$ como: 

$$ C(S, t) = e^{-r(T - t)} \mathbb{E}^{\mathbb{Q}} \left[\text{max} \left(\frac{1}{T - t_0}\int_{t_0}^{T}S_t dt - K, 0 \right) \right] $$

Aqui, $C(S, t)$ representa o valor da opção no instante $t$ com $r$ sendo o retorno do ativo livre de risco, e $\mathbb{E}^{\mathbb{Q}}$ representando a esperança sob uma medida de risco-neutro $\mathbb{Q}$. O que, basicamente, significa o valor presente esperado do payoff da opção. 

Vale notar que o preço da opção asiática é *path-dependent* (sensível ao caminho), significando que seu valor é influenciado por todo o caminho do preço da ação ao longo do período $[t_0, T]$. Como resultado, métodos tradicionais de precificação analítica podem não ser viáveis, e os métodos numéricos tornam-se particularmente vantajosos.


### Medida de Risco-Neutro 

Antes de mais nada, precisamos entender como obtemos essa expressão. Vamos começar adotando as mesmas suposições utilizadas no modelo de Black-Scholes. Neste quadro econômico, existem dois ativos-chave: um ativo livre de risco, representado por $B_t$, que se valoriza a uma taxa de juros constante $r$ ao longo do tempo, de acordo com a fórmula $B_t = B_0 e^{rt}$; e um ativo arriscado, representado por $S_t$. O ativo arriscado apresenta um retorno esperado de $\mathbb{E}\left[\frac{dS_t}{S_t}\right] = \mu dt$ e uma volatilidade de $\mathbb{C}\left[\frac{dS_t}{S_t}\right] = \sigma^2 t$.

A dinâmica do preço do ativo arriscado sob a medida do mundo real, denotada como $\mathbb{P}$, é regida pela seguinte equação diferencial estocástica:

$$ dS_t = \mu S_t dt  + \sigma S_t dW_t^{\mathbb{P}}. $$

Precisamos introduzir o conceito de Movimento Browniano Geométrico sob a medida de risco-neutro, denotada como $\mathbb{Q}$. Para garantir a ausência de oportunidades de arbitragem, aplicamos o teorema fundamental de precificação de ativos, que nos permite identificar uma medida equivalente de martingale. Primeiro, considere o preço descontado do ativo arriscado sob $\mathbb{Q}$, denotado como $e^{r(T - t)}S_t$. A medida $\mathbb{Q}$, equivalente a $\mathbb{P}$, é definida através da derivada de Radon-Nikodym

$$ \frac{d \mathbb{Q}}{d \mathbb{P}} = e^{\frac{1}{2}\lambda^2 t - \lambda W_t} $$

onde $\lambda$ é determinado pela equação

$$ \frac{dS_t}{S_t} = r dt + \sigma(\frac{\mu - r}{\sigma} dt + dW_t^{\mathbb{P}}). $$

De acordo com o teorema de Girsanov, podemos definir um novo processo de movimento Browniano, denotado como $dW_t^{\mathbb{Q}}$, da seguinte forma:

$$ dW_t^{\mathbb{Q}} := \lambda t + dW_t^{\mathbb{P}}, \quad \text{where} \quad \lambda = \frac{\mu - r}{\sigma} $$

Essa transformação resulta na seguinte dinâmica de risco-neutro para o ativo arriscado:

$$ dS_t = r S_t dt  + \sigma S_t dW_t^{\mathbb{Q}}. $$

Em essência, caracterizamos agora a dinâmica do ativo arriscado sob a medida de risco-neutro $\mathbb{Q}$ em termos de um Movimento Browniano Geométrico.


### Esquema Numérico

Uma abordagem para solucionar a problemática mencionada é por meio de métodos numéricos. Nesse contexto, aplicarei método de Euler-Maruyama para simular a trajetória de um ativo e a técnica de simulação de Monte Carlo para aproximar o valor esperado do payoff. A seguir apresento um esboço do esquema: 

1. Simular o Passeio Aleatório sob a Medida Neutra ao Risco:
 - Comece gerando passeios aleatórios para o preço do ativo subjacente (no nosso caso, $S_t$) sob a medida neutra ao risco ($\mathbb{Q}$). Utilize o modelo de Movimento Browniano Geométrico para simular esses caminhos. 

2. Calcular o Pagamento da Opção para Cada Trajetória:
 - Para cada passeio simulado, calcule o pagamento da opção asiática no momento de exercício $T$. Isso envolve calcular a média dos preços do ativo durante o período de observação $[t_0, T]$ e, em seguida, compará-la ao preço de exercício $K$.

3. Realizar um Grande Número de Simulações para o Mesmo Intervalo de Tempo:
 - A chave para a precisão do método de Monte Carlo é gerar um grande número de simulações para o preço do ativo. Quanto mais simulações forem realizadas, mais próximo você estará de uma estimativa precisa do valor da opção.

4. Calcular a Média dos Pagamentos em Todas as Realizações:
 - Após realizar um grande número de simulações, calcule a média dos pagamentos obtidos em todas as realizações. Essa média representará uma estimativa do valor esperado do pagamento da opção.

5. Desconto para o Valor Presente:
 - Por fim, desconte o valor médio dos pagamentos calculado no passo anterior para trazê-lo ao valor presente. Isso é feito multiplicando o valor médio pelo fator de desconto $e^{-r(T - t)}$, onde $r$ é a taxa de juros livre de risco e $(T - t)$ é o tempo até a expiração do contrato.


### Método de Euler-Maruyama

Para simular os preços de ativos, empregaremos o método de Euler-Maruyama. É essencial destacar que precisamos simular todo caminho, uma vez que o contrato é path-dependent. Consideremos uma Equação Diferencial Estocástica:

$$ dX_t = \mu(t, X(t))dt + \sigma(t, X(t))dW(t), \quad t \in [0, T] \quad X(0) = X_0. $$

Portanto, o Movimento Browniano Geométrico pode ser representado em forma integral da seguinte maneira:

$$ X(t_{j+1}) = X(t_j) + \int_{t_j}^{t_{j+1}} \mu(s, X(s)) dS + \int_{t_j}^{t_{j+1}} \sigma(s, X(s)) dW(s) $$

É importante reconhecer que essa aproximação fundamental é uma extensão do método de Euler aplicado a equações diferenciais ordinárias. A discretização da Equação Diferencial Estocástica usando esse método pode ser expressa da seguinte forma:

$$ X_{j+1} = X(t_j) + \mu(t_j, X_j) \Delta t_j + \sigma(t_j, X_j) \Delta W(t_j) $$

Embora esse método de aproximação possa ser intuitivamente compreendido considerando a primeira integral aproximada por $\mu(t_j, X_j) \Delta t$ e a segunda por $\sigma(t_j, X_j) \Delta W(t_j)$, sua derivação formal é baseada em uma expansão de Taylor.

Agora, procederemos à simulação de uma única trajetória para a aproximação numérica, que é inerentemente uma variável aleatória. Diferentes resultados são obtidos para cada passeio $t \rightarrow W(t)$ do processo de Wiener. Consequentemente, podemos determinar o preço de uma opção asiática usando a fórmula fornecida:

$$ C = e^{-rT} \mathbb{E}^{\mathbb{Q}}\left[\max\left(\frac{1}{T}\int_{0}^{T}S_t dt - K, 0\right)\right] $$

Aqui, a dinâmica de $S_{t+1}$ é dada por:

$$  S_{t+1} = S_t + r S_t \Delta t + \sigma S_t \sqrt{\Delta t} Z, $$

onde $Z$ representa uma variável aleatória Normal padrão para modelar o movimento browniano, e $\Delta t$ é o passo usado para dividir o intervalo $[0, T]$ em $n$ partes iguais. Podemos definir $\Delta t = \frac{T}{n}$ para que $t_j = j \Delta t$ para $j = 0, 1, \ldots, n$.

O esquema para implementar o método de Euler-Maruyama segue como:

**Para $$ j $$ de 1 a $$ n $$:**
   - Gerar $Z$ a partir de uma distribuição Normal padrão, $Z \sim \mathcal{N}(0, 1)$.
   - Atualizar $S$ usando a fórmula: $S_{t+1} \gets S_t + r S_t \Delta t + \sigma S_t \sqrt{\Delta t}Z$.

Esse processo iterativo nos permite aproximar os valores de $S$ ao longo de um intervalo de tempo discreto.


### Simulação de Monte Carlo

Monte Carlo (MC) é um algoritmo prontamente implementável utilizado para aproximar expectativas. Sua eficácia baseia-se no princípio conhecido como a Lei Forte dos Grandes Números (SLLN). Vamos explorar a expectativa de uma variável aleatória $X$, denotada como $\mathbb{E}[X]$, expressa como uma integral:

$$ \mathbb{E}[X] = \int x p_X(x) dx $$

Agora, se pudéssemos obter numerosas amostras independentes e identicamente distribuídas (i.i.d.), $x_1, x_2, \ldots, x_N$, da distribuição de probabilidade $p_X$, onde cada $x_i$ adere à distribuição $p_X$. À medida que o número de amostras, denotado como $m$, tende para o infinito, de acordo com a Lei dos Grandes Números:

$$ \frac{1}{m}\sum_{i=1}^{m}x_i \rightarrow \mathbb{E}[X] = \int x p_X(x)dx $$

Essencialmente, observamos:

$$ \frac{1}{m}\sum_{i=1}^{m}x_i \rightarrow \int x p_X(x) \quad \text{(SLLN)} $$

Agora, considere uma integral envolvendo uma função $f(x)$ em relação a $x$:

$$\int f(x) dx = \int \frac{f(x)}{p_X(x)} p_X(x) dx $$

Isso pressupõe que $p_X(x)$ permanece não nulo ao longo do intervalo de integração.

Se possuirmos amostras i.i.d. $x_1, \ldots, x_N$, então, relacionado à SLLN, à medida que o tamanho da amostra $m$ cresce:

$$ \frac{1}{m}\sum_{i=1}^{m} f(x_i) p_X(x_i) \rightarrow \int \frac{f(x)}{p_X(x)} p_X(x) dx = \int f(x) dx. $$

Essencialmente, estamos estimando a integral $\int f(x)dx$ por meio do uso de $\frac{1}{m} \sum_{i=1}^{m}f(x_i)p_X(x_i)$.

Vale ressaltar que a escolha de $p_X(x)$ é inteiramente discricionária, e a precisão da estimativa é notavelmente influenciada pela escolha da distribuição de amostragem. Dentro desse framework, conceitos pertinentes incluem técnicas como amostragem e técnicas de redução de variância.

Para estimar o valor de uma opção asiática usando simulações de Monte Carlo, as seguintes etapas seriam realizadas

\begin{align*}
&\text{Definir} \Delta t \leftarrow \frac{T}{n} \\
&\textbf{Para cada} i \textbf{de} 1 \textbf{a} m: \\
&\quad\text{Definir} S \leftarrow S_0 \\
&\quad\textbf{Para cada} j \textbf{e} 1 \textbf{a} n: \\
&\quad\quad\text{Gerar} Z \text{de uma distribuição Normal padrão,} Z \sim \mathcal{N}(0, 1) \\
&\quad\quad\text{Atualizar} S \text{usando a fórmula:} S_{t+1} \gets S_t + r S_t \Delta t + \sigma S_t \sqrt{\Delta t}Z \\
&\quad\text{Calcular} \overline{S}_i = \frac{1}{n} \sum_{j=1}^{n} S_{t_j} \\
&\quad\text{Definir} C_i \leftarrow \max(\overline{S}_i - K, 0) \\
&\text{Calcular} C = \frac{1}{m} \sum_{i=1}^{m} V_i
\end{align*}

