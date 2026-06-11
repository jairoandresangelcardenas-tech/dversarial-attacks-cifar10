# Ataques adversarios sobre clasificadores convolucionales en CIFAR-10

**Proyecto final — Redes neuronales: arquitecturas y aplicaciones**
Prof. Arles Rodríguez · Universidad Nacional de Colombia · 2026-I

Autor: Andrés Ángel

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/jairoandresangelcardenas-tech/dversarial-attacks-cifar10/blob/main/proyecto_final_completo.ipynb)

---

## Resumen

Este repositorio estudia dos grandes familias de amenazas adversarias sobre clasificadores convolucionales entrenados en CIFAR-10, replicando ataques canónicos de la literatura y midiendo su efecto sobre una ResNet-18.

| | Familia | Ataques | Momento |
|---|---|---|---|
| **Parte I-II** | Evasión | FGSM, PGD, C&W | Inferencia (el modelo ya está entrenado) |
| **Parte III** | Entrenamiento | Backdoor, label-flipping, clean-label | Sobre los datos (antes de entrenar) |

## Estructura

```
.
├── proyecto_final_completo.ipynb    # Notebook con las tres partes
├── reporte_final_completo.pdf       # Reporte técnico (10 páginas)
├── reporte_final_completo.tex       # Fuente LaTeX
├── presentacion.html                # Presentación de 10 minutos (HTML)
├── README.md                        # Este archivo
└── requirements.txt                 # Dependencias
```

## Ejecución

### Google Colab (recomendado)

Click en el badge "Open in Colab" arriba. Activa GPU en *Entorno de ejecución → Cambiar tipo de entorno de ejecución → GPU*. La primera celda instala las dependencias. El notebook completo (3 partes) corre en ~45-60 min en T4; menos en GPUs superiores.

### Local

```bash
git clone https://github.com/jairoandresangelcardenas-tech/dversarial-attacks-cifar10.git
cd dversarial-attacks-cifar10
pip install -r requirements.txt
jupyter notebook proyecto_final_completo.ipynb
```

## Formulación matemática (ataques de evasión)

Dado un clasificador `f_θ` con pesos fijos, el problema del atacante es:

```
δ* = argmax  L(f_θ(x + δ), y)
     ‖δ‖_p ≤ ε
```

Los tres ataques de evasión son aproximaciones distintas a este problema:

| Ataque | Estrategia | Norma |
|---|---|---|
| FGSM | Linealización en un paso (dualidad Hölder) | ℓ∞ |
| PGD | Ascenso de gradiente proyectado iterativo | ℓ∞, ℓ₂ |
| C&W | Formulación Lagrangiana con reparametrización tanh | ℓ₂ |

## Resultados

### Parte I-II — Ataques de evasión (ResNet-18, ε = 8/255)

| Modelo | Limpia | FGSM | PGD-40 | C&W |
|---|---|---|---|---|
| ResNet-18 estándar | 93.39% | 16.49% | 0.00% | 0.00% |

Se confirma la jerarquía de fortaleza **FGSM < PGD ≤ C&W**. La implementación manual de FGSM y PGD coincide con `torchattacks`.

**Curva de robustez FGSM:**

| ε (escala 0–255) | Accuracy |
|---|---|
| 0 (limpia) | 93.39% |
| 1/255 | 53.95% |
| 2/255 | 34.13% |
| 4/255 | 23.03% |
| 8/255 (estándar) | 16.49% |
| 16/255 | 10.91% |

**Análisis de norma de perturbación (subset 1000):**

| Ataque | ‖δ‖₂ medio | ‖δ‖∞ medio |
|---|---|---|
| FGSM | 1.724 | 0.0314 |
| PGD-40 | 1.365 | 0.0314 |
| C&W | 0.183 | variable |

C&W logra misclassification total con una perturbación ℓ₂ ~9× menor que FGSM.

**Transferibilidad** (adversarios generados contra ResNet-18, evaluados en una SimpleCNN distinta):

| Adversario | White-box (ResNet-18) | Transfer (SimpleCNN) |
|---|---|---|
| FGSM | 16.49% | 40.10% |
| PGD | 0.00% | 38.21% |

Los adversarios transfieren parcialmente (SimpleCNN limpia: 85.5%), evidencia de *non-robust features*.

### Parte III — Ataques en entrenamiento

**Backdoor (BadNets)** — trigger 3×3, clase objetivo `airplane`:

| Envenenamiento | Clean accuracy | Attack Success Rate |
|---|---|---|
| 0.1% | 90.95% | 60.71% |
| 0.5% | 91.45% | 92.54% |
| 1.0% | 90.20% | 93.80% |
| 5.0% | 93.61% | 97.38% |
| 10.0% | 90.35% | 97.48% |

Con solo 0.5% de envenenamiento se logra >92% de ASR manteniendo la clean accuracy intacta — el backdoor es prácticamente indetectable por validación estándar.

**Label-flipping** (degradación de disponibilidad):

| Etiquetas volteadas | Clean accuracy |
|---|---|
| 0% | 91.34% |
| 10% | 88.74% |
| 20% | 87.56% |
| 40% | 82.01% |

**Clean-label (Poison Frogs):** la muestra envenenada (etiqueta correcta) se acerca 95% al objetivo en el espacio de features, con perturbación visual moderada (ℓ₂ = 1.11).

## Conclusión

La vulnerabilidad adversaria es una propiedad estructural de los clasificadores entrenados de forma convencional: se manifiesta tanto en inferencia (un atacante con acceso a la entrada engaña al modelo con perturbaciones imperceptibles) como en la cadena de suministro de datos (un atacante con acceso al entrenamiento incrusta comportamientos maliciosos indetectables).

## Stack técnico

PyTorch · torchvision · torchattacks · CIFAR-10 · ResNet-18

## Referencias principales

1. Goodfellow, Shlens, Szegedy. *Explaining and Harnessing Adversarial Examples*. ICLR 2015.
2. Madry et al. *Towards Deep Learning Models Resistant to Adversarial Attacks*. ICLR 2018.
3. Carlini, Wagner. *Towards Evaluating the Robustness of Neural Networks*. IEEE S&P 2017.
4. Gu, Dolan-Gavitt, Garg. *BadNets: Identifying Vulnerabilities in the ML Supply Chain*. 2017.
5. Shafahi et al. *Poison Frogs! Targeted Clean-Label Poisoning Attacks*. NeurIPS 2018.
6. Ilyas et al. *Adversarial Examples Are Not Bugs, They Are Features*. NeurIPS 2019.

## Licencia

MIT
