|           |     | Universidad |              | del Valle      | de  | Guatemala     |     |                   |               |
| --------- | --- | ----------- | ------------ | -------------- | --- | ------------- | --- | ----------------- | ------------- |
|           |     | Facultad    | de           | Ingeniería     |     |               |     |                   |               |
|           |     | Ciencia     | de           | la Computación |     | y Tecnologías |     | de la información |               |
|           |     | CC2017      | - Modelación |                | y   | Simulación    |     |                   | Ciclo 2, 2026 |
| Ejercicio | 1   |             |              |                |     |               |     |                   |               |
Sea X una variable aleatoria exponencial con media 1. Proporcione un algoritmo eficiente para
simular una variable aleatoria cuya distribución es la distribución condicional de X dado que X <
| 0,05. Es | decir, | su función |     | de densidad |     | es  |     |     |     |
| -------- | ------ | ---------- | --- | ----------- | --- | --- | --- | --- | --- |
e−x
|     |     |     |     | f(x) | =   |     | ,   | 0 < x < 0,05. |     |
| --- | --- | --- | --- | ---- | --- | --- | --- | ------------- | --- |
1−e−0,05
Genere 1000 de estas variables y utilícelas para estimar E[X | X < 0,05]. Luego determine el
| valor exacto | de  | E[X     | | X | < 0,05].     |     |     |     |     |     |
| ------------ | --- | ------- | --- | ------------ | --- | --- | --- | --- | --- |
| Ejercicio    | 2   | (Método | de  | composición) |     |     |     |     |     |
Suponga que es relativamente fácil generar variables aleatorias a partir de cualquiera de las dis-
F i = 1,...,n.
tribuciones i , ¿Cómo podríamos generar una variable aleatoria con función de dis-
tribución
n
∑︂
|     |     |     |     |     |     | F(x) | = p | F (x) |     |
| --- | --- | --- | --- | --- | --- | ---- | --- | ----- | --- |
i i
i=1
|           | p i = | 1,...,n, |     |         |     |           |      |            |     |
| --------- | ----- | -------- | --- | ------- | --- | --------- | ---- | ---------- | --- |
| donde     | i ,   |          | son | números | no  | negativos | cuya | suma es 1? |     |
| Ejercicio | 3     |          |     |         |     |           |      |            |     |
Utilizando el resultado del Ejercicio 2, proporcione algoritmos para generar variables aleatorias a
| partir de | las siguientes |     | distribuciones. |     |     |     |     |     |     |
| --------- | -------------- | --- | --------------- | --- | --- | --- | --- | --- | --- |
x+x3+x5
| (a) F(x) | =   |     |     | , 0 ≤ | x ≤ 1 |     |     |     |     |
| -------- | --- | --- | --- | ----- | ----- | --- | --- | --- | --- |
3
⎧
1−e−2x+2x
⎪
|          |     | ⎪   |     | 0   | < x | < 1 |     |     |     |
| -------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|          |     | ⎨   | 3   |     |     |     |     |     |     |
| (b) F(x) | =   |     |     |     |     |     |     |     |     |
3−e−2x
⎪
|     |     | ⎪   |     | 1   | < x | < ∞ |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
⎩
3
|           |     | n   |     |         |     |       |        | n     |     |
| --------- | --- | --- | --- | ------- | --- | ----- | ------ | ----- | --- |
|           |     | ∑︂  |     |         |     |       |        | ∑︂    |     |
| (c) F(x)  | =   | α   | xi, | 0 ≤ x ≤ | 1,  | donde | α ≥ 0, | α = 1 |     |
|           |     |     | i   |         |     |       | i      | i     |     |
|           |     | i=1 |     |         |     |       |        | i=1   |     |
| Ejercicio | 4   |     |     |         |     |       |        |       |     |
Una compañía de seguros contra siniestros tiene 1000 asegurados, cada uno de los cuales pre-
sentará una reclamación en el próximo mes de manera independiente con probabilidad 0.05. Su-
poniendo que los montos de las reclamaciones son variables aleatorias exponenciales indepen-
dientes con media $800, utilice simulación para estimar la probabilidad de que la suma de estas
| reclamaciones |     | exceda | $50,000. |     |     |     |     |     |     |
| ------------- | --- | ------ | -------- | --- | --- | --- | --- | --- | --- |
Página 1

