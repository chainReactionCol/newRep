Expresiones regulares: regex

Para encontrar algunos patrones y poder usar esto para extraer información se puede usar la librería de python re

Se importa de la siguiente manera:

import re

Para construir una expresión regular que corresponde a un patrón se puede hacer el siguiente proceso:

Supongamos los siguientes registros en un campo llamado gender:

1.) Death Metal/Thrash Metal (1988-93)| Grunge/Hard Rock (1994-present)
2.) Black/Death Metal/Doom Metal (early)| Drone/Dark Ambient/Noise (later)
3.)Thrash/Groove Metal (early)| Brutal/Death Metal with hardcore influences (later)
4.)Thrash Metal (early)| Thrash/Death Metal (later)
5.) Death Metal/Grindcore

En estos ejemplos se puede observar que se usa el caracter "|" para separar los cambios de genero en las bandas y el caracter "/" para separar los diferentes tipos de generos que las bandas pueden tocar, además se usa el string "(early)" para referirse a una etapa temprana de la banda o en su defecto un periodo de tiempo, por ejemplo "(1988-93)"... asimismo, para periodos intermedios y posteriores se usan los string "(mid)" y "(later)"

Supongamos que queremos asignar un booleano para distinguir aquellas bandas que iniciaron haciendo death metal y para ello vamos a buscar aquellas que contengan Death Metal en su descripción antes del primer "|" que es el separador de etapas de la banda. Una alternativa sencilla es construir una patrón de epxresión regular que tenga las siguientes caracteristicas:

1.) Que contenga el string "Death Metal"
2.) Que contenga cualquier caracter excepto el salto de línea
3.) Que ese carácter aparezca de 0 a n veces
4.) Que coincide con el menor número de cáracteres
5.) Que busque cualquier dígito en la cadena de texto entre el 0 y el 9
6.) Que busque de forma literal el string "|"

Para lograr esto en este ejemplo simple, se puede usar la siguiente descomposición:


Las expresiones regulares son una forma de buscar patrones específicos en una cadena de texto. En Python, las expresiones regulares se utilizan a menudo con el módulo re.

Aquí está la descomposición de esta expresión regular:

r: En Python, r antes de una cadena de texto indica que es una cadena "cruda". Esto significa que los caracteres de escape (como \n para una nueva línea, \t para una tabulación, etc.) no se tratan como tales, y se consideran como caracteres normales. En el caso de las expresiones regulares, se utiliza r para evitar que Python trate los \ como caracteres de escape.

Death Metal: Esto busca la cadena exacta "Death Metal".

: Esto representa un espacio.

\\(: Dado que los paréntesis ( y ) tienen un significado especial en las expresiones regulares (se utilizan para definir grupos), necesitamos escaparlos con \\ para buscar literalmente un "(".

.*?: . representa cualquier carácter (excepto el salto de línea), * significa "cero o más veces", y ? hace que la coincidencia sea "perezosa", lo que significa que coincide con la menor cantidad posible de caracteres.

\d: Esto busca cualquier dígito (0-9).

.*?: Esto tiene el mismo significado que antes, busca cualquier carácter (excepto el salto de línea) cero o más veces, y hace que la coincidencia sea perezosa.

\\): Esto busca literalmente un ")".

.*?: Esto tiene el mismo significado que antes, busca cualquier carácter (excepto el salto de línea) cero o más veces, y hace que la coincidencia sea perezosa.

\\|: Esto busca literalmente un "|".

en ese orde de ideas la expresión regular que podemos construir es:

r"Death Metal.*?\|"

Con el patrón identificado podemos construir una función python que haga la busqueda:



 def startAsDeath(x):
    patterns = [
                r"Death Metal.*?\|",  
               ]

    for pattern in patterns:
        match = re.search(pattern, x)
        if match is not None:  
            return True

    return False  

 Esta función accede a una lista de patrones (en este caso uno, pero pueden usarse varios si las estructuras son muy irregulares) y usa el metódo search de la librería re para buscar, si no hay coincidencias, asigna el booleano a False.



Ahora bien, para aplicar esta función a los registros podemos usar una función lambda sobre las filas y estas tienen la siguiente estructura:

