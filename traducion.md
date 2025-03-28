**Web Scraping con Beautiful Soup**  
Íconos usados en este cuaderno

🔔 **Pregunta**: Una pregunta rápida para ayudarte a entender lo que está pasando.  
🥊 **Desafío**: Ejercicio interactivo. ¡Trabajaremos en ellos durante el taller!  
⚠️ **Advertencia**: Aviso sobre aspectos complicados o errores comunes.  
💡 **Consejo**: Cómo hacer algo de manera más eficiente o efectiva.  
🎬 **Demostración**: Presentando algo más avanzado, para que veas para qué se puede usar Python.

### **Objetivos de aprendizaje**

1. **Reflexión**: ¿Hacer scraping o no hacerlo?
2. **Extracción y análisis de HTML**
3. **Scraping de la Asamblea General de Illinois**

### **Hacer Scraping o No Hacer Scraping**

Cuando queremos acceder a datos desde la web, primero debemos verificar si el sitio web de nuestro interés ofrece una API web. Plataformas como Twitter, Reddit y el New York Times ofrecen APIs. Si quieres aprender a usarlas, consulta el taller de APIs web en Python de D-Lab.

Sin embargo, a veces no existe una API web disponible. En estos casos, podemos recurrir al **web scraping**, donde extraemos el HTML subyacente de una página web para obtener directamente la información que necesitamos. Existen varios paquetes en Python que nos ayudan a realizar estas tareas. Nos enfocaremos en dos paquetes: **Requests** y **Beautiful Soup**.

Nuestro caso de estudio será el scraping de información sobre los **senadores estatales de Illinois**, así como la lista de proyectos de ley que ha patrocinado cada senador. Antes de comenzar, explora estos sitios web para familiarizarte con su estructura.



### **Instalación**

Utilizaremos dos paquetes principales: **Requests** y **Beautiful Soup**. Si aún no los tienes instalados, puedes hacerlo con el siguiente comando:
```
%pip install requests
%pip install beautifulsoup4
```
También instalaremos el paquete **lxml**, que ayuda a mejorar el análisis de datos que realiza Beautiful Soup. Puedes instalarlo con el siguiente comando:
```
%pip install lxml
```
```
#Import required libraries
from bs4 import BeautifulSoup
from datetime import datetime
import requests
import time
```

### **Extracción y Análisis de HTML**

Para hacer **web scraping** y analizar HTML con éxito, seguiremos estos **cuatro pasos**:

1.  **Hacer una solicitud GET** 
2.  **Analizar la página con Beautiful Soup**   
3.  **Buscar elementos HTML** 
4.  **Obtener los atributos y el texto de estos elementos** 
    


### **Paso 1: Hacer una Solicitud GET para Obtener el HTML de una Página**

Podemos usar la biblioteca **Requests** para:

1.  **Hacer una solicitud GET** a la página web.    
2.  **Leer el código HTML** de la página.
    
El proceso de hacer una solicitud y obtener un resultado es similar al flujo de trabajo de una API web. Sin embargo, en este caso, estamos haciendo la solicitud **directamente al sitio web**, y **tendremos que analizar el HTML nosotros mismos**. Esto es diferente de obtener datos organizados en un formato más estructurado, como **JSON** o **XML**.
```
#Make a GET request
req = requests.get('http://www.ilga.gov/senate/default.asp')
#Read the content of the server’s response
src = req.text
#View some output
print(src[:1000])
```
### **Paso 2: Analizar la Página con Beautiful Soup**

Ahora usaremos la función **BeautifulSoup** para **convertir la respuesta en una estructura de árbol HTML**. Esto nos devuelve un objeto llamado **soup object**, que contiene todo el HTML del documento original.

Si te encuentras con un error relacionado con una biblioteca de análisis (**parser**), asegúrate de que tienes instalado el paquete **lxml**, ya que proporciona a Beautiful Soup las herramientas necesarias para procesar el HTML.
```
#Parse the response into an HTML tree
soup = BeautifulSoup(src, 'lxml')
#Take a look
print(soup.prettify()[:1000])
```
La salida se ve bastante similar a la anterior, pero ahora está organizada en un **objeto soup**, lo que nos permite recorrer la página de manera más sencilla y estructurada

### **Paso 3: Buscar Elementos HTML**

Beautiful Soup cuenta con varias funciones para encontrar componentes útiles en una página. Beautiful Soup permite encontrar elementos por: 

-   **Etiquetas HTML** 
-   **Atributos HTML** 
-   **Selectores CSS**
    
#### **Buscar por Etiquetas HTML**

La función **`find_all`** busca en el árbol de **soup** y devuelve **todos los elementos** que coincidan con una etiqueta HTML específica.