|           |     | Universidad |              | del Valle      | de Guatemala |               |     |                   |               |
| --------- | --- | ----------- | ------------ | -------------- | ------------ | ------------- | --- | ----------------- | ------------- |
|           |     | Facultad    | de           | Ingeniería     |              |               |     |                   |               |
|           |     | Ciencia     | de           | la Computación |              | y Tecnologías |     | de la información |               |
|           |     | CC2017      | - Modelación |                | y Simulación |               |     |                   | Ciclo 2, 2026 |
| Ejercicio | 5   |             |              |                |              |               |     |                   |               |
Escriba un programa que genere variables aleatorias normales utilizando el método del Ejemplo
| 5f (método | de  | rechazo | con | una distribución |     | exponencial |     | de tasa 1). |     |
| ---------- | --- | ------- | --- | ---------------- | --- | ----------- | --- | ----------- | --- |
| Ejercicio  | 6   |         |     |                  |     |             |     |             |     |
Escriba un programa que genere las primeras T unidades de tiempo de un proceso de Poisson
λ.
con tasa
| Ejercicio | 7   |     |     |     |     |     |     |     |     |
| --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
(a) Escriba un programa que utilice el algoritmo de adelgazamiento (thinning) para generar las
primeras 10 unidades de tiempo de un proceso de Poisson no homogéneo con función de
intensidad
4
|     |     |     |     |     |     | λ(t) | = 3+ |     |     |
| --- | --- | --- | --- | --- | --- | ---- | ---- | --- | --- |
t+1
(b) Proponga una manera de mejorar el algoritmo de adelgazamiento para este ejemplo.
| Ejercicio | 8   |     |     |     |     |     |     |     |     |
| --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Escriba un programa para generar los puntos de un proceso de Poisson bidimensional dentro de
un círculo de radio R, y ejecute el programa para λ = 1 y R = 5. Grafique los puntos obtenidos.
Material de apoyo: Ejemplo 5f — Generación de una variable aleatoria normal
Para generar una variable aleatoria normal estándar Z (es decir, con media 0 y varianza 1), se
Z
| observa | primero | que | el valor | absoluto | de  | tiene | función | de densidad |     |
| ------- | ------- | --- | -------- | -------- | --- | ----- | ------- | ----------- | --- |
√︃
2
e−x2/2,
|     |     |     |     | f(x) | =   |     |     | 0 < x < ∞. |     |
| --- | --- | --- | --- | ---- | --- | --- | --- | ---------- | --- |
π
Esta densidad se genera usando el método de rechazo, tomando como densidad auxiliar g la
| exponencial | de  | media | 1,  | es decir | g(x) = | e−x, | 0 < x < | ∞. Entonces |     |
| ----------- | --- | ----- | --- | -------- | ------ | ---- | ------- | ----------- | --- |
f(x) √︁
|     |     |     |     |     |     | =   | 2/π ex−x2/2, |     |     |
| --- | --- | --- | --- | --- | --- | --- | ------------ | --- | --- |
g(x)
| cuyo máximo |     | ocurre | en x | = 1, lo que | da    | la constante |      |         |     |
| ----------- | --- | ------ | ---- | ----------- | ----- | ------------ | ---- | ------- | --- |
|             |     |        |      |             |       | f(x)         | f(1) | √︁      |     |
|             |     |        |      | c           | = máx |              | =    | = 2e/π. |     |
|             |     |        |      |             | x     | g(x)         | g(1) |         |     |
Página 2