df['nombre_columna'] = df['nombre_columna'].apply(lambda x: #operación sobre x)


Ejemplos:

Supón que tienes una columna 'edad' y quieres aumentar todas las edades en 1:

df['edad'] = df['edad'].apply(lambda x: x + 1)


Si tienes una columna 'nombre' y quieres convertir todos los nombres a minúsculas:

df['nombre'] = df['nombre'].apply(lambda x: x.lower())


Supongamos que tienes un DataFrame de Pandas df con una columna 'edad'. Quieres crear una nueva columna 'mayor_de_edad' que sea True si la edad es 18 o más y False si no lo es. Esto se puede lograr con una función lambda y un condicional:

df['mayor_de_edad'] = df['edad'].apply(lambda x: True if x >= 18 else False)

En este código, lambda x: True if x >= 18 else False es una función que devuelve True si x (la edad) es 18 o más y False en caso contrario.

La función apply() aplica esta función lambda a cada elemento de la columna 'edad', creando una nueva columna 'mayor_de_edad' con los resultados.

En el caso de las bandas, uso dos funciones lambda para construir y actualizar la nueva columna:


bands['started_as_death'] = bands.apply(lambda row: startAsDeath(str(row['genre'])) if row['has_transitions'] else False, axis=1)

bands['started_as_death'] = bands.apply(lambda row: True if row['genre'].strip() == 'Death Metal' else row['started_as_death'], axis=1)


En este código, la función lambda recibe cada fila del DataFrame (en lugar de cada valor de 'genre' o 'started_as_death'). Comprueba si el valor de 'genre' en esa fila es igual a 'Death Metal' (usando row['genre'].strip() para eliminar los espacios en blanco al principio y al final). Si es así, devuelve True. Si no es así, devuelve el valor actual de 'started_as_death' para esa fila.

El argumento axis=1 en la función apply() indica que la función se debe aplicar a las filas del DataFrame. De esta manera, puedes acceder a todas las columnas de cada fila dentro de tu función lambda.

-----------------------------------------------------------

Conteos de valores

Puedes contar la cantidad de False en una columna en Pandas utilizando la función value_counts(). Este método cuenta la ocurrencia de cada valor único en la columna. 

false_count = bands['started_as_death'].value_counts()[False]

En este código, bands['started_as_death'].value_counts() devuelve una serie con el recuento de cada valor único en la columna 'started_as_death'. Luego, [False] selecciona el recuento para False.

Si hay la posibilidad de que no haya ningún valor False en la columna y quieres evitar un error en ese caso, puedes utilizar el método get() de la serie, que permite especificar un valor por defecto si el índice no se encuentra:

En este código, si no hay ningún valor False en la columna, get(False, 0) devolverá 0 en lugar de provocar un error.

Ahora quiero crear un booleano para aquellas bandas que terminaron haciendo Death Metal y algún otro genero, las bandas que terminaron haciendo algún genero distinto al death metal al final del estudio

 Para el primer caso, haremos lo siguiente

Buscaremos la palabra Death Metal despúes del cáracter "|"  en este caso el pattern es

r"\|.*Death Metal"

En este caso se uso "\|" porque en una expresión regular, | es un carácter especial que significa "o". Para buscar literalmente el carácter |, necesitas escaparlo con un \. 

Si Death Metal fuese la última palabra en todos los registros debería ajustarse la expresión regular para tener esto en cuenta como sigue:

 r"\|.*Death Metal\s*$"


Ahora bien, como debemos enfocarnos en el último "|" ya que pueden haber varios cambios, se debe hacer uso de un lookahead:

Un lookahead en expresiones regulares es una forma de especificar una especie de "condición futura" que debe cumplirse para que el patrón coincida.

¿Qué es un lookahead?

El lookahead es un tipo de aserción en las expresiones regulares que "mira hacia adelante" para ver si algún texto coincide con el patrón especificado, pero sin consumir caracteres del texto de entrada. Esto significa que después de que el motor de la expresión regular evalúa el lookahead, su posición en el texto de entrada no ha avanzado; el motor está aún en el mismo lugar para continuar la evaluación de la expresión regular.

Aquí está el desglose:

Afirmación: Un término para una comprobación que se realiza en una ubicación específica en el texto, pero que no coincide con ningún carácter por sí mismo. Las afirmaciones pueden ser positivas (deben cumplirse para que el patrón coincida) o negativas (deben no cumplirse para que el patrón coincida).

Lookahead: Este es un tipo de afirmación que comprueba si un patrón coincide "hacia adelante" en el texto. Pero aunque el motor de expresiones regulares comprueba si el patrón lookahead coincide, no "consume" los caracteres del texto de entrada. Esto significa que el motor no avanza en el texto; se queda en la misma ubicación, listo para comprobar el siguiente elemento en la expresión regular.

Por ejemplo, si tuvieras el texto "123abc" y la expresión regular \d+(?=abc), la expresión regular coincidiría con "123", porque es seguido por "abc". Pero después de que se evalúa la afirmación lookahead, el motor de la expresión regular todavía está al final del "123", listo para seguir comprobando cualquier otra cosa que pueda haber en la expresión regular.

Consumir caracteres: En una expresión regular, cuando se hace coincidir un elemento con parte del texto de entrada, se dice que ese elemento "consume" los caracteres con los que coincide. Así que, por ejemplo, si tienes el texto "123abc" y la expresión regular \d+abc, la expresión regular consume todos los caracteres (es decir, coincide con todo el texto). Pero con un lookahead, los caracteres que coinciden con la parte lookahead de la expresión regular no se consumen.


Sintaxis de un lookahead:

Lookahead positivo (?=...)

Lookahead negativo (?!...)

Los "..." representan el patrón que se busca en los caracteres.

Lookahead positivo ((?=...)): Esto coincide si la subexpresión ... coincide hacia adelante a partir de la posición actual en la cadena de caracteres. Pero aunque la subexpresión coincida, eso no consume caracteres en la cadena de caracteres. En otras palabras, el motor de la expresión regular no avanza. Por ejemplo, en la cadena de caracteres "abc", la expresión regular a(?=b) coincide con la "a", pero solo si la "a" es seguida por una "b".

Lookahead negativo (?!...): Esto es lo contrario de un lookahead positivo. Coincide solo si la subexpresión ... NO coincide hacia adelante a partir de la posición actual en la cadena de caracteres. Por ejemplo, en la cadena de caracteres "abc", la expresión regular a(?!b) no coincide con nada, porque la "a" es seguida por una "b".

Ejemplos de lookaheads:


Comprobar si una contraseña tiene al menos un dígito, una letra minúscula y una letra mayúscula, con una longitud de al menos ocho caracteres:

password = "Password123"
pattern = r'^(?=.*\d)(?=.*[a-z])(?=.*[A-Z]).{8,}$'

match = re.search(pattern, password)
if match:
    print("La contraseña es segura.")
else:
    print("La contraseña no es segura.")


En este ejemplo se usa la siguiente expresión regular que explicaré en detalle:

r'^(?=.*\d)(?=.*[a-z])(?=.*[A-Z]).{8,}$'

eL caracter "^" se usa para indicar que busque coincidencias en toda la cadena de texto despúes de dicho caracter. En este caso, se coloca al principio de toda la expresión regular para buscar coincidencias en todo el input, si no se colocase este caracter, entonces la expresión regular buscará coincidencias ebn cualquier parte de la cadena de entrada y no desde el principio. En este ejemplo, entonces  el ^ está al principio de la expresión, por lo que se aplica a toda la expresión. Eso significa que la expresión regular solo coincidirá si toda la cadena de texto cumple con los requisitos especificados.

Seguido a esto se encuentran los siguientes lookaheads:

(?=.*\d) este lookahead busca que desde el principio de la cadena para todos los carateres 0 o más veces con dígítos entre el 0 y el 9 (al menos uno)

(?=.*[a-z]) este lookahedad busca que exista al menos una letra minúscula

(?=.*[A-Z]) este lookahead busca que exista al menos una letra mayúscula

.{8,}? 



. : Este es un carácter especial en las expresiones regulares que coincide con cualquier carácter excepto un salto de línea, a menos que se especifique la opción re.DOTALL, en cuyo caso también coincidirá con los saltos de línea.

{8,} : Esto se conoce como un cuantificador. Significa "coincide con el elemento anterior 8 o más veces". En este caso, el elemento anterior es ., por lo que {8,} aplicado a . significa "coincide con cualquier carácter 8 o más veces".

? : Este carácter tiene dos usos en las expresiones regulares. Cuando se coloca después de un carácter o un grupo de caracteres, actúa como un cuantificador que significa "cero o una vez". Pero cuando se coloca después de otro cuantificador como *, +, ?, o {}, cambia el comportamiento de ese cuantificador para que sea no ávido. Un cuantificador ávido intentará coincidir con tanto texto como sea posible, mientras que un cuantificador no ávido (también conocido como "lazy" o "reluctante") intentará coincidir con la menor cantidad de texto posible. Así que en .{8,}?, el ? está haciendo que el cuantificador {8,} sea no ávido.

Entonces, la expresión regular .{8,}? significa "coincide con cualquier secuencia de al menos 8 caracteres, pero no más de lo necesario para hacer coincidir el resto de la expresión regular". Este tipo de coincidencia es útil cuando estás tratando de extraer una parte específica de un texto y quieres asegurarte de que no consumes más caracteres de los necesarios.

----------------------------------------------------------------------------------------

Cuantificadores ávidos y no ávidos

Empezaremos con un ejemplo básico para ilustrar la diferencia entre los cuantificadores ávidos y no ávidos. Supongamos que tienes el siguiente texto:


<div> Hola </div> <div> Mundo! </div>

Y quieres extraer el texto que se encuentra entre las etiquetas <div>.

Cuantificador ávido

Si usas la expresión regular <div>.*</div>, el .* es un cuantificador ávido que intentará coincidir con la mayor cantidad de texto posible:


import re

text = "<div> Hola </div> <div> Mundo! </div>"
pattern = "<div>.*</div>"

matches = re.findall(pattern, text)
print(matches)  # Outputs: ['<div> Hola </div> <div> Mundo! </div>']


Como puedes ver, la expresión regular coincide con todo el texto, porque el .* consume todo el texto que puede entre el primer <div> y el último </div>.


Cuantificador no ávido

Ahora, si cambias el cuantificador a no ávido usando .*?, intentará coincidir con la menor cantidad de texto posible:

import re

text = "<div> Hola </div> <div> Mundo! </div>"
pattern = "<div>.*?</div>"

matches = re.findall(pattern, text)
print(matches)  # Outputs: ['<div> Hola </div>', '<div> Mundo! </div>']

Como puedes ver, ahora la expresión regular encuentra dos coincidencias: una para cada par de etiquetas <div>. El .*? consume solo el texto necesario para que coincida con cada par de etiquetas.

Estos ejemplos deberían ayudarte a entender cómo los cuantificadores ávidos y no ávidos pueden afectar a tus expresiones regulares, y cómo puedes usarlos para extraer de manera efectiva los datos que necesitas.

-------------------------------------------------------------------------------------------------

Volviendo al caso del análisis de las bandas de Death Metal, nos encontramos con el hecho que deseamos buscar las transiciones en las bandas y en este caso es poder conocer en que genero se unicaron al final y sabemos que el caracter "|" es el delimitador de transiciones

import ipywidgets as wd
from ipywidgets import interact


@interact
def tuFiltro(filtro=list(tuDataFrame.columnaDeTuDataFrame.unique())):
    return tuDataFrame[tuDataFrame.columnaDeTuDataFrame==filtro] 

Para poder analizar los datos de forma estructurada, procedí a hacer fusiones (merging) de las tablas a partir de los id que identifican los discos, las bandas y  las review, con lo que construí 
tres dataframes partiendo de los booleanos 	has_transitions, started_as_death y ends_as_death con esto, los dataframes contienen la información de los reviews según si la banda hizo o no cambios de genero lo cual está definido por estos booleanos, luego se procedió a promediar los reviews por cada banda y se redujenron los dataframes para estudiar la evolución de las scores de reviews de acuerdo al pais de procedencia.

-----------------------------------------------------------------------------------------------------

SOBRE LOS BOXPLOT, VIOLIN PLOTS Y FINALMENTE LOS RAINPLOTS


Cuando se buscan representar datos para realiazr comparaciones entre categorías y poder distinguir información relavante sobre la forma de la distribución de los datos, por ejemplo en experimentos con grupos de control algunos tipos de gráficos pueden llevar a conclusiones equivocadas. Por ejemplo los gráficos de barras pueden llevar a conclusiones equivocadas, pues dos datasets obtenidos de distribuciones diferentes y con la misma media y la misma desviación estandar pueden hacer pensar que se tienen medidas equivalentes aunque provienen de distribuciones completamente distintas. Los box plot pueden proveer información adicional acerca de los quartiles, pero se pierde o es dificil visualizar información comparativa acerca de la presencia de picos o de la relación entre la enlongación de la distribución y la presencia de picos en la misma. Asimismo, los boxplots no son tan utilies para hacer comparaciones entre grupos, mientras que los violin plots si son mas aporpiados para ese tipo de analisis.   

¿Qué es un violin plot?

Para responder esta pregunta, primero debemos saber que es un KDE plot o density plot. Un KDE es una representación de un suavizado de la estimación de la función de densidad de probabilidad de una variable aleatoria a partir de  un set finito de datos, básicamente a partir de inferencias sobre la muestra, básicamente atribuyendo pesos a los datos del dataset. Para esto se usan Kernels, que pueden pensarse como "funciones de peso"  y existen varios tipos de Kernels, entre los cuales usaremos, destacaremos o pensaremos en tres: kernel normal, kernel de Epanenchnikov y el kernel triangular. Para counstruir la curva de densidad final, basicamente se apilan las curvas de densidad de todos los datos, de esta forma al haber más datos acumulados en una región, la altura de la curva de densidad será mayor, los violin plots por el contrario combinan un diagrama de caja (box plot) y un KDE (Kernel Density Estimate, una forma de estimación de densidad) para mostrar la distribución de los datos. Los diagramas de violín son particularmente útiles cuando se quieren comparar múltiples categorías de datos simultáneamente.

En un violin plot, las curvas de densidad individuales se construyen alrededor de las líneas centrales en lugar de apilarse en las líneas de base.

La línea central en un violin plot generalmente representa la mediana de los datos.
Las curvas alrededor de la línea central muestran la densidad estimada de los puntos de datos, es decir, en qué partes del rango de valores los datos son más densos. Esta densidad se refleja en el ancho del "violín": un violín más ancho indica una mayor densidad de datos.
Esto significa que en lugar de tener que mirar a la altura de las barras (como en un histograma) para entender dónde se encuentran la mayoría de los datos, puedes mirar el ancho del "violín".

En resumen, los violin plots proporcionan una representación visual de la misma información que un box plot, pero también añaden la densidad de los datos, similar a un histograma, lo que puede dar una visión más detallada de la distribución de los datos.  Para ilustra esa idea, te comparto el siguiente código que puedes correr en un notebook de jupyter:

import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

# Generar datos de muestra
np.random.seed(0)
data1 = np.random.normal(loc=0, scale=1, size=100)
data2 = np.random.normal(loc=2, scale=2, size=100)
df = pd.DataFrame({
    'Category': ['Category 1']*100 + ['Category 2']*100,
    'Value': np.concatenate([data1, data2])
})

# Gráfico de densidad apilado
plt.figure(figsize=(10,5))
plt.subplot(1,2,1)
sns.kdeplot(data=df, x="Value", hue="Category", fill=True)
plt.title('Density Plot (Stacked)')

# Violin Plot
plt.subplot(1,2,2)
sns.violinplot(data=df, x='Category', y='Value')
plt.title('Violin Plot')
plt.tight_layout()
plt.show()



En la gráfica de densidad apilada, los gráficos de densidad para cada categoría se apilan uno encima del otro en la línea de base. La altura de cada curva de densidad representa la densidad de los datos.
-------------------------------------------------------------------------------------

ALGUNOS ASPECTOS IMPORTANTES SOBRE LOS VIOLINPLOTS Y LOS KERNEL


Para dibujar medio violinplot usando seaborn, podemos usar un código como el siguiente:

# plotting the clouds
f, ax = plt.subplots(figsize=(7, 5))
dy="group"; dx="score"; ort="h"; pal = sns.color_palette(n_colors=1)
ax=pt.half_violinplot( x = dx, y = dy, data = df, palette = pal,
 bw = .2, cut = 0.,scale = "area", width = .6, inner = None, 
 orient = ort)

en este caso se alamcena los datos en el dataframe df que tiene las columnas "score" y "group" Por otro lado, la variable ort se usa para definir la disposición del plot (vertical u horizontal) y adicional se definen los siguientes parámetros:

f, ax = plt.subplots(figsize=(7, 5)): Se está creando una figura (f) y un conjunto de ejes (ax) utilizando matplotlib. Los ejes se crean con un tamaño especificado de 7x5 pulgadas.

dy="group"; dx="score"; ort="h"; pal = sns.color_palette(n_colors=1): Aquí se definen varias variables que se usarán en la llamada a la función half_violinplot. dy y dx son los nombres de las columnas del dataframe df que se usarán en el eje y y x respectivamente, ort se usa para definir la orientación del gráfico (horizontal en este caso), y pal define una paleta de colores con un solo color.

ax=pt.half_violinplot( x = dx, y = dy, data = df, palette = pal, bw = .2, cut = 0.,scale = "area", width = .6, inner = None, orient = ort): Aquí se está llamando a la función half_violinplot del módulo ptitprince para dibujar un medio gráfico de violín en los ejes que se crearon anteriormente. Los argumentos de esta función son los siguientes:

x y y: especifican los datos que se usarán en los ejes x e y respectivamente.
data: el dataframe que contiene los datos.
palette: la paleta de colores que se usará para el gráfico.
bw: se refiere al ancho de banda del kernel y se usa en la estimación de la densidad del kernel que se utiliza para hacer el gráfico del violín.
cut: número de desviaciones estándar a partir de las cuales se cortará el gráfico del violín.
scale: si es "area", cada violin tendrá el mismo área (es decir, cada violín tiene la misma cantidad de observaciones).
width: ancho del violin.
inner: define qué representa el interior del gráfico del violín. Si es None, no se dibuja nada en el interior.
orient: la orientación del gráfico. "h" para horizontal, "v" para vertical.

Ne cuando al ancho de banda del violinplot o de un density line entraremos en detalle a continuación:

El ancho de banda bw (bandwidth en inglés) es un parámetro esencial en la estimación de densidad del núcleo, un proceso clave para crear un gráfico de violín. Este parámetro controla cuán suave será la curva de densidad en el gráfico.

Elegir un bw adecuado puede ser un poco complicado. No existe un valor universalmente óptimo para bw, ya que lo que podría funcionar bien para un conjunto de datos puede no funcionar tan bien para otro. Aquí hay algunas pautas generales:

Un bw pequeño: Si eliges un bw muy pequeño, la estimación de la densidad del núcleo será muy sensible a las fluctuaciones de los datos, lo que puede llevar a un gráfico de violín con muchos picos y valles. Esto puede ser útil si tus datos tienen muchas características de alta frecuencia que deseas destacar, pero también puede llevar a una representación excesivamente ruidosa de tus datos.

Un bw grande: Si eliges un bw muy grande, la estimación de la densidad del núcleo será muy suave, lo que puede llevar a un gráfico de violín que parece demasiado simplificado. Esto puede ser útil si tus datos tienen una distribución general suave y quieres destacar la forma general, pero puede ocultar características de los datos.

Selección de bw por validación cruzada: Algunas bibliotecas proporcionan métodos para seleccionar automáticamente el bw óptimo utilizando técnicas como la validación cruzada. En este enfoque, se prueba un rango de valores de bw y se selecciona el que proporciona el mejor ajuste a los datos.

En la práctica, una buena forma de comenzar es probar diferentes valores de bw y ver cuál proporciona la representación más útil de tus datos. En general, querrás un equilibrio entre la suavidad y la captura de las características importantes de los datos.

El bw (ancho de banda) en un gráfico de violín, que utiliza un método de estimación de la densidad del núcleo, es un parámetro que controla la suavidad de la curva de densidad que se muestra en el gráfico. Su valor es un número positivo.

En el caso de un gráfico de violín, la curva de densidad es esencialmente un suavizado del histograma de los datos. Cuando calculas un histograma, divides los datos en bins y cuentas cuántos puntos caen en cada bin. Cuando calculas una estimación de la densidad del núcleo, en lugar de contar los puntos en cada bin, colocas una pequeña "protuberancia" o "campana" en cada punto de datos y luego sumas todas estas protuberancias para obtener la curva final.

El ancho de banda bw controla el ancho de estas pequeñas protuberancias. Un bw pequeño producirá protuberancias estrechas, lo que resultará en una curva de densidad más rugosa o picuda, mientras que un bw grande producirá protuberancias más anchas, lo que resultará en una curva de densidad más suave.

Es importante tener en cuenta que, aunque un bw mayor puede hacer que la curva de densidad parezca más "suave", también puede suavizar demasiado los datos y ocultar características importantes. Del mismo modo, un bw demasiado pequeño puede hacer que la curva de densidad sea demasiado rugosa y resalte el ruido en los datos. Por lo tanto, la elección de un buen valor para bw es a menudo un compromiso entre suavidad y fidelidad a los datos.

En el contexto de la estimación de la densidad, el término "kernel" se utiliza para describir la función que se utiliza para "suavizar" los datos. Así, "estimación de densidad del núcleo" y "estimación de densidad del kernel" son dos términos que se usan indistintamente.

Un "kernel" es simplemente una función que asigna un peso a cada punto de datos. Este peso disminuye con la distancia desde el punto en el que se estima la densidad. El ancho de banda ("bandwidth") controla cuánto disminuye este peso con la distancia.

Existen diferentes tipos de funciones de kernel que se pueden usar, incluyendo el kernel Gaussiano (una distribución normal), el kernel Epanechnikov, y muchos otros. Cada uno de estos kernels dará una forma ligeramente diferente a la curva de densidad estimada.

Por lo tanto, sí, en resumen, cuando hablamos de "estimación de densidad del núcleo" estamos hablando de la misma idea que "estimación de densidad del kernel".

En el violin plot, los gráficos de densidad para cada categoría se dibujan alrededor de una línea central. La anchura del "violín" en cualquier punto representa la densidad de los datos. Esto proporciona una representación visual más intuitiva de la densidad de los datos en cada categoría.

Ambos gráficos proporcionan información sobre la distribución de los datos, pero el violin plot puede ser más útil cuando se comparan múltiples categorías de datos al mismo tiempo.

A pesar que los violin plot combinados con dot-plots proveen de mayor información que el boxplot por si solo, existen alternativas aún más versatiles para presentar información de la distribución y de las medidas de tendencia central de la misma. El ejemplo en cuestión es el raincloudplot, el cual es una amalgama robusta entre medio violinplot, un boxplot y los datos crudos con un poco de jitter (agitación o fluctuación)  
...........................................................................................................................................................
-----------------------------------------------------------------------------------------------------------------------------------------------------------


CONSTRUCCIÓN DE RAINCLOUDPLOTS SIGUIENDO EL PAPER Raincloud plots: a multi-platform tool for robust data visualization [version 2; peer review: 2 approved] (https://wellcomeopenresearch.org/articles/4-63/v2)

En este apartado voy a explicar en detalle un resumen del excelente tutorial que esta en el paper antes mencionado

Primero que todo, vamos a presentar los dos approach que difieren realmente en la cantidad de código requerido y per se la escalabilidad del mismo.


Paso 1.) Crear medio violin plot:

Para poder construir medio violon plot se usa una librería que usa recursos de Seaborn, esta libreía se llama  ptitprince y se importa como sigue:

import ptitprince as pt

Una vez con la librería importada definimos un gráfico con subplots como ya es rutinario en matplotlib:

f, ax=plt.subplots(figsize=(7,5)) #recordemos que 7 t 5 son dimensiones horizontal y vertical en pulgadas, respectivamente

Necesitamos que nuestro dataframe contenga variables categorícas, en mi ejemplo la variable categorica es el país de procedencia de la banda:

dy="country"; dx="score"; ort="h";pal="Set1"

dy y dx son las columnas del dataframe, ort solo es una varible para el parametro de orientación de la gráfica (violin plot) y pal el conjunto de colores de una paleta de colores en matplotlib

ax=pt.half_violinplot(x=dx,y=dy,data=dfTest_Norway,palette=pal,bw=.6,cut=0.,scale="area",width= .6,inner=None,orient=ort)

los parámetros bw y cut ya los había explicado antes. Sin embargo, pt es porque estamos usando la librería ptitprince para hacer el medio violinplot

Ahora, para agregar el gráfico de puntos al medio violin plot se usa un gráfico tipo stripplot  
----------------------------------------------------------------------------------------------------
STRIPPLOTS:

Un stripplot en Seaborn es básicamente un gráfico de dispersión donde el eje x representa la variable categórica y el eje y la variable numérica. Se utiliza para visualizar la distribución de los datos a través de diferentes categorías.

El gráfico stripplot coloca cada punto de datos en su categoría a lo largo del eje x y ajusta el eje y según su valor numérico. Como resultado, es posible que los puntos de datos se superpongan, especialmente si tienes muchos puntos de datos.

Aquí tienes un ejemplo básico de cómo se puede utilizar:

import seaborn as sns
import matplotlib.pyplot as plt
tips = sns.load_dataset("tips")
sns.stripplot(x="day", y="total_bill", data=tips)
plt.show()


En este ejemplo, el eje x representa los días de la semana y el eje y representa la cantidad total de la factura. Cada punto en el gráfico representa una factura individual.

En cuanto a zorder, es un parámetro que se utiliza en Matplotlib (y por extensión en Seaborn, que se construye sobre Matplotlib) para determinar el orden en el que se dibujan los elementos en el gráfico. Esto es útil cuando tienes elementos que se superponen y quieres controlar cuál se dibuja encima del otro.

El valor de zorder es un número, y los elementos con un zorder más alto se dibujan encima de los elementos con un zorder más bajo. Por ejemplo, si tienes dos líneas que se cruzan en tu gráfico y quieres asegurarte de que una línea siempre se dibuje encima de la otra, podrías darle a esa línea un valor de zorder más alto.

Por defecto, diferentes tipos de elementos en Matplotlib tienen diferentes valores zorder predeterminados. Por ejemplo, las líneas se dibujan encima de las áreas rellenas porque las líneas tienen un zorder predeterminado más alto que las áreas rellenas.
----------------------------------------------------------------

Para crear el strippplot en nuestra figura de ejemplo hacemos lo siguiente:

ax=sns.stripplot( x = dx, y = dy, data = dfTest_Norway, palette = pal,
 edgecolor = "white", size = 3, jitter = 1, zorder = 0,
 orient = ort)

el parámetro jitter es para agregar fluctuación a los datos y que si están todos concentrados en una región del gráfico, se dispersen aleatoriamente y sean distinguibles. Por otro lado, el parámetro zorder es para definir que imagen se superpone sobre que otra. Finalmente creamos el boxplot de la siguiente forma:

ax=sns.boxplot( x = dx, y = dy, data = dfTest_Norway, color = "black", 
 width = .15, zorder = 10, showcaps = True,
 boxprops = {'facecolor':'none', "zorder":10}, showfliers=True,
 whiskerprops = {'linewidth':2, "zorder":10},
 saturation = 1, orient = ort)

boxprops: Es un diccionario que define las propiedades de la caja en la caja de un boxplot. En este caso, la caja es transparente ('facecolor':'none') y tiene un nivel de zorder de 10, lo que determina la prioridad de dibujo en el gráfico. Los objetos con mayor zorder se dibujan encima de los que tienen menor zorder.

whiskerprops: Similar a boxprops, es un diccionario que define las propiedades de los bigotes en la caja de un boxplot. Aquí, los bigotes tienen un grosor de línea de 2 ('linewidth':2) y un nivel de zorder de 10, que de nuevo determina la prioridad de dibujo en el gráfico.

showfliers: Es un booleano para decidir si se muestran los puntos atípicos o no. En este caso, showfliers=True, por lo que se mostrarán los puntos atípicos.

Estos parámetros se utilizan para personalizar la apariencia del boxplot en el gráfico.


El código final para este raincloud plot es:


f, ax=plt.subplots(figsize=(7,5))
dy="country"; dx="score"; ort="h";pal="Set1"

ax=pt.half_violinplot(x=dx,y=dy,data=dfTest_Norway,palette=pal,bw=.6,cut=0.,scale="area",width= .6,inner=None,orient=ort)

ax=sns.stripplot( x = dx, y = dy, data = dfTest_Norway, palette = pal,
 edgecolor = "white", size = 3, jitter = 1, zorder = 0,
 orient = ort)
ax=sns.boxplot( x = dx, y = dy, data = dfTest_Norway, color = "black", 
 width = .15, zorder = 10, showcaps = True,
 boxprops = {'facecolor':'none', "zorder":10}, showfliers=True,
 whiskerprops = {'linewidth':2, "zorder":10},
 saturation = 1, orient = ort)

-----------------------------------------------------------------------------------------------------------------------------------------------------------

Extracción y limpieza de los datos de la TRM historica extraídos de la API del gobierno nacional:

Regex para la cadena de las fechas:

\d: Este símbolo coincide con cualquier dígito (0-9).

{4}: Este cuantificador indica que el símbolo o grupo anterior debe aparecer exactamente 4 veces. Entonces, \d{4} coincide con cualquier grupo de 4 dígitos, como "2023".

-: Este es un carácter literal que coincide con el símbolo "-" en la cadena. Lo utilizamos para mover nuestra coincidencia más allá del año y hacia el mes y el día.

\d{2}: Al igual que antes, esto coincide con un grupo de 2 dígitos, como "09" o "15". Usamos esto para el mes y el día.

T: Este es un carácter literal que coincide con el "T" en la cadena. Utilizamos este carácter para indicar el final del patrón que queremos capturar para la fecha.

(): Los paréntesis se utilizan para capturar un grupo. Esto nos permite extraer sólo la parte de la coincidencia que nos interesa. En este caso, queremos extraer el año, por lo que lo rodeamos con paréntesis: (\d{4}).

Entonces, al juntar todo:

(\d{4}) captura los 4 dígitos que representan el año.
-\d{2}-\d{2} coincide con el mes y el día, pero no los captura.
T coincide con el carácter "T" que sigue al día en la cadena.
La función match.group(1) en el código extrae el primer (y único, en este caso) grupo capturado, que es el año.