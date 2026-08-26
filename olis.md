# Clase Magistral — Simulación de Variables Aleatorias
### Guía completa para resolver la Práctica 2 (CC2017 — Modelación y Simulación)

Este documento está pensado para que lo leas de principio a fin **sin conocimientos previos** de simulación estocástica más allá de probabilidad básica (variables aleatorias, densidad, función de distribución, esperanza). Al terminar deberías poder resolver los 10 ejercicios de la práctica entendiendo *por qué* funciona cada algoritmo, no solo copiándolo.

---

## 0. La idea central de todo el curso

Una computadora, en el fondo, solo sabe generar (con un algoritmo determinístico llamado *generador pseudoaleatorio*) una secuencia de números que **se comportan como** si vinieran de una distribución **Uniforme(0,1)**. Llamamos a esta capacidad la primitiva básica:

$$U \sim \text{Uniforme}(0,1)$$

**Todo lo demás en este curso es: "¿cómo transformo uno o varios $U$'s para que el resultado tenga la distribución que yo quiero?"**

- Si quiero una exponencial → aplico una fórmula a $U$.
- Si quiero una normal → aplico un algoritmo (rechazo o método polar) a varios $U$'s.
- Si quiero puntos de un proceso de Poisson → uso varias exponenciales (que a su vez vienen de $U$'s).

Cada ejercicio de la práctica es una instancia de este problema general: **dado un $U(0,1)$ (o varios), producir una variable con la distribución pedida, de la forma más eficiente posible.**

"Eficiente" en este contexto casi siempre significa: *generar el menor número de $U$'s posible por cada muestra útil que obtienes*, evitando desperdiciar cálculos en muestras que vas a descartar.

---

## 1. Repaso rápido de lo que necesitas de probabilidad

- **Función de distribución acumulada (CDF)**: $F(x) = P(X \le x)$. Va de 0 a 1, es no decreciente.
- **Función de densidad (PDF)**: $f(x) = F'(x)$ para variables continuas. $f$ es la "pendiente" de $F$.
- **Esperanza**: $E[X] = \int x f(x)\, dx$.
- **Esperanza condicional**: $E[X \mid A] = \int x \, f(x \mid A) \, dx$, donde $f(x\mid A)$ es la densidad de $X$ *sabiendo que* ocurrió el evento $A$ (por ejemplo $A = \{X<0.05\}$).
- **Ley de los Grandes Números (LGN)**: si generas $X_1, X_2, \dots, X_n$ independientes con la misma distribución, entonces
$$\bar X_n = \frac{1}{n}\sum_{i=1}^n X_i \xrightarrow[n\to\infty]{} E[X]$$
  Esta es la ley que hace que "simular y promediar" sea una forma válida de **estimar** una esperanza o una probabilidad que no puedes (o no quieres) calcular a mano. Es la base de toda la simulación de Monte Carlo.

Con esto ya tienes todo el álgebra que necesitas. Vamos a las técnicas.

---

## 2. Método de la Transformada Inversa

### 2.1 La idea, en una frase

Si conoces la fórmula de $F(x)$ y puedes despejar $x$, entonces generar $X$ es tan fácil como generar un $U(0,1)$ y meterlo en la fórmula despejada.

### 2.2 Por qué funciona (la demostración, corta)

Queremos generar $X$ con CDF $F$. Afirmación: si $U\sim \text{Unif}(0,1)$, entonces $X = F^{-1}(U)$ tiene exactamente la distribución $F$.

**Prueba:**
$$P(X \le x) = P(F^{-1}(U) \le x) = P(U \le F(x)) = F(x)$$
(la segunda igualdad usa que $F$ es creciente, así que $F^{-1}(U)\le x \iff U \le F(x)$; la tercera usa que $P(U\le u)=u$ porque $U$ es uniforme). Y $P(X\le x) = F(x)$ es **exactamente** la definición de que $X$ tenga distribución $F$. $\blacksquare$

### 2.3 El algoritmo (siempre es este, en 2 pasos)

1. Genera $U \sim \text{Unif}(0,1)$.
2. Calcula $X = F^{-1}(U)$, es decir, resuelve $F(x) = u$ para $x$, usando el valor generado de $U$ como $u$.

**Ejemplo mínimo (exponencial estándar, tasa 1):** $F(x) = 1-e^{-x}$. Despejando: $u = 1-e^{-x} \Rightarrow x = -\ln(1-u)$. Como $1-U$ tiene la misma distribución que $U$, normalmente se escribe $X=-\ln U$.

### 2.4 Caso discreto (lo vas a necesitar para el método de composición, sección 4)

Si $X$ es discreta con $P(X=x_i)=p_i$, "invertir" la CDF significa: parte el intervalo $(0,1)$ en pedazos consecutivos de tamaños $p_1, p_2, p_3,\dots$. Genera $U$, y $X=x_i$ si $U$ cae en el $i$-ésimo pedazo, es decir si
$$p_1+\cdots+p_{i-1} \;\le\; U \;<\; p_1+\cdots+p_i$$

Visualmente:
```
0 ─────p1─────|────p2────|──p3──|... 1
   x=x1          x=x2       x=x3
```

### 2.5 Aplicación directa: Ejercicio 1

$X\sim\text{Exp}(1)$, y quieres la distribución **condicional** $X \mid X<0.05$, con densidad
$$f(x) = \frac{e^{-x}}{1-e^{-0.05}}, \qquad 0<x<0.05$$

**Paso A — encuentra $F(x)$ integrando $f$:**
$$F(x) = \int_0^x f(t)\,dt = \frac{1-e^{-x}}{1-e^{-0.05}}, \qquad 0<x<0.05$$
(Verifica: en $x=0$, $F=0$; en $x=0.05$, $F=1$. Correcto, es una CDF válida en ese intervalo truncado.)

**Paso B — despeja $x$ de $F(x)=u$:**
$$u(1-e^{-0.05}) = 1-e^{-x} \;\Longrightarrow\; e^{-x} = 1-u(1-e^{-0.05}) \;\Longrightarrow\; X = -\ln\!\big(1-U(1-e^{-0.05})\big)$$

**Paso C — el algoritmo final:**
1. Genera $U\sim\text{Unif}(0,1)$.
2. $X \gets -\ln\!\big(1-U(1-e^{-0.05})\big)$.

Repite 1000 veces y calcula $\bar X = \frac{1}{1000}\sum X_i$: esa es tu **estimación** de $E[X\mid X<0.05]$ (aquí es donde usas la LGN de la sección 1).

**¿Por qué es "eficiente"?** Porque cada $U$ genera una $X$ **válida y utilizable**, sin ningún descarte. La alternativa ingenua sería: genera $X\sim\text{Exp}(1)$ normal (sin truncar) y quédate solo con las que caen por debajo de 0.05. Como $P(X<0.05)=1-e^{-0.05}\approx 0.0488$, ¡tirarías a la basura **más del 95%** de tus muestras! Ese contraste es exactamente lo que el ejercicio quiere que notes y expliques.

**Paso D — el valor exacto (no es simulación, es cálculo):**
$$E[X\mid X<0.05] = \int_0^{0.05} x\, f(x)\, dx = \frac{\int_0^{0.05} x e^{-x}\,dx}{1-e^{-0.05}}$$

Usando integración por partes, $\int_0^{a} x e^{-x}\,dx = 1-(1+a)e^{-a}$. Con $a=0.05$:

$$E[X\mid X<0.05] = \frac{1-(1.05)e^{-0.05}}{1-e^{-0.05}} \approx \frac{1-0.998791}{0.048771} \approx \frac{0.001209}{0.048771} \approx 0.02479$$

Nota algo bonito: como el intervalo $(0,0.05)$ es tan chico, la exponencial ahí adentro es casi plana (casi como una uniforme), y por eso el resultado ($\approx 0.02479$) está muy cerca de $0.05/2 = 0.025$, que sería la media de una $\text{Unif}(0,0.05)$. Tu promedio simulado con 1000 muestras debería acercarse mucho a $0.02479$.

---

## 3. Estimación por Monte Carlo (aplicación directa de la LGN)

Esta no es una técnica nueva de *generación*: es cómo **usas** las técnicas de generación para responder preguntas del tipo *"¿cuál es la probabilidad de que ocurra tal cosa?"* o *"¿cuál es el valor esperado de tal cantidad?"*, cuando la respuesta exacta es difícil o imposible de calcular a mano.

### 3.1 Receta general

1. Diseña un experimento aleatorio que puedas simular de principio a fin y que produzca, en cada réplica, el número que te interesa (una suma, un indicador de "sí ocurrió", etc.).
2. Repite el experimento $N$ veces, de forma independiente.
3. Promedia los resultados. Por LGN, ese promedio converge al valor real que buscas.

Si lo que buscas es una **probabilidad** $P(A)$, el truco es definir el indicador $I = \mathbb{1}\{A \text{ ocurrió}\}$ en cada réplica (vale 1 si $A$ ocurrió, 0 si no). Como $E[I] = P(A)$, promediar los $I$'s estima directamente la probabilidad.

### 3.2 Aplicación: Ejercicio 4

Tienes 1000 asegurados. Cada uno, independientemente:
- reclama con probabilidad $0.05$ (Bernoulli),
- si reclama, el monto es $\text{Exp}(\text{media } 800)$, es decir tasa $\lambda = 1/800$.

Quieres $P(\text{suma de reclamos} > 50{,}000)$.

**Una réplica del experimento:**
```
S = 0
para i = 1 hasta 1000:
    U ~ Unif(0,1)
    si U < 0.05:                     # el asegurado i reclama
        V ~ Unif(0,1)
        monto = -800 * ln(V)         # transformada inversa de Exp(1/800)
        S = S + monto
I = 1 si S > 50000, si no I = 0
```

**Repite esto $N$ veces** (por ejemplo $N=10{,}000$) y estima:
$$\hat p = \frac{1}{N}\sum_{j=1}^N I_j$$

Este es el mismo patrón del Ejercicio 1 (parte de estimación): generas, repites, promedias. La diferencia es que ahí promediabas los valores de $X$ mismos (para estimar $E[X\mid X<0.05]$), y aquí promedias indicadores 0/1 (para estimar una probabilidad). Es la misma idea de fondo.

**Bono (opcional pero recomendable):** puedes reportar el error estándar de tu estimación, $\sqrt{\hat p (1-\hat p)/N}$, para dar una noción de cuán confiable es tu número.

---

## 4. Método de Composición

### 4.1 El problema que resuelve

A veces la $F(x)$ que te dan **no** es fácil de invertir directamente, pero se puede escribir como una **mezcla ponderada** de otras CDFs $F_1,\dots,F_n$ que sí son fáciles:

$$F(x) = \sum_{i=1}^n p_i F_i(x), \qquad p_i \ge 0, \quad \sum_{i=1}^n p_i = 1$$

Intuición: en vez de generar directamente de la mezcla complicada $F$, primero **eliges al azar cuál de las $F_i$ vas a usar** (con probabilidad $p_i$ para la $i$-ésima), y luego generas de esa $F_i$ que sí sabes generar fácil.

### 4.2 El algoritmo

1. Genera un índice aleatorio $I \in \{1,\dots,n\}$ con $P(I=i)=p_i$ (usando la transformada inversa discreta de la sección 2.4).
2. Dado que salió $I=i$, genera $X$ con distribución $F_i$ (usando el método que corresponda a $F_i$, típicamente transformada inversa).
3. Devuelve $X$.

### 4.3 Por qué funciona (la demostración)

Condicionando en el valor de $I$ (regla de la probabilidad total):
$$P(X\le x) = \sum_{i=1}^n P(X\le x \mid I=i)\, P(I=i) = \sum_{i=1}^n F_i(x)\, p_i = F(x) \qquad \blacksquare$$

Esta es literalmente la respuesta al **Ejercicio 2**: describe el algoritmo de los 3 pasos de arriba y da esta demostración de una línea como justificación de por qué genera la distribución $F$ correcta.

### 4.4 La habilidad clave para el Ejercicio 3: reconocer la mezcla

Lo difícil del Ejercicio 3 no es aplicar el algoritmo (eso ya lo sabes), sino **mirar una $F(x)$ dada y descubrir cómo se descompone** en $\sum p_i F_i(x)$. Un truco muy útil: si $F_i(x) = x^{k}$ en $(0,1)$ (para $k>0$), entonces $F_i$ es la CDF de $X_i = U^{1/k}$, porque:
$$F_i(x) = P(U^{1/k}\le x) = P(U \le x^k) = x^k \quad\checkmark$$

Con esa pieza, resolvamos los tres incisos:

**(a)** $F(x) = \dfrac{x+x^3+x^5}{3}$, $0\le x\le 1$.

Reescribe: $F(x) = \frac13 x + \frac13 x^3 + \frac13 x^5$. Ya está en la forma $\sum p_i F_i(x)$ con:
- $p_1=p_2=p_3=\frac13$
- $F_1(x)=x$ (es decir $X_1=U$), $F_2(x)=x^3$ ($X_2=U^{1/3}$), $F_3(x)=x^5$ ($X_3=U^{1/5}$)

**Algoritmo:** genera $I\in\{1,2,3\}$ uniforme (cada uno con prob. $1/3$: por ejemplo, con un $U_0\sim\text{Unif}(0,1)$, sea $I=1$ si $U_0<1/3$, $I=2$ si $1/3\le U_0<2/3$, $I=3$ si no). Luego genera $U\sim\text{Unif}(0,1)$ nuevo y devuelve $U^{1/k}$ según el $I$ elegido ($k=1,3,5$ respectivamente).

**(b)**
$$F(x) = \begin{cases} \dfrac{1-e^{-2x}+2x}{3}, & 0<x<1 \\[4pt] \dfrac{3-e^{-2x}}{3}, & 1<x<\infty \end{cases}$$

Este es el más instructivo. Vamos a descomponerlo con cuidado, porque el proceso es el mismo que usarás siempre que te den una $F$ "rara" por tramos.

*Paso 1 — separa visualmente los términos que sí reconoces.* En ambos tramos aparece un término con $e^{-2x}$ (huele a exponencial de tasa 2) y otro término "lineal/acotado" (huele a uniforme). Intenta escribir:
$$F(x) = \frac13\underbrace{(1-e^{-2x})}_{\text{CDF de Exp(2)}} + \frac13 G(x)$$
donde $G(x)$ es lo que queda. Para $0<x<1$: $F(x)-\frac13(1-e^{-2x}) = \frac{2x}{3}$, así que $G(x)=2x$. Para $x>1$: $F(x)-\frac13(1-e^{-2x}) = \frac{3-e^{-2x}}{3}-\frac{1-e^{-2x}}{3} = \frac{2}{3}$, así que $G(x)=2$.

*Paso 2 — identifica $G(x)/2$.* Tenemos $G(x)=2x$ para $0<x<1$ y $G(x)=2$ (constante) para $x>1$. Entonces $H(x):=G(x)/2$ vale $x$ en $(0,1)$ y $1$ para $x\ge 1$: **¡esa es exactamente la CDF de una Uniforme(0,1)!**

*Paso 3 — junta todo:*
$$F(x) = \underbrace{\frac13}_{p_1}\underbrace{(1-e^{-2x})}_{F_1:\ \text{Exp}(2)} \;+\; \underbrace{\frac23}_{p_2}\underbrace{H(x)}_{F_2:\ \text{Unif}(0,1)}$$

Verifica que $p_1+p_2=\frac13+\frac23=1$. ✓

**Algoritmo final:** genera $U_0\sim\text{Unif}(0,1)$. Si $U_0<1/3$, genera $X\sim\text{Exp}(2)$ (con transformada inversa: $X=-\frac12\ln U$). Si $U_0\ge 1/3$, genera $X\sim\text{Unif}(0,1)$ directamente (¡$X=U$!).

Esto ilustra la habilidad general: cuando veas una $F$ por tramos con una mezcla de "algo exponencial" y "algo lineal/constante", casi siempre se puede separar así. Vale la pena que **verifiques tú mismo** con álgebra que el $F(x)$ reconstruido efectivamente coincide con el original en ambos tramos (yo ya lo verifiqué arriba, pero rehacerlo a mano es la mejor forma de interiorizar la técnica).

**(c)** $F(x) = \sum_{i=1}^n \alpha_i x^i$, $0\le x\le1$, con $\alpha_i\ge0$, $\sum\alpha_i=1$.

Este es el caso general de (a): ya está en la forma $\sum p_i F_i(x)$ con $p_i=\alpha_i$ y $F_i(x)=x^i$ (es decir, $X_i = U^{1/i}$). El algoritmo es idéntico: elige $I=i$ con probabilidad $\alpha_i$ (transformada inversa discreta), genera $U$ nuevo, y devuelve $U^{1/I}$.

---

## 5. Método de Aceptación-Rechazo

### 5.1 Cuándo lo necesitas

Cuando $F$ **no** se puede invertir en forma cerrada (no puedes despejar $x$ algebraicamente), pero conoces otra densidad $g$ —de la que sí sabes generar fácil— tal que la densidad objetivo $f$ nunca supera a $c\cdot g$ para alguna constante $c\ge 1$:
$$f(x) \le c\, g(x) \quad \text{para todo } x$$

### 5.2 Intuición geométrica (la que más ayuda a entenderlo)

Imagina la curva $c\cdot g(x)$ dibujada sobre la curva $f(x)$ — como un "techo" que la cubre completamente. La idea es:

1. Genera un punto $Y$ bajo la curva $c\cdot g$ (esto es fácil porque $g$ es una densidad simple).
2. Genera una altura aleatoria uniforme bajo el techo en $x=Y$.
3. Si esa altura cae **debajo** de la curva real $f(Y)$, acepta el punto — cayó "dentro" de la región de $f$. Si cae **arriba** de $f(Y)$ (pero seguía bajo el techo $c g$), recházalo.

Los puntos aceptados, vistos solo en su coordenada $x$, tienen exactamente la densidad $f$. Es literalmente "dibujar puntos al azar bajo una curva fácil, y quedarte solo con los que caen bajo la curva difícil".

### 5.3 El algoritmo

1. Genera $Y\sim g$.
2. Genera $U\sim\text{Unif}(0,1)$, independiente de $Y$.
3. Si $U \le \dfrac{f(Y)}{c\, g(Y)}$: **acepta**, $X=Y$. Si no: vuelve al paso 1.

### 5.4 Por qué funciona (demostración corta)

$$P(\text{aceptar} \mid Y=y) = \frac{f(y)}{c\,g(y)}$$
La densidad conjunta de "$Y$ cae cerca de $y$ **y** se acepta" es $g(y)\cdot \frac{f(y)}{cg(y)} = \frac{f(y)}{c}$. Integrando sobre todo $y$: $P(\text{aceptar}) = \frac1c\int f(y)\,dy = \frac1c$. Entonces la densidad condicional de $Y$ dado que fue aceptado es
$$\frac{f(y)/c}{1/c} = f(y) \qquad \blacksquare$$
Exactamente la densidad que queríamos.

**Eficiencia:** cada intento se acepta con probabilidad $1/c$, así que el número de intentos hasta la primera aceptación sigue una distribución Geométrica con media $c$. **Por eso siempre conviene el $c$ más pequeño posible** — se obtiene tomando $c = \max_x \frac{f(x)}{g(x)}$ (el punto donde $f$ y $g$ están "más separadas" relativamente).

### 5.5 Aplicación: Ejercicio 5 (generar normales vía rechazo)

Queremos $Z\sim N(0,1)$. Truco: primero genera $|Z|$ (el valor absoluto), cuya densidad es
$$f(x) = \sqrt{\frac{2}{\pi}}\, e^{-x^2/2}, \qquad 0<x<\infty$$
y luego decide el signo por separado (con probabilidad $1/2$ cada uno, ya que la normal es simétrica).

**Elección de $g$:** usamos $g(x)=e^{-x}$ (Exponencial de tasa 1), porque es fácil de generar y tiene soporte igual $(0,\infty)$.

**Encontrar $c$:**
$$\frac{f(x)}{g(x)} = \sqrt{\frac{2}{\pi}}\, e^{-x^2/2+x}$$
Maximiza esto derivando el exponente $-x^2/2+x$ respecto a $x$: la derivada es $-x+1=0 \Rightarrow x=1$ (máximo). Sustituyendo:
$$c = \sqrt{\frac{2}{\pi}}\, e^{-1/2+1} = \sqrt{\frac{2e}{\pi}} \approx 1.32$$

**El algoritmo (versión directa):**
1. Genera $Y\sim\text{Exp}(1)$.
2. Genera $U\sim\text{Unif}(0,1)$.
3. Acepta si $U \le \dfrac{f(Y)}{c\,g(Y)} = \exp\{-(Y-1)^2/2\}$ (verifica tú mismo el álgebra: es $f(Y)/(cg(Y))$ simplificado). Si se acepta, $X=Y=|Z|$. Si no, vuelve al paso 1.
4. Genera $U'\sim\text{Unif}(0,1)$ nuevo: si $U'\le 1/2$, $Z=X$; si no, $Z=-X$.

**Versión "simultánea" (la que da el material de apoyo, más elegante):** usa el hecho de que $-\ln U$ es también una Exponencial(1) independiente. Entonces en vez de generar $U$ y comparar con $\exp\{-(Y-1)^2/2\}$, generas directamente una **segunda** exponencial $Y_2\sim\text{Exp}(1)$ y la condición de aceptación $U\le\exp\{-(Y-1)^2/2\}$ se vuelve equivalente a:
$$Y_2 - \frac{(Y_1-1)^2}{2} > 0$$
Si se cumple, defines $Y \gets Y_2 - \frac{(Y_1-1)^2}{2}$ (esto, como bono, es una exponencial(1) independiente de $Z$ — útil si luego necesitas pares independientes). Si no se cumple, regresas al paso 1. Esta versión es la que literalmente describe el "Ejemplo 5f" del material de apoyo — es la que deberías implementar si el ejercicio pide "el método del Ejemplo 5f".

**Costo esperado:** como $c\approx1.32$, se necesitan en promedio $1.32$ intentos para aceptar, y cada intento usa **2** exponenciales (una para $Y_1$, otra para probar la aceptación) $\Rightarrow$ en promedio $2\times 1.32\approx 2.64$... el material reporta $1.64$ exponenciales y $1.32$ cuadrados en promedio por normal generada (contando con más cuidado el número de exponenciales por intento vs. por normal aceptada). El número exacto no es lo importante — lo importante es que **entiendas que $c$ mide directamente cuánto trabajo desperdicias**, y que por eso siempre se busca minimizarlo.

---

## 6. Procesos de Poisson (una dimensión: el tiempo)

### 6.1 ¿Qué es un proceso de Poisson?

Es un modelo para "eventos que llegan al azar en el tiempo" (llamadas a un call center, clientes a una tienda, decaimientos radiactivos...), con tres propiedades:

1. Empieza en $N(0)=0$.
2. **Incrementos independientes:** lo que pasa en $(t_1,t_2]$ no depende de lo que pasó antes de $t_1$.
3. **Incrementos estacionarios con tasa $\lambda$:** el número de eventos en cualquier intervalo de longitud $h$ es Poisson($\lambda h$), sin importar dónde empiece el intervalo.

### 6.2 El hecho que lo hace simulable: tiempos entre llegadas

**Propiedad clave (la vas a usar constantemente):** los tiempos entre eventos consecutivos de un proceso de Poisson de tasa $\lambda$ son variables **Exponencial($\lambda$) independientes**. Esto convierte "simular un proceso de Poisson" en "sumar exponenciales una tras otra".

### 6.3 Algoritmo — Ejercicio 6

Para generar los eventos de un proceso de Poisson de tasa $\lambda$ en $[0,T]$:

```
t = 0
lista_de_eventos = []
mientras t < T:
    U ~ Unif(0,1)
    t = t - (1/λ) * ln(U)        # siguiente tiempo entre llegadas ~ Exp(λ)
    si t < T:
        lista_de_eventos.agrega(t)
devolver lista_de_eventos
```

Cada iteración usa la transformada inversa de la sección 2 (sobre la exponencial) para dar el siguiente "salto" de tiempo, y vas acumulando el reloj $t$ hasta pasarte de $T$.

---

## 7. Procesos de Poisson **no homogéneos**: adelgazamiento (thinning)

### 7.1 El problema

Si la tasa **cambia con el tiempo**, $\lambda(t)$, ya no puedes usar exponenciales de tasa fija para los tiempos entre eventos (esa propiedad de la sección 6.2 depende de que la tasa sea constante). Necesitas otra estrategia.

### 7.2 La idea del adelgazamiento

1. Encuentra una cota superior $\lambda^*$ tal que $\lambda(t)\le \lambda^*$ para todo $t$ en el intervalo de interés (típicamente $\lambda^* = \max_t \lambda(t)$).
2. Genera un proceso de Poisson **homogéneo** ordinario de tasa $\lambda^*$ (sección 6.3) — este proceso tiene "más eventos de los que necesitas".
3. **Adelgázalo**: cada evento que ocurrió en el tiempo $t$ lo conservas (aceptas) con probabilidad $\dfrac{\lambda(t)}{\lambda^*}$, y lo descartas con probabilidad complementaria.
4. Los eventos que sobreviven forman exactamente un proceso de Poisson no homogéneo con intensidad $\lambda(t)$.

**Intuición de por qué funciona:** en el instante $t$, el proceso "grande" de tasa $\lambda^*$ tiene una probabilidad $\lambda^* dt$ de tener un evento en $[t,t+dt)$. Al aceptar cada uno con probabilidad $\lambda(t)/\lambda^*$, la probabilidad de que sobreviva un evento en ese instante queda en $\lambda^* dt \cdot \frac{\lambda(t)}{\lambda^*} = \lambda(t)\, dt$ — exactamente la tasa que queríamos en ese punto.

### 7.3 Aplicación: Ejercicio 7

**Intensidad dada:** $\lambda(t) = 3+\dfrac{4}{t+1}$, en $[0,10]$.

**(a) Encuentra $\lambda^*$.** Como $\frac{4}{t+1}$ es decreciente en $t$ (para $t\ge0$), $\lambda(t)$ alcanza su máximo en $t=0$:
$$\lambda^* = \lambda(0) = 3+4 = 7$$

**Algoritmo completo:**
```
t = 0; eventos = []
mientras t < 10:
    U ~ Unif(0,1)
    t = t - (1/7) * ln(U)              # propone el siguiente candidato (tasa λ*=7)
    si t < 10:
        U2 ~ Unif(0,1)
        si U2 <= λ(t) / 7:              # λ(t) = 3 + 4/(t+1)
            eventos.agrega(t)           # se acepta
        # si no, se descarta y se sigue generando candidatos
devolver eventos
```

**(b) ¿Cómo mejorarlo?** El problema con usar un solo $\lambda^*=7$ para todo el intervalo es que $\lambda(t)$ **decrece** bastante: en $t=10$, $\lambda(10)=3+\frac{4}{11}\approx 3.36$, menos de la mitad de $\lambda^*=7$. Eso significa que, cerca del final del intervalo, la mayoría de los candidatos propuestos se van a **rechazar** (probabilidad de aceptar $\approx 3.36/7\approx 0.48$ ahí, contra casi $1.0$ cerca de $t=0$), desperdiciando trabajo.

**Mejora estándar:** parte $[0,10]$ en varios subintervalos (por ejemplo $[0,2), [2,4), \dots$) y usa, en cada uno, una cota **local** más ajustada — el máximo de $\lambda(t)$ solo dentro de ese subintervalo (que, como $\lambda$ es decreciente, es simplemente $\lambda(t)$ evaluada en el extremo izquierdo de cada subintervalo). Así, en el último tramo usarías $\lambda^*\approx 3.36$ en vez de $7$, casi duplicando la probabilidad de aceptación ahí. Cuantos más subintervalos uses, más se ajusta la cota a la forma real de $\lambda(t)$ y menos candidatos desperdicias — es un intercambio entre "más precisión" y "más complejidad de implementación".

---

## 8. Procesos de Poisson **bidimensionales**

### 8.1 Definición formal

Un proceso de Poisson espacial de tasa $\lambda$ (puntos por unidad de **área**) sobre el plano cumple: para cualquier región $A$, el número de puntos dentro de $A$ es
$$N(A) \sim \text{Poisson}(\lambda\cdot \text{área}(A))$$
y los conteos en regiones disjuntas son independientes entre sí. Es la generalización natural del proceso de Poisson temporal, cambiando "longitud de intervalo" por "área de región".

**Para qué sirve (Ejercicio 10b):** modela fenómenos donde eventos ocurren dispersos aleatoriamente en el espacio (no en el tiempo) con densidad constante — árboles en un bosque, estrellas en una porción del cielo, defectos en una lámina de material, ubicaciones de accidentes en un mapa, etc. Poder simularlo permite, por ejemplo, generar escenarios sintéticos para probar algoritmos de detección o estimar cantidades (como la probabilidad de que haya al menos $k$ puntos en una subregión) que serían difíciles de calcular analíticamente.

### 8.2 Propiedad clave para simularlo: dado el conteo, los puntos son uniformes

Un hecho fundamental (y muy útil) de los procesos de Poisson espaciales: **si sabes que hay exactamente $n$ puntos dentro de una región $A$, esos $n$ puntos están distribuidos de forma independiente y uniforme sobre $A$.** Esto da un primer algoritmo simple:

**Método A (genera el conteo, luego posiciones uniformes) — bueno para el Ejercicio 8:**
1. Genera $N\sim\text{Poisson}(\lambda\pi R^2)$ (el área del círculo de radio $R$ es $\pi R^2$).
2. Para cada uno de los $N$ puntos, genera su posición de forma independiente en coordenadas polares:
   - Ángulo: $\Theta\sim\text{Unif}(0,2\pi)$.
   - Radio: **atención**, aquí *no* es $\text{Unif}(0,R)$. Como el área de un anillo a distancia $r$ del centro crece proporcionalmente a $r$ (hay "más espacio" lejos del centro que cerca), la densidad correcta del radio de un punto uniforme en el disco es proporcional a $r$. Formalmente, $F_{\text{radio}}(r) = r^2/R^2$ en $(0,R)$ (la probabilidad de estar a distancia $\le r$ del centro es la razón de áreas $\pi r^2/\pi R^2$). Invirtiendo con la sección 2:
   $$\text{Radio} = R\sqrt{U}, \qquad U\sim\text{Unif}(0,1)$$
3. Convierte a cartesianas si lo necesitas: $x=\text{Radio}\cdot\cos\Theta$, $y=\text{Radio}\cdot\sin\Theta$.

Para el Ejercicio 8 ($\lambda=1$, $R=5$): genera $N\sim\text{Poisson}(25\pi)\approx\text{Poisson}(78.5)$, y luego cada punto con $\Theta\sim\text{Unif}(0,2\pi)$, Radio$=5\sqrt{U}$. Grafica los $(x,y)$ resultantes — deberías ver puntos dispersos de forma pareja por todo el disco (ni amontonados en el centro ni en el borde), lo cual es una buena señal visual de que el radio se generó correctamente (si por error usaras $\text{Unif}(0,R)$ para el radio, verías los puntos amontonados cerca del centro — un error clásico, ¡ojo con no cometerlo!).

### 8.3 Método B (secuencial, punto por punto según distancia al centro) — el que pide el Ejercicio 10

El Ejercicio 10 menciona explícitamente generar "valores de $X_1, X_2,\dots$" — esto apunta a un segundo algoritmo, equivalente en distribución al Método A, pero que construye los puntos **uno por uno en orden de cercanía al centro**, sin necesidad de generar $N$ primero. Es elegante porque reutiliza directamente la maquinaria de la sección 6 (proceso de Poisson 1-D).

**La idea:** ordena los puntos del proceso por su distancia al origen: $R_{(1)} \le R_{(2)} \le \cdots$. El número de puntos dentro de radio $r$ es $\text{Poisson}(\lambda \pi r^2)$ — si defines la variable transformada $\tau = \pi r^2$ (proporcional al área), entonces, visto como función de $\tau$, el conteo se comporta **exactamente** como un proceso de Poisson 1-D ordinario de tasa $\lambda$. Por lo tanto, los valores $\tau_i = \pi R_{(i)}^2$ son los tiempos de llegada de un proceso de Poisson de tasa $\lambda$, es decir:
$$\tau_i = \frac{X_1+X_2+\cdots+X_i}{\lambda}, \qquad X_j \overset{iid}{\sim} \text{Exp}(1)$$
Despejando el radio: $\pi R_{(i)}^2 = \tau_i \Rightarrow$
$$R_{(i)} = \sqrt{\frac{X_1+\cdots+X_i}{\lambda\pi}}$$

**Algoritmo (dentro de un círculo de radio $r_{\max}$, tasa $\lambda$):**
```
Y = 0          # suma acumulada de exponenciales
i = 0
puntos = []
repetir:
    X ~ Exp(1)                    # genera X_{i+1}
    Y = Y + X
    R = sqrt(Y / (λ * π))
    si R > r_max:
        parar                     # este punto ya cayó fuera del círculo, se descarta
    Θ ~ Unif(0, 2π)                # ángulo independiente para este punto
    puntos.agrega( (R, Θ) )        # coordenadas polares del punto i-ésimo
devolver puntos
```

**Ejemplo numérico para el Ejercicio 10** ($\lambda=1$, radio $r_{\max}=2$; área total $=\pi\cdot 2^2 = 4\pi\approx 12.566$):

Elige (o genera) valores de $X_1, X_2,\dots \sim\text{Exp}(1)$. Por ejemplo, supón que obtienes:

| $i$ | $X_i$ | $Y_i=\sum_{j\le i}X_j$ | $R_{(i)}=\sqrt{Y_i/\pi}$ | ¿$R_{(i)}\le 2$? |
|---|---|---|---|---|
| 1 | 0.30 | 0.30 | $\sqrt{0.30/\pi}=0.309$ | Sí |
| 2 | 0.55 | 0.85 | $\sqrt{0.85/\pi}=0.520$ | Sí |
| 3 | 1.20 | 2.05 | $\sqrt{2.05/\pi}=0.808$ | Sí |
| 4 | 2.10 | 4.15 | $\sqrt{4.15/\pi}=1.150$ | Sí |
| 5 | 4.80 | 8.95 | $\sqrt{8.95/\pi}=1.688$ | Sí |
| 6 | 5.50 | 14.45 | $\sqrt{14.45/\pi}=2.145$ | **No → parar** |

(Aquí usé $\lambda=1$, por lo que $Y_i/(\lambda\pi)=Y_i/\pi$.) Te quedas con los primeros 5 puntos (el sexto se descarta por caer fuera del círculo). Para cada uno de esos 5, generas de forma independiente un ángulo $\Theta_i\sim\text{Unif}(0,2\pi)$ — por ejemplo, si obtuvieras $U=0.7$ para el primero, $\Theta_1 = 2\pi(0.7)\approx 4.40$ rad — y así tendrías la coordenada polar completa $(R_{(1)},\Theta_1)=(0.309,\ 4.40)$ de tu primer punto simulado, y análogamente para los otros cuatro.

*(Nota: en tu entrega puedes usar tus propios valores elegidos de $U$ o $X_i$; lo importante es que el procedimiento —acumular exponenciales, transformarlas a radio, parar al salir del círculo, generar ángulos independientes— quede claro y aplicado paso a paso con números concretos, tal como pide el enunciado.)*

Los métodos A y B dan, en distribución, **exactamente el mismo tipo de proceso** — son dos caminos válidos al mismo resultado. Usa el Método A (más simple de programar) para el Ejercicio 8, y explica el Método B (el que usa $X_1,X_2,\dots$) para el Ejercicio 10, ya que es el que el enunciado sugiere explícitamente.

---

## 9. Generación de Normales: Box-Muller vs. Método Polar

### 9.1 Box-Muller "directo" (para contraste — no es lo que pide el Ejercicio 9, pero ayuda a entenderlo)

Genera dos uniformes $U_1,U_2$ y produce dos normales estándar independientes:
$$X = \sqrt{-2\ln U_1}\,\cos(2\pi U_2), \qquad Y = \sqrt{-2\ln U_1}\,\sin(2\pi U_2)$$

Es correcto y directo (transformada inversa aplicada en 2 dimensiones a la vez), pero tiene una desventaja práctica: evaluar $\cos$ y $\sin$ es computacionalmente más caro que operaciones aritméticas simples.

### 9.2 Método Polar (Marsaglia) — lo que pide el Ejercicio 9

**Idea:** logra el mismo resultado que Box-Muller (dos normales independientes) pero **evita calcular $\cos$/$\sin$ explícitamente**, sustituyéndolos por razones geométricas que salen "gratis" de generar un punto uniforme dentro del círculo unitario por rechazo.

**Algoritmo:**
1. Genera $U_1,U_2\sim\text{Unif}(0,1)$ y define $V_1=2U_1-1$, $V_2=2U_2-1$ (ahora uniformes en $(-1,1)$ — un punto uniforme en el cuadrado $[-1,1]^2$).
2. Calcula $S=V_1^2+V_2^2$.
3. Si $S>1$ (el punto cayó fuera del círculo unitario), **descarta** y vuelve al paso 1.
4. Si $S\le 1$, calcula:
$$X = V_1\sqrt{\frac{-2\ln S}{S}}, \qquad Y = V_2\sqrt{\frac{-2\ln S}{S}}$$
$X$ y $Y$ son dos normales estándar **independientes**.

**(a) En qué consiste** — ya está arriba: es un método de rechazo geométrico (genera puntos uniformes en el cuadrado, quédate solo con los que caen en el círculo unitario) combinado con una transformación algebraica que reemplaza trigonometría por álgebra.

**(b) Para qué sirve / por qué es preferible a Box-Muller directo:** el punto $(V_1,V_2)$ aceptado, al estar uniformemente distribuido en el círculo unitario, tiene un ángulo uniforme en $(0,2\pi)$ automáticamente — pero en vez de tener que *calcularlo* con $\cos/\sin$ (funciones trascendentales, relativamente costosas), el método polar obtiene el mismo efecto usando directamente $V_1/\sqrt S$ y $V_2/\sqrt S$ como "coseno" y "seno" implícitos, que solo requieren una raíz cuadrada y una división. El costo de esto es tener que rechazar ocasionalmente (cuando $S>1$), pero la probabilidad de aceptar es
$$P(S\le 1) = \frac{\text{área del círculo unitario}}{\text{área del cuadrado } [-1,1]^2} = \frac{\pi}{4}\approx 0.785$$
es decir, aceptas casi el 79% de las veces, así que el rechazo ocasional es mucho más barato que evaluar trigonometría en *cada* muestra. Ese es el intercambio: cambias "trig siempre" por "álgebra casi siempre + rechazo raro".

**(c) Ejemplo numérico paso a paso:**

Elige, por ejemplo, $U_1=0.9$, $U_2=0.85$.
- $V_1 = 2(0.9)-1 = 0.8$
- $V_2 = 2(0.85)-1 = 0.7$
- $S = 0.8^2+0.7^2 = 0.64+0.49 = 1.13$
- Como $S=1.13>1$, **se rechaza este par** y hay que generar uno nuevo.

Segundo intento: $U_1=0.6$, $U_2=0.55$.
- $V_1 = 2(0.6)-1 = 0.2$
- $V_2 = 2(0.55)-1 = 0.1$
- $S = 0.2^2+0.1^2 = 0.04+0.01 = 0.05$
- Como $S=0.05\le 1$, **se acepta**. Calcula el factor común:
$$\sqrt{\frac{-2\ln(0.05)}{0.05}} = \sqrt{\frac{-2(-2.9957)}{0.05}} = \sqrt{\frac{5.9915}{0.05}} = \sqrt{119.83}\approx 10.947$$
- $X = V_1 \times 10.947 = 0.2\times10.947 \approx 2.189$
- $Y = V_2\times 10.947 = 0.1\times10.947\approx 1.095$

Entonces el par de normales estándar obtenido es $(X,Y)\approx(2.189,\ 1.095)$. (En tu entrega puedes usar estos mismos números o elegir los tuyos — lo esencial es mostrar al menos un intento rechazado por $S>1$ y luego el intento aceptado con todos los cálculos intermedios, tal como se hizo aquí.)

---

## 10. Glosario relámpago (para repasar antes del examen o de entregar)

| Término | Qué significa |
|---|---|
| **CDF** $F(x)$ | $P(X\le x)$; siempre entre 0 y 1, no decreciente |
| **PDF** $f(x)$ | Derivada de $F$; describe "densidad de probabilidad" |
| **Transformada inversa** | $X=F^{-1}(U)$; genera exacto, sin desperdicio, **si** puedes invertir $F$ |
| **LGN** | El promedio de muchas simulaciones converge al valor esperado real |
| **Composición** | $F=\sum p_i F_i$: elige al azar cuál $F_i$ usar, luego genera de ella |
| **Aceptación-Rechazo** | Genera de una $g$ fácil "bajo un techo" $cg$, acepta solo si cae bajo la $f$ real |
| **Constante $c$** en rechazo | $c=\max f/g$; mide cuántos intentos promedio necesitas ($c$ = número esperado de intentos) |
| **Proceso de Poisson** | Eventos al azar con tasa $\lambda$; tiempos entre eventos $\sim\text{Exp}(\lambda)$ i.i.d. |
| **Thinning (adelgazamiento)** | Para tasa variable $\lambda(t)$: genera con tasa $\lambda^*$ (cota superior) y acepta cada evento con prob. $\lambda(t)/\lambda^*$ |
| **Poisson bidimensional** | Igual que el de 1-D pero con "área" en vez de "longitud de intervalo" |
| **Método Polar (Marsaglia)** | Genera normales evitando trig, usando rechazo dentro del círculo unitario |

---

## 11. Checklist: qué técnica usar en cada ejercicio

| Ejercicio | Técnica | Idea de una línea |
|---|---|---|
| 1 | Transformada inversa | Invierte la CDF truncada; sin desperdicio, a diferencia del rechazo ingenuo |
| 2 | Composición (teoría) | Elige $F_i$ al azar con prob. $p_i$, luego genera de esa $F_i$ |
| 3 | Composición (aplicada) | Reconoce $F=\sum p_i F_i$ mirando los términos (potencias de $x$, exponenciales, uniformes) |
| 4 | Monte Carlo | Simula Bernoulli + montos exponenciales, repite muchas veces, promedia indicadores |
| 5 | Aceptación-Rechazo | $g=\text{Exp}(1)$ como propuesta para $|Z|$; $c=\sqrt{2e/\pi}$ |
| 6 | Poisson homogéneo | Suma exponenciales de tasa $\lambda$ hasta pasar $T$ |
| 7 | Thinning | $\lambda^*=\lambda(0)=7$; genera Poisson(7) y acepta con prob. $\lambda(t)/7$ |
| 8 | Poisson 2-D (Método A) | $N\sim\text{Poisson}(\lambda\pi R^2)$, luego $\Theta\sim\text{Unif}(0,2\pi)$, Radio$=R\sqrt U$ |
| 9 | Método Polar | Rechazo en el círculo unitario; evita $\cos/\sin$ |
| 10 | Poisson 2-D (Método B) | Acumula exponenciales $\to$ radios $R_{(i)}=\sqrt{Y_i/(\lambda\pi)}$; para cuando $R>r_{\max}$ |

---

Cuando quieras, seguimos con el código real (Python, R, o lo que uses en el curso) ejercicio por ejercicio — este documento ya te da toda la teoría y las fórmulas exactas que vas a necesitar traducir a código.