|     |      | Universidad |     |              | del Valle   | de  | Guatemala     |     |                   |     |     |               |
| --- | ---- | ----------- | --- | ------------ | ----------- | --- | ------------- | --- | ----------------- | --- | --- | ------------- |
|     |      | Facultad    |     | de           | Ingeniería  |     |               |     |                   |     |     |               |
|     |      | Ciencia     |     | de la        | Computación |     | y Tecnologías |     | de la información |     |     |               |
|     |      | CC2017      |     | - Modelación |             | y   | Simulación    |     |                   |     |     | Ciclo 2, 2026 |
|     | f(x) |             | {︃  | (x−1)2}︃     |             |     |               |     |                   |     |     |               |
Como = exp − ,seobtieneelsiguientealgoritmoparagenerarelvalorabsoluto
|     | cg(x)      |           |     |           | 2           |           |          |     |     |     |     |     |
| --- | ---------- | --------- | --- | --------- | ----------- | --------- | -------- | --- | --- | --- | --- | --- |
| de  | una normal | estándar: |     |           |             |           |          |     |     |     |     |     |
|     | Paso       | 1: Genere |     | Y, una    | exponencial |           | con tasa | 1.  |     |     |     |     |
|     | Paso       | 2: Genere |     | un número |             | aleatorio | U.       |     |     |     |     |     |
−1)2/2},
Paso 3: Si U ≤ exp{−(Y haga X = Y. En caso contrario, regrese al Paso 1.
Una vez generado X (que se distribuye como el valor absoluto de una normal estándar), se ob-
tieneZ haciendoqueseaigualmenteprobablequevalgaX o−X (estosedecideconunnúmero
| aleatorio | adicional: |     | si  | U ≤ | 1/2 entonces |     | Z = X; | si no, Z | = −X). |     |     |     |
| --------- | ---------- | --- | --- | --- | ------------ | --- | ------ | -------- | ------ | --- | --- | --- |
PuededemostrarseademásquelacondicióndeaceptacióndelPaso3esequivalentea−logU ≥
(Y −1)2/2, y como −logU es exponencial de tasa 1, se obtiene la siguiente versión equivalente,
que genera simultáneamente una normal estándar y — como subproducto — una exponencial
| independiente |      | de        | tasa | 1:              |     |     |          |     |     |     |     |     |
| ------------- | ---- | --------- | ---- | --------------- | --- | --- | -------- | --- | --- | --- | --- | --- |
|               | Paso | 1: Genere |      | Y , exponencial |     |     | con tasa | 1.  |     |     |     |     |
1
|     | Paso | 2: Genere |     | Y , exponencial |     |     | con tasa | 1.  |     |     |     |     |
| --- | ---- | --------- | --- | --------------- | --- | --- | -------- | --- | --- | --- | --- | --- |
2
|     |            |       | Y −(Y   | −1)2/2  |     | > 0, | Y    | = Y −(Y | −1)2/2 |            |     |                 |
| --- | ---------- | ----- | ------- | ------- | --- | ---- | ---- | ------- | ------ | ---------- | --- | --------------- |
|     | Paso       | 3: Si | 2       | 1       |     |      | haga | 2       | 1      | y continúe | al  | Paso 4. En caso |
|     | contrario, |       | regrese | al Paso | 1.  |      |      |         |        |            |     |                 |
U
|     | Paso | 4: Genere |     | un número |     | aleatorio | y   | haga |     |     |     |     |
| --- | ---- | --------- | --- | --------- | --- | --------- | --- | ---- | --- | --- | --- | --- |
{︄
|     |     |     |     |     |     |     | Y   | si  | U ≤ 1/2 |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ------- | --- | --- | --- |
1
Z =
|     |     |     |     |     |     |     | −Y  | si  | U > 1/2 |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | ------- | --- | --- | --- |
1
√︁
Z (normalestándar)eY (exponencialdetasa1)resultanindependientes.Dadoquec = 2e/π ≈
1,32, en promedio el algoritmo requiere generar 1,64 exponenciales y calcular 1,32 cuadrados por
cadanormalgenerada.(Sisedeseaunanormaldemediaµyvarianzaσ2,bastacontomarµ+σZ.)
| Ejercicio |     | 9   |     |     |     |     |     |     |     |     |     |     |
| --------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
Explique el método polar para generar variables aleatorias normales. Su explicación debe incluir:
(a) En qué consiste el método (idea general y algoritmo paso a paso).
(b) Para qué sirve (qué problema resuelve y por qué es preferible frente al uso directo de las
|     | transformaciones |     |     | de  | Box–Muller). |     |     |     |     |     |     |     |
| --- | ---------------- | --- | --- | --- | ------------ | --- | --- | --- | --- | --- | --- | --- |
(c) Un ejemplo numérico: elija valores de U y U y aplique el algoritmo paso a paso hasta
|     |     |     |     |     |     |     |     | 1 2 |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
obtener el par de normales X,Y (si el primer par (V ,V ) cae fuera del círculo unitario, elija
|     |            |     |     |              |     |     |     |     | 1 2 |     |     |     |
| --- | ---------- | --- | --- | ------------ | --- | --- | --- | --- | --- | --- | --- | --- |
|     | un segundo |     | par | y continúe). |     |     |     |     |     |     |     |     |
Página 3

|           | Universidad |     | del Valle      | de Guatemala  |                   |               |
| --------- | ----------- | --- | -------------- | ------------- | ----------------- | ------------- |
|           | Facultad    |     | de Ingeniería  |               |                   |               |
|           | Ciencia     | de  | la Computación | y Tecnologías | de la información |               |
|           | CC2017      |     | - Modelación   | y Simulación  |                   | Ciclo 2, 2026 |
| Ejercicio | 10          |     |                |               |                   |               |
Explique el proceso de Poisson bidimensional y el algoritmo para simularlo dentro de una región
| circular. | Su explicación |     | debe incluir: |     |     |     |
| --------- | -------------- | --- | ------------- | --- | --- | --- |
(a) En qué consiste (definición formal del proceso y el algoritmo de simulación paso a paso).
(b) Para qué sirve (qué tipo de fenómenos se pueden modelar con este proceso y por qué es
| útil | poder | simularlo). |     |     |     |     |
| ---- | ----- | ----------- | --- | --- | --- | --- |
(c) Un ejemplo numérico: para λ = 1 y r = 2, elija valores de X ,X ,... (o de los números
1 2
aleatorios necesarios para generarlos) y aplique el algoritmo paso a paso hasta obtener las
| coordenadas |     | polares | de los | puntos simulados. |     |     |
| ----------- | --- | ------- | ------ | ----------------- | --- | --- |
Página 4