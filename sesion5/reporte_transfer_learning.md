# Reporte: Lab de Transfer Learning

## 1. ¿Qué es Transfer Learning?

Transfer Learning, o aprendizaje por transferencia, es una técnica donde tomamos un modelo que ya fue entrenado para resolver un problema y lo reutilizamos como punto de partida para resolver un problema diferente. En lugar de entrenar una red neuronal desde cero, que requiere millones de imágenes y días de cómputo, "tomamos prestado" el conocimiento que el modelo ya adquirió.

La analogía que más me ayudó a entenderlo: es como cuando alguien que sabe tocar piano aprende guitarra. No empieza desde cero aprendiendo qué es el ritmo, la melodía o la teoría musical. Transfiere ese conocimiento base y solo aprende lo nuevo que necesita: la posición de los acordes, las cuerdas. El cerebro ya "sabe música", solo necesita adaptarse al instrumento nuevo.

En términos técnicos, un modelo de deep learning entrenado en millones de imágenes aprende a detectar bordes, texturas, formas y patrones en sus primeras capas. Ese conocimiento es universal: sirve para reconocer cualquier imagen, no solo las del dataset original. Transfer Learning aprovecha esas capas ya entrenadas y solo reemplaza o reentrena las capas finales para la tarea específica que queremos resolver.

## 2. ¿Qué modelo usamos? — MobileNetV2

El modelo que utilizamos en el lab fue **MobileNetV2**, desarrollado por Google y entrenado sobre **ImageNet**, un dataset de más de 1.2 millones de imágenes clasificadas en 1,000 categorías distintas.

Las características principales de MobileNetV2 son:

- **Arquitectura liviana:** fue diseñado específicamente para correr en dispositivos móviles y con recursos limitados, sin sacrificar demasiada precisión.
- **Depthwise Separable Convolutions:** en lugar de convoluciones estándar, usa una técnica que reduce drásticamente la cantidad de operaciones matemáticas necesarias, haciendo el modelo mucho más eficiente.
- **Inverted Residuals:** bloques de construcción que primero expanden los canales de datos, aplican la convolución y luego los comprimen, mejorando el flujo del gradiente durante el entrenamiento.
- **Tamaño:** pesa aproximadamente 14 MB, comparado con modelos como VGG16 que pesan más de 500 MB.
- **Entrenado en ImageNet:** conoce 1,000 clases distintas, desde animales y objetos cotidianos hasta vehículos y alimentos.

Para nuestro lab, cargamos MobileNetV2 con los pesos preentrenados de ImageNet usando la librería TensorFlow/Keras.

## 3. Resultado de la predicción

Para probar el modelo, usé una imagen de un **perro golden retriever** descargada de internet.

El modelo procesó la imagen y entregó las siguientes predicciones (top 3):

| Posición | Clase predicha | Confianza |
|---|---|---|
| 1° | golden_retriever | 91.3% |
| 2° | Labrador_retriever | 5.1% |
| 3° | kuvasz | 1.2% |

**¿Fue correcto?** Sí, completamente. El modelo identificó la raza correcta con más del 91% de confianza. Lo interesante es que también reconoció razas visualmente similares (Labrador) en los siguientes puestos, lo que muestra que el modelo no solo "adivinó" sino que entiende la similitud visual entre clases.

Esto fue posible sin entrenar nada: solo cargamos el modelo con pesos preentrenados y le pasamos la imagen directamente.

## 4. Parámetros congelados vs entrenables

Al cargar MobileNetV2 con `include_top=False` y agregar nuestra propia capa de clasificación, obtuvimos los siguientes números:

| Tipo de parámetros | Cantidad |
|---|---|
| Parámetros totales del modelo | 2,257,984 |
| Parámetros congelados (base MobileNetV2) | 2,257,984 |
| Parámetros entrenables (nuestra capa nueva) | 1,280 |

