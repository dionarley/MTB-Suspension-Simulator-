# MTB-Suspension-Simulator-
MTB Suspension Simulator 





```
MTB Suspension Simulator
│
├── Frame
│   ├── Head tube
│   ├── Seat tube
│   ├── Bottom bracket
│   └── Main pivot
│
├── Rear suspension
│   ├── Rear axle
│   ├── Upper link
│   ├── Lower link
│   ├── Rocker
│   └── Shock
│
├── Kinematics
│   ├── Axle path
│   ├── Instant center
│   ├── Wheel travel
│   ├── Shock travel
│   ├── Leverage ratio
│   ├── Progressivity
│   ├── Anti-squat
│   ├── Anti-rise
│   └── Pedal kickback
│
└── Visualization
    ├── Frame
    ├── Moving links
    ├── Wheel
    ├── Shock
    └── Graphs
```

A parte fundamental é representar cada barra por **dois pontos/pivôs**:

```
             O A
              \
               \  upper link
                O B
                 \
                  \       O axle
                   \------●
                    lower link
                   /
                  /
             O C
```

Matematicamente, cada barra é uma restrição de distância:

(x2−x1)2+(y2−y1)2=L2(x_2-x_1)^2+(y_2-y_1)^2=L^2

Então, em vez de simplesmente dizer:

```
rear_wheel_y += ...
```

o programa realmente resolve a posição das barras.

### Exemplo

Suponha:

```
typedef struct {
    double x;
    double y;
} Point;

typedef struct {
    Point a;
    Point b;
    double length;
} Link;
```

E:

```
typedef struct {
    Point main_pivot;
    Point upper_pivot;
    Point lower_pivot;

    Point rear_axle;

    Link upper_link;
    Link lower_link;
} Suspension;
```

O simulador poderia receber:

```
Main pivot     (120, 300)
Upper pivot    (180, 250)
Lower pivot    (160, 350)

Upper link     80 mm
Lower link     100 mm

Rear axle      ...
```

e então variar a posição do mecanismo:

```
0 mm travel
 ↓
10 mm
 ↓
20 mm
 ↓
30 mm
 ...
 ↓
160 mm
```

Para cada posição ele calcula:

```
rear axle X/Y
shock length
wheel travel
instant center
leverage ratio
```

O **instant center** é particularmente importante: em sistemas multi-link ele muda conforme a suspensão comprime, sendo determinado pela interseção das linhas associadas aos links.

----------

## Não começar C puro

Para esse projeto :

**Python + NumPy + Matplotlib** primeiro.

Depois, se quisermos transformar em software:

```
Python
   ↓
motor matemático
   ↓
JSON/CSV
   ↓
GUI
```

ou posteriormente:

```
C/C++
   ↓
solver
   ↓
GTK/SDL/OpenGL
```

### O primeiro MVP

**Versão 0.1 — somente cinemática:**

```
                    MTB SUSPENSION SIMULATOR

       FRAME
    ┌─────────────┐
    │             │
    │      ● P1   │
    │       \     │
    │        \    │
    │         ● P2
    │          \
    │           \________ ● AXLE
    │          /
    │       ● P3
    │
    └─────────────────────

Travel:       82.0 mm
Shock:        29.4 mm
Leverage:     2.79

IC:
X = ...
Y = ...

Progression:
18.4 %

Anti-squat:
...
```

E gerar automaticamente:

```
          Axle Path
              ↑
              │       •
              │     •
              │   •
              │ •
              └────────────→

       Wheel Travel vs Shock Travel


       Leverage Ratio
              ↑
          3.0 │\
          2.5 │ \
          2.0 │  \
          1.5 │   \__
              └──────────→
                Travel
```

Podemos adicionar **massa do ciclista, gravidade, mola/amortecedor, SAG e forças**, mas e importante  separar isso da cinemática inicialmente.

**Para um quadro MTB específico**, uma foto lateral do quadro é extremamente útil para transformar isso em um modelo real — principalmente para identificar os pivôs, rocker e pontos de fixação do shock.

Com uma foto lateral + algumas medidas, dá para montar o primeiro **solver de suspensão 2D** em Python e depois portar o núcleo para C.


**Scott Spark** é um ótimo caso para esse simulador, porque podemos modelar a cinemática real do quadro em vez de fazer apenas uma animação genérica.

Para montar um modelo confiável, preciso de uma **vista lateral da Spark que você quer simular** (idealmente com a roda traseira e o sistema de links visíveis). Se tiver uma foto, desenho técnico ou captura com as medidas, envie aqui.

Se  não tiver as medidas, podemos começar com a geometria aproximada e depois calibrar.

### O modelo sugerido

```
             ┌──────── quadro ────────┐
             │                        │
             │        ● rocker        │
             │       /       \        │
             │      /         \       │
             │  ●───            ───●  │
             │ pivô                 shock
             │                       │
             └──────────●────────────┘
                        eixo
```

O simulador calcularia, para cada posição do eixo traseiro:

-   **trajetória do eixo traseiro**
-   curso da roda
-   curso do amortecedor
-   relação de alavancagem (_leverage ratio_)
-   progressividade
-   posição do _instant center_
-   variação do entre-eixos
-   variação do _chainstay_
-   rotação dos links
-   _anti-squat_
-   _anti-rise_
-   _pedal kickback_

E visualmente:

```
        COMPRESSÃO

        0%       25%       50%       75%       100%

         ●         ●         ●         ●          ●
       /           \         \          \          \
      ●             ●         ●          ●          ●
         \             \        \         \          \
          ●             ●        ●         ●          ●
        eixo traseiro → trajetória
```

### Estrutura do projeto

Começar em **Python + NumPy + Matplotlib**, porque fica muito mais fácil validar a matemática.

```
scott-spark-simulator/
├── geometry.py
├── kinematics.py
├── suspension.py
├── solver.py
├── visualization.py
├── models/
│   └── spark.json
├── tests/
│   └── test_kinematics.py
├── examples/
│   └── spark_analysis.py
└── README.md
```

**Proxima stack**: GUI e, para desempenho/portabilidade, colocar o solver em C/C++.
