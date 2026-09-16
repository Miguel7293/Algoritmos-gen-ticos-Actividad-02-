# Actividad 02: algoritmos genéticos en aprendizaje de máquina

Repositorio de los ejemplos y del resumen ejecutivo de la Actividad 02. **Presentación: 16/09/2026.**

## Objetivo y alcance

Comprender y demostrar tres aplicaciones de los algoritmos genéticos (AG): seleccionar características,
optimizar hiperparámetros y buscar la arquitectura de una red neuronal. Cada ejemplo hace explícitos
los siete pasos solicitados: representación, inicialización, aptitud, selección, cruzamiento,
mutación y terminación. Las soluciones son las mejores observadas dentro del presupuesto de búsqueda;
no se garantiza encontrar el óptimo global.

La [consigna original](Actividad%2002.md) se conserva como referencia. Los tres cuadernos incluyen
código, explicaciones, salidas de una ejecución completa, gráficos y exportación de resultados.

## Entregables y acceso a Google Colab

| Entregable | Archivo | Abrir y ejecutar |
| --- | --- | --- |
| Selección de características (nuevo) | [ag_feature_selection_colab.ipynb](ag_feature_selection_colab.ipynb) | [Abrir en Colab](https://colab.research.google.com/github/Miguel7293/Algoritmos-gen-ticos-Actividad-02-/blob/main/ag_feature_selection_colab.ipynb) |
| Optimización de hiperparámetros (revisado) | [ag_hpo_colab.ipynb](ag_hpo_colab.ipynb) | [Abrir en Colab](https://colab.research.google.com/github/Miguel7293/Algoritmos-gen-ticos-Actividad-02-/blob/main/ag_hpo_colab.ipynb) |
| Neuroevolución (revisado) | [Untitled4.ipynb](Untitled4.ipynb) | [Abrir en Colab](https://colab.research.google.com/github/Miguel7293/Algoritmos-gen-ticos-Actividad-02-/blob/main/Untitled4.ipynb) |
| Resumen ejecutivo, 2 páginas | [Descargar PDF](output/pdf/Resumen_ejecutivo_Actividad_02.pdf) | Entregar en el aula virtual |
| Evidencia de ejecución | [Resultados JSON](resultados/) | Métricas, versiones e historiales |

Se conserva el nombre `Untitled4.ipynb` para mantener la continuidad con el cuaderno original;
su título interno identifica el ejemplo de neuroevolución.

**Repositorio:** [https://github.com/Miguel7293/Algoritmos-gen-ticos-Actividad-02-](https://github.com/Miguel7293/Algoritmos-gen-ticos-Actividad-02-).

## Cómo ejecutarlo

1. Abre el enlace de Colab del ejemplo. Si deseas modificarlo, guarda una copia en tu Drive.
2. Usa un entorno de Python 3 con CPU; no hace falta GPU.
3. Selecciona **Entorno de ejecución > Ejecutar todas** desde una sesión nueva.
4. Revisa los gráficos y la comparación final con la referencia. La última celda guarda un JSON
   en la carpeta `resultados` del entorno; puedes descargarlo desde el panel Archivos de Colab.

Cada cuaderno es autónomo y genera o carga sus datos sin descargas externas ni archivos del repositorio.
Colab suele incluir NumPy, scikit-learn y Matplotlib; hay una celda comentada de instalación si falta alguna.
Las salidas incluidas se verificaron localmente con un kernel de Jupyter, no en una sesión remota de Colab.
Las versiones de Colab pueden producir pequeñas diferencias.

Para ejecutar localmente, con Python 3.11 o compatible:

```bash
python -m venv .venv
# Activa el entorno según tu sistema operativo.
python -m pip install -r requirements.txt
python -m pip install notebook
python -m notebook
```

Abre un cuaderno y ejecuta todas sus celdas en orden. No mezcles variables entre cuadernos.

## Qué realiza cada ejemplo

### 1. Selección de características

Wine: 178 observaciones, 13 variables y 3 clases. Se mantiene fija una regresión logística (`C=1`)
con estandarización dentro del pipeline. El cromosoma contiene 13 bits y determina las variables utilizadas.
Se repara cualquier máscara vacía activando una variable. Se reservan 36 muestras de test;
la aptitud se calcula mediante CV estratificada de 3 pliegues sobre las otras 142.

`fitness = exactitud_CV - 0.02 * (variables_seleccionadas / 13)`.

La penalización favorece subconjuntos pequeños y se fija antes de la búsqueda. No confundir fitness
penalizado con exactitud. Referencia: el mismo modelo usando todas las variables.

### 2. Optimización de hiperparámetros

Random Forest sobre Wine, con la misma división desarrollo/test y CV de 3 pliegues.
Los genes son `n_estimators`, `max_depth`, `min_samples_split`, `max_features` y `criterion`.
La aptitud es la exactitud media de CV, sin penalización. Referencia: Random Forest con parámetros
predeterminados y la misma semilla. El AG no entrena directamente los árboles ni sus divisiones.

### 3. Neuroevolución

300 puntos sintéticos uniformes en `[-1, 1]²`, con etiqueta `x1*x2 > 0`.
Se separan 180 muestras de entrenamiento, 60 de validación y 60 de prueba.
El cromosoma define `capa1 ∈ {2,4,8}`, `capa2 ∈ {0,2,4}` y `activacion ∈ {relu,tanh}`:
18 arquitecturas posibles. `capa2=0` elimina la segunda capa oculta.
La aptitud es la exactitud de validación. Los pesos se entrenan mediante L-BFGS, con hasta
2000 iteraciones y 30000 evaluaciones de función; **el AG evoluciona arquitectura, no pesos**.
Referencia: una capa de 4 neuronas con tanh. Al terminar, ambos modelos se reentrenan con las
240 muestras de desarrollo y se evalúan en el test reservado.

## Ciclo de los algoritmos genéticos

| Paso | Selección de características | Hiperparámetros | Neuroevolución |
| --- | --- | --- | --- |
| Representación | Máscara de 13 bits | 5 genes discretos | Capas y activación |
| Inicialización | 16 individuos: todas las variables + 15 aleatorios | 10 individuos aleatorios | 8 individuos aleatorios |
| Aptitud | CV menos penalización de tamaño | Exactitud CV | Exactitud de validación |
| Selección | Torneo de 3 | Torneo de 3 | 2 padres entre los 5 mejores |
| Cruzamiento | Un punto; probabilidad 0.8 | Uniforme; probabilidad 0.8 | Uniforme, siempre |
| Mutación | 1/13 por bit | 0.25 por gen | 0.20 por gen |
| Elitismo | 2 individuos | 2 individuos | 2 individuos |
| Terminación | 10 generaciones | 6 generaciones | 10 generaciones |

La población inicial se registra como generación 0. El historial incluye también la población final.
La caché evita reevaluar cromosomas idénticos. El elitismo preserva la mejor aptitud, pero la media puede bajar.

## Resultados obtenidos en la ejecución verificada

| Ejemplo | Referencia CV/validación | Ganador CV/validación | Referencia test | AG test |
| --- | ---: | ---: | ---: | ---: |
| Características | 97.19 % | 100.00 % | 97.22 % | 97.22 % |
| Hiperparámetros | 95.11 % | 95.80 % | 100.00 % | 100.00 % |
| Neuroevolución | 98.33 % | 100.00 % | 96.67 % | 100.00 % |

- **Características:** 7/13 variables: alcohol, ash, alcalinity_of_ash, flavanoids, color_intensity, hue, proline. Aptitud penalizada: 0.9892.
- **Hiperparámetros ganadores:** `{"n_estimators": 80, "max_depth": 12, "min_samples_split": 3, "max_features": "log2", "criterion": "entropy"}`.
- **Arquitectura ganadora:** capas ocultas `[8, 2]`, activación `relu`.
- **Cambio de test frente a la referencia:** características +0.00 puntos porcentuales, hiperparámetros +0.00 puntos porcentuales, neuroevolución +3.33 puntos porcentuales.

Las dos primeras columnas son exactitudes de selección, no estimaciones independientes de generalización.
Las tareas y particiones son distintas: no se debe usar esta tabla para ordenar los tres métodos entre sí.
Un empate o un resultado menor en prueba también es un resultado válido. Son demostraciones pequeñas,
con una semilla por ejemplo y sin validación cruzada anidada ni comparación con otros buscadores.
No se demuestra significancia estadística ni superioridad general de los AG.

## Revisión y comprobaciones realizadas

- Se creó el ejemplo que faltaba de selección de características.
- Hiperparámetros: se añadió un test independiente, comparación de referencia, caché y una explicación
  que no promete mejora continua ni ausencia de estancamiento.
- Neuroevolución: se añadieron secciones explicativas, test independiente, referencia, copias para elitismo,
  registro de la última generación y mutaciones a valores distintos. Se cambió Adam por L-BFGS
  para este pequeño problema y se contabilizan las advertencias de convergencia.
- Advertencias de convergencia en neuroevolución: búsqueda 0,
  reentrenamiento de referencia 0, ganador 0.
- Los tres cuadernos se ejecutaron de principio a fin en kernels nuevos, sin errores de celdas,
  con gráficos incorporados. Se comprobaron la separación de índices, máscaras válidas y conservación
  de la mejor aptitud por elitismo. Evidencia: [verificacion.json](resultados/verificacion.json).
- Versiones de cálculo: Python 3.11.7, NumPy 2.4.6,
  scikit-learn 1.9.1 y Matplotlib 3.11.2.

Los resultados revisados sustituyen las antiguas salidas del cuaderno: no son directamente comparables
porque ahora hay datos reservados para prueba y cambió el optimizador de la red.

## Material complementario

Los HTML existentes contienen laboratorios interactivos para apoyar la explicación. Utilizan ejemplos
propios; sus métricas no son las métricas de los cuadernos incluidas en el PDF. Para usarlos, descarga
o clona el repositorio y abre los archivos en tu navegador; GitHub muestra el código fuente del HTML.

- [Guía de la actividad](01_guia_actividad.html).
- [Selección de características](02_seleccion_caracteristicas.html).
- [Optimización de hiperparámetros](03_optimizacion_hiperparametros.html).
- [Neuroevolución](04_neuroevolucion.html).

## Entrega y exposición

La consigna evalúa explicación y funcionamiento de cada ejemplo (18 puntos) y PDF/README (2 puntos).
El PDF tiene **dos páginas** e incluye el enlace a este repositorio. El equipo debe subirlo al aula virtual
y estar preparado para ejecutar y explicar los tres ejemplos. La entrega al aula virtual no se realiza
automáticamente desde este repositorio.

Reparto sugerido para los cuatro integrantes: un ejemplo por integrante y el cuarto a cargo del
ciclo común, comparación de resultados y conclusiones. Todos deben poder explicar el conjunto.

## Referencias

- [Scikit-learn: validación cruzada](https://scikit-learn.org/stable/modules/cross_validation.html).
- [Scikit-learn: conjunto Wine](https://scikit-learn.org/stable/modules/generated/sklearn.datasets.load_wine.html).
- [Scikit-learn: MLPClassifier](https://scikit-learn.org/stable/modules/generated/sklearn.neural_network.MLPClassifier.html).