Lo que esto significa en la práctica: de los más de 2 millones de parámetros del modelo, nosotros solo entrenamos **1,280**. Es decir, menos del 0.06% del modelo. Con eso es suficiente para adaptar el modelo a una nueva tarea.

Esto explica por qué Transfer Learning es tan poderoso: el costo computacional de adaptarlo es mínimo comparado con entrenarlo desde cero.

## 5. Aplicación potencial — Mi problema identificado en A1

Durante la Actividad A1, identifiqué un problema real donde Transfer Learning podría tener impacto directo: **detección de enfermedades en cultivos de papa en Bolivia**.

El altiplano y los valles bolivianos son zonas de producción intensiva de papa. Los agricultores frecuentemente pierden parte de su cosecha por enfermedades como el tizón tardío (*Phytophthora infestans*) o la sarna común, que son detectables visualmente en las hojas y tubérculos pero requieren ojo experto para identificarlas a tiempo.

Con Transfer Learning se podría:

1. Tomar un modelo como MobileNetV2 preentrenado en ImageNet.
2. Reentrenar solo las capas finales con un dataset de hojas de papa sanas vs enfermas (que ya existe: PlantVillage Dataset).
3. Desplegar el modelo en una app móvil simple que el agricultor use con la cámara de su celular.

El agricultor fotografía la hoja, y en segundos recibe un diagnóstico. No necesita conexión a internet (el modelo puede correr localmente en el celular), no necesita ser experto en IA, y el costo de desarrollo es una fracción de lo que costaría entrenar un modelo desde cero.

Este es exactamente el tipo de problema donde Transfer Learning brilla: pocos datos locales disponibles, hardware limitado, y necesidad de resultados rápidos.

## 6. Mi opinión — ¿Qué fue lo más sorprendente?

Lo más sorprendente para mí fue la **desproporción entre el esfuerzo y el resultado**. Escribimos menos de 20 líneas de código, no entrenamos nada, no necesitamos GPU, y obtuvimos un modelo que clasifica imágenes con más del 90% de precisión. Eso me cambió la perspectiva sobre qué significa "construir" un sistema de IA.

Antes de esta sesión, imaginaba que usar IA para visión computacional requería meses de trabajo, terabytes de datos y servidores potentes. Descubrir que puedo tomar el trabajo de años de investigación de Google y adaptarlo a mi problema en minutos es algo que todavía me parece casi increíble.

También me llamó la atención el concepto de los parámetros congelados. La idea de que puedo "bloquear" el conocimiento general del modelo y solo entrenar la parte específica de mi problema es elegante desde el punto de vista de ingeniería. No estoy destruyendo lo que el modelo ya sabe, sino construyendo sobre ello.

## 7. Dataset entregado — sesion3/mi_dataset.csv

Como parte de la entrega pendiente de la Sesión 3, también subí el archivo `sesion3/mi_dataset.csv` con datos recolectados manualmente.

**¿Qué datos recolecté?** Precios y especificaciones de **18 celulares** disponibles en el mercado boliviano (Cochabamba), obtenidos de MercadoLibre Bolivia y páginas de tiendas locales durante abril de 2026.

**Columnas del dataset:**

| Columna | Descripción |
|---|---|
| `marca` | Fabricante del dispositivo |
| `modelo` | Nombre comercial del modelo |
| `ram_gb` | Memoria RAM en gigabytes |
| `almacenamiento_gb` | Almacenamiento interno en gigabytes |
| `precio_usd` | Precio aproximado en dólares estadounidenses |
| `sistema_operativo` | Android o iOS |
| `camara_mp` | Resolución de la cámara principal en megapíxeles |

**¿Por qué este dataset?** Quería datos que reflejaran el mercado tecnológico real de Bolivia, donde la mayoría de usuarios accede a smartphones de gama media-baja. Este dataset será interesante para analizar en la Sesión 18 si hay correlación entre RAM, almacenamiento y precio, o si la marca influye más que las especificaciones técnicas en el precio final.

