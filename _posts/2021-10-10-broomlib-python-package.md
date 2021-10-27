---
layout: post
title:  "Broomlib. Librería de procesado previo a Machine Learning"
author: ana
categories: [ Visualization, Machine Learning ]
image: assets/images/2021-10-10-broomlib-python-package.png
---

¿Tienes un deadline de Machine Learning imposible de cumplir? Esta línea de código puede ayudarte:
```
pip install broomlib
```

Broomlib es un paquete de Python que busca facilitar el preprocesado de datos antes de la aplicación de modelos de Machine Learning. La librería abarca tres áreas:
- Visualización inicial del dataset
- Limpieza de datos
- Preprocesado de los datos mediante Machine learning

Como colaboradora de esta librería, principalmente en el módulo de visualización, me gustaría destacar algunas funciones que pueden ser muy útiles antes de aplicar un modelo de Machine Learning sobre todo si el tiempo apremia.

## `corr_bars`
Muestra un gráfico de barras horizontales de las variables más correlacionadas entre sí, de pares a pares. 
Presents a Pandas horizontal bar plot of the most correlated feature pairs and their correlation coefficient.
Esta función solo utiliza variables númericas.

En todas las gráficas del módulo de visualización de broomlib hay dos parámetros comunes:
- `figsize`: determina el tamaño y proporción de la imagen.
- `style`: estilo de gráfico matplotlib. Para buscar los estilos disponibles puede usarse: `plt.style.available`.

```
from broomlib import visualization as vis
import seaborn as sns
mpg = sns.load_dataset('mpg')
vis.corr_bars(mpg, threshold=0.6, figsize=(13, 6))
```

![](/assets/images/2021-10-10-broomlib_corr_bars.png)


## `outliers_mahalanobis_plot`
Este gráfico muestra los outliers del dataset. Para determinarlos utiliza la distancia de Mahalanobis para comparar cada punto con la distribución *Chi Square*. Los puntos más extremos son los que llevan los índices y su número se determina mediante el parámetro `extreme_points`. Esta función solo utiliza variables númericas.

```
from broomlib import visualization as vis
from sklearn import datasets
import pandas as pd
diabetes = datasets.load_diabetes()
df = pd.DataFrame(diabetes.data)
vis.outliers_mahalanobis_plot(df, extreme_points=10, figsize=(15,7), style='ggplot')
```

![](/assets/images/2021-10-10-broomlib_outliers_mahalanobis_plot.png)

## `pca_analisis`
Esta función elimina las variables con baja variabilidad de un dataset dado. La idea principal es realizar un análisis de *feature importance* basado en la variabilidad de cada variable que permite determinar qué *features* aportarán menos información al futuro modelo de Machine Learning para eliminarlas.

Internamente utiliza el análisis de componentes principales [PCA](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.PCA.html) de sklearn. Hay que recordar que `sklearn.decomposition.PCA` centra los datos pero no los escala por ello es necesario escalarlos previamente y que se encuentren en las mismas unidades.
Ejemplo:
```
from broomlib import MachineLearning as ml

ml.pca_analisis(df)
```

## `broomResample`
El objetivo de `broomResample` es afrontar el no poco frecuente problema de tener un target desbalanceado. Una posible solución sería hacer un *oversampling* de la clase minoritaria, muchas veces un duplicado de registros que no aporta información al modelo.

Para la función aplica una mezcla de técnicas. En primer lugar hace un *undersampling* de la clase mayoritaria que puede ser personalizado mediante el párametro `sampling_strategy_o`. Después, se aplica un *oversampling* mediante la técnica SMOTE *Synthetic Minority Oversampling Technique* que también puede ser parametrizada mediante el argumento `sampling_strategy_o`.

Ejemplo:

```
from broomlib.MachineLearning import broomResample
from imblearn.over_sampling import SMOTE
from imblearn.under_sampling import RandomUnderSampler
from imblearn.pipeline import Pipeline

X, y = make_classification(n_samples=10000, n_features=2, n_redundant=0, 
                           n_clusters_per_class=1, weights=[0.99], flip_y=0, random_state=1)

counter = Counter(y)
print(counter)

for label, _ in counter.items():
  row_ix = np.where(y == label)[0]
  plt.scatter(X[row_ix, 0], X[row_ix, 1], label=str(label))
  plt.legend()
  plt.show()

X_resampled, y_resampled = broomResample(X,y)

print(f'Resampled dataset samples per class {{Counter(y_resampled)}}')

for label, _ in counter.items():
row_ix = np.where(y_resampled == label)[0]
plt.scatter(X_resampled[row_ix, 0], X_resampled[row_ix, 1], label=str(label))
plt.legend()
plt.show()
```

```
Resampled dataset samples per class Counter({{0: 900, 1: 900}})
```