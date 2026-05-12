# Ataques adversarios sobre clasificadores convolucionales en CIFAR-10

**Proyecto final — Redes neuronales: arquitecturas y aplicaciones**
Prof. Arles Rodríguez · Universidad Nacional de Colombia · 2026-I

Autor: Andrés Angel

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github//jairoandresangelcardenas-tech/dversarial-attacks-cifar10/blob/main/proyecto_final_parte1.ipynb)

---

## Resumen

Este repositorio contiene la replicación de tres ataques adversarios canónicos sobre clasificadores convolucionales entrenados en CIFAR-10:

- **FGSM** — Fast Gradient Sign Method (Goodfellow et al., 2014)
- **PGD** — Projected Gradient Descent (Madry et al., 2018)
- **C&W** — Carlini & Wagner (2017)

Los modelos evaluados incluyen una ResNet-18 entrenada estándar y un modelo robusto del leaderboard de [RobustBench](https://robustbench.github.io/), permitiendo cuantificar el efecto del *adversarial training* como defensa.

## Estructura

```
.
├── proyecto_final_parte1.ipynb    # Notebook principal: pipeline + FGSM
├── reporte_proyecto_final.tex     # Reporte técnico en LaTeX
├── reporte_proyecto_final.pdf     # Versión compilada
├── README.md                      # Este archivo
└── requirements.txt               # Dependencias
```

## Ejecución rápida

### Opción 1: Google Colab (recomendado)

Click en el badge "Open in Colab" arriba. Activa GPU en *Entorno de ejecución → Cambiar tipo de entorno de ejecución → T4 GPU*. La primera celda instala las dependencias automáticamente.

### Opción 2: Local

```bash
git clone https://github.com/TU_USUARIO/TU_REPO.git
cd TU_REPO
pip install -r requirements.txt
jupyter notebook proyecto_final_parte1.ipynb
```

## Formulación matemática

Dado un clasificador $f_\theta : [0,1]^d \to \mathbb{R}^K$ con pesos $\theta$ fijos, el problema del atacante es:

$$\delta^* = \arg\max_{\|\delta\|_p \leq \epsilon,\ x+\delta \in [0,1]^d} \mathcal{L}(f_\theta(x+\delta), y)$$

Los tres ataques implementados son aproximaciones distintas a este problema no convexo con restricciones convexas:

| Ataque | Estrategia | Norma |
|---|---|---|
| FGSM | Linealización one-shot vía dualidad Hölder | $\ell_\infty$ |
| PGD | Ascenso de gradiente proyectado iterativo | $\ell_\infty$, $\ell_2$ |
| C&W | Formulación Lagrangiana con reparametrización tanh | $\ell_2$ |

Ver Sección 4 del reporte para el desarrollo detallado.

## Resultados (preliminares — Parte I)

Configuración: CIFAR-10, $\epsilon = 8/255$ (norma $\ell_\infty$), test set completo.

| Modelo | Limpia | FGSM | PGD-40 | C&W |
|---|---|---|---|---|
| ResNet-18 estándar | TBD | TBD | TBD | TBD |
| WRN-28-10 robusto (RobustBench) | TBD | TBD | TBD | TBD |

## Plan de trabajo

- [x] **Parte I (12 mayo)**: pipeline CIFAR-10 + ResNet-18 + FGSM, curva accuracy-vs-$\epsilon$, visualizaciones.
- [ ] **Parte II (11 junio)**: PGD, C&W, comparación contra modelo robusto, análisis de transferibilidad entre arquitecturas.

## Referencias principales

1. Goodfellow, Shlens, Szegedy. *Explaining and Harnessing Adversarial Examples*. ICLR 2015.
2. Madry et al. *Towards Deep Learning Models Resistant to Adversarial Attacks*. ICLR 2018.
3. Carlini, Wagner. *Towards Evaluating the Robustness of Neural Networks*. IEEE S&P 2017.
4. Croce et al. *RobustBench: a standardized adversarial robustness benchmark*. NeurIPS 2021.

## Licencia

MIT