¿Qué hace el siguiente ejemplo?
```
#Find all elements with a certain tag
a_tags = soup.find_all("a")
print(a_tags[:10])
```
Porque `find_all()` es el método más popular en la API de búsqueda de Beautiful Soup, puedes usar un atajo para ello. Si tratas el objeto BeautifulSoup como si fuera una función, entonces es lo mismo que llamar a `find_all()` en ese objeto.

Estas dos líneas de código son equivalentes:
```
a_tags = soup.find_all("a")
a_tags_alt = soup("a")
print(a_tags[0])
print(a_tags_alt[0])
```
¿Cuántos enlaces obtuvimos?
```
print(len(a_tags))
```
¡Eso es mucho! Muchos elementos en una página tendrán la misma etiqueta HTML. Por ejemplo, si buscas todo lo que tenga la etiqueta a, es probable que obtengas más resultados, muchos de los cuales quizás no desees. Recuerda que la etiqueta a define un hipervínculo, por lo que generalmente encontrarás muchos en cualquier página dada.

¿Qué pasaría si quisiéramos buscar etiquetas HTML con ciertos atributos, como clases CSS particulares?

Podemos hacer esto añadiendo un argumento adicional a `find_all`. En el ejemplo a continuación, estamos encontrando todas las etiquetas a y luego filtrando aquellas con `class_="sidemenu"`.
```
#Get only the 'a' tags in 'sidemenu' class
side_menus = soup("a", class_="sidemenu")
side_menus[:5]
```
Una forma más eficiente de buscar elementos en un sitio web es a través de un selector CSS. Para esto, tenemos que usar un método diferente llamado `select()`. Simplemente pasa una cadena al `.select()` para obtener todos los elementos que tengan esa cadena como un selector CSS válido.

En el ejemplo anterior, podemos usar "a.sidemenu" como un selector CSS, que devuelve todas las etiquetas a con la clase sidemenu.
```
#Get elements with "a.sidemenu" CSS Selector.
selected = soup.select("a.sidemenu")
selected[:5]
```


### **Paso 4: Obtener Atributos y Texto de los Elementos**

Una vez que identificamos los elementos, queremos acceder a la información dentro de ellos. Generalmente, esto significa dos cosas:

-   **Texto** 
-   **Atributos** 

Obtener el texto dentro de un elemento es fácil. Solo necesitamos usar la propiedad **`.text`** de un objeto de etiqueta:
```
#Get all sidemenu links as a list
side_menu_links = soup.select("a.sidemenu")

#Examine the first link
first_link = side_menu_links[0]
print(first_link)

#What class is this variable?
print('Class: ', type(first_link))
```
¡Es una etiqueta de Beautiful Soup! Esto significa que tiene un miembro de texto:
```
print(first_link.text)
```
A veces queremos el valor de ciertos atributos. Esto es particularmente relevante para las etiquetas a, o enlaces, donde el atributo href nos indica a dónde lleva el enlace.

💡 Consejo: Puedes acceder a los atributos de una etiqueta tratándola como un diccionario:
```
print(first_link['href'])
```

## **Scraping la Asamblea General de Illinois**

Créalo o no, esas son realmente las herramientas fundamentales que necesitas para raspar un sitio web. Una vez que dediques más tiempo a familiarizarte con HTML y CSS, simplemente se trata de entender la estructura de un sitio web en particular y aplicar inteligentemente las herramientas de Beautiful Soup y Python.

Apliquemos estas habilidades para raspar la 98ª Asamblea General de Illinois.

Específicamente, nuestro objetivo es raspar información sobre cada senador, incluyendo su nombre, distrito y partido.

### **Scrapear y Analizar la Página Web**

Ahora vamos a **scrapear** y **parsear** la página web utilizando las herramientas que aprendimos en la sección anterior. 
```
#Make a GET request
req = requests.get('http://www.ilga.gov/senate/default.asp?GA=98')
#Read the content of the server’s response
src = req.text
#Soup it
soup = BeautifulSoup(src, "lxml")
```
### **Buscar los elementos de la tabla**

Nuestro objetivo es obtener los elementos de la tabla en la página web. Recuerda: las filas se identifican por la etiqueta **tr**. Vamos a usar **find_all** para obtener estos elementos.
```
#Get all table row elements
rows = soup.find_all("tr")
len(rows)
```
⚠️ **Advertencia:** Ten en cuenta que **`find_all`** obtiene **todos** los elementos con la etiqueta `<tr>`. Sin embargo, solo queremos algunos de ellos.

Si usamos la función **"Inspeccionar"** en Google Chrome y observamos con atención, podemos utilizar **selectores CSS** para obtener solo las filas que nos interesan.

Específicamente, queremos **las filas internas** de la tabla.
```
#Returns every ‘tr tr tr’ css selector in the page
rows = soup.select('tr tr tr')

for row in rows[:5]:
    print(row, '\n')
```
Parece que necesitamos todo después de las **primeras dos filas**.

Comencemos trabajando con una sola fila y, a partir de ahí, construiremos nuestro bucle. 
```
example_row = rows[2]
print(example_row.prettify())
```
Desglosaremos esta fila en sus **celdas/columnas** usando el método **`select`** con **selectores CSS**.

Si observamos detenidamente el HTML, hay varias formas de hacerlo:

1.  Podemos identificar las celdas por su etiqueta **`td`**.    
2.  Podemos usar el nombre de la clase **`.detail`**.    
3.  Podemos combinar ambos y usar el selector **`td.detail`**. 


```
for cell in example_row.select('td'):
    print(cell)
print()

for cell in example_row.select('.detail'):
    print(cell)
print()

for cell in example_row.select('td.detail'):
    print(cell)
print()
```
Podemos confirmar que todas estas opciones producen el mismo resultado. 
```
assert example_row.select('td') == example_row.select('.detail') == example_row.select('td.detail')
```
Utilicemos el selector **`td.detail`** para ser lo más específicos posible. 
```
#Select only those 'td' tags with class 'detail' 
detail_cells = example_row.select('td.detail')
detail_cells
```
La mayor parte del tiempo, estamos interesados en el texto real de un sitio web, no en sus etiquetas. Recuerda que para obtener el texto de un elemento HTML, usamos el atributo text:
```
#Keep only the text in each of those cells
row_data = [cell.text for cell in detail_cells]

print(row_data)
```
¡Se ve bien! Ahora solo necesitamos usar nuestros conocimientos básicos de Python para obtener los elementos de esta lista que queremos. Recuerda, queremos el nombre del senador, su distrito y su partido.
```
print(row_data[0]) # Name
print(row_data[3]) # District
print(row_data[4]) # Party
```
### **Eliminar Filas No Deseadas**

Vimos al principio que no todas las filas que obtuvimos corresponden realmente a un senador. Necesitaremos hacer algo de limpieza antes de poder continuar. Echa un vistazo a algunos ejemplos:
```
print('Row 0:\n', rows[0], '\n')
print('Row 1:\n', rows[1], '\n')
print('Last Row:\n', rows[-1])
```
Cuando escribamos nuestro bucle **_for_**, queremos que solo se aplique a las filas relevantes. Por lo tanto, necesitaremos filtrar las filas irrelevantes. La forma de hacerlo es compararlas con las filas que sí queremos, ver en qué se diferencian y luego formular una condición basada en esas diferencias.

Como puedes imaginar, hay muchas formas posibles de hacerlo, y dependerá del sitio web. Mostraremos algunas aquí para darte una idea de cómo hacerlo.
```
#Bad rows
print(len(rows[0]))
print(len(rows[1]))

#Good rows
print(len(rows[2]))
print(len(rows[3]))
```
Quizás las filas correctas tengan una longitud de 5. Vamos a comprobarlo:
```
good_rows = [row for row in rows if len(row) == 5]
#Let's check some rows
print(good_rows[0], '\n')
print(good_rows[-2], '\n')
print(good_rows[-1])
```
Encontramos una fila de pie de página en nuestra lista que queremos evitar. Probemos otra cosa:
```
rows[2].select('td.detail') 
```
```
#Bad row
print(rows[-1].select('td.detail'), '\n')

#Good row
print(rows[5].select('td.detail'), '\n')

#How about this?
good_rows = [row for row in rows if row.select('td.detail')]

print("Checking rows...\n")
print(good_rows[0], '\n')
print(good_rows[-1])
```
¡Parece que encontramos algo que funcionó!

### **Recorriéndolo Todo**

Ahora que hemos visto cómo obtener los datos que queremos de una fila y cómo filtrar las filas que no necesitamos, juntémoslo todo en un bucle.
```
#Define storage list
members = []

#Get rid of junk rows
valid_rows = [row for row in rows if row.select('td.detail')]

#Loop through all rows
for row in valid_rows:
    # Select only those 'td' tags with class 'detail'
    detail_cells = row.select('td.detail')
    # Keep only the text in each of those cells
    row_data = [cell.text for cell in detail_cells]
    # Collect information
    name = row_data[0]
    district = int(row_data[3])
    party = row_data[4]
    # Store in a tuple
    senator = (name, district, party)
    # Append to list
    members.append(senator)
 ```
```
#Should be 61
len(members)
```
Echemos un vistazo a lo que tenemos en **_members_**.
```
print(members[:5])
```
