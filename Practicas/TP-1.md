# TP-1: JSON-LD y Schema.org

## Objetivo didáctico 
Reconocer el valor de contar con datos estructurados en base a estándares. 

## Requerimientos 

Analizar información obtenida de tres fuentes diferentes descriptas en archivos con formato JSON-LD. No necesitará hacer uso de ningún lenguaje o herramienta de programación.  Este formato ya se encuentra estructurado y utiliza como vocabulario a schema.org

## Actividad 

En este TP nos enfocaremos, nuevamente, solo en el aspecto de recuperar y combinar la información existente y almacenarla localmente para dejarla disponible para su futura reutilización en una eventual aplicación a desarrollar. 

Nos vamos a concentrar en combinar información de sitios que sabemos utilizan vocabularios de Schema.org y los integran en sus páginas via JSON-LD. 

En particular, vamos a integrar la información de películas de 3 sitios web en los que encontramos URLs como estas para la película Wonder Woman 1984: 

- https://www.rottentomatoes.com/m/wonder_woman_1984 
- https://www.imdb.com/title/tt7126948/ 
- https://www.ecartelera.com/peliculas/wonder-woman-1984 

El objetivo es obtener toda la información que sea posible de cada película para integrarla en un solo archivo o base de datos (sobre el que se podrían hacer aplicaciones a futuro). Note que ahora tenemos información sobre la película y sobre opiniones de usuarios. 

La tarea principal es tratar de unificar la información de las tres fuentes en única base de conocimiento. En su caso un nuevo archivo unificando información. En esta unificación va a tener que tomar decisiones. Anote todas las decisiones que tomó.  

Va a tener que investigar un poco de aquellos terminos que puede no llegar a conocer. La tarea no implica programar una solución sino describir en palabras precisas y detalladas los procesos para que se concrete en forma automatica la integración de datos.

## Tareas 

1. Seleccione solamente 3 propiedades de la pelicula que les parezca interesante definir una estrategia de integracion, por ejemplo porque los valores no coinciden en las diferentes fuentes, o son complementarios, o parecen contradictorios, o utilizan diferentes escalas. Evite aquellos atributos que poseen iguales valores.

2. Por cada una de ellas, detalle el procedimiento paso a paso. Para esto puede utilizar un diagrama y/o pseudocódigo, indicando en cada paso los detalles del mismo.

3. Se va a encontrar con situaciones excepcionales (el mismo atributo en dos de las fuentes con valores diferentes). Piense cuáles son esas situaciones, descríbalas y proponga como tratarlas.

4. Si tuviese que programar un nuevo scrapper para obtener información de un sitio que no describe sus datos con json ld. Que desafíos encontraría y como integraría esta solucion a la base de conocimiento anterior. 

## Resolución tareas

### Ejercicio 1
Selecciono las siguientes tres propiedades:

#### Calificación de la obra (`aggregateRating`):
**Estado de los datos**: eCartelera reporta un valor de "7,6" sobre una escala de 10, mientras que en Rotten Tomatoes reporta un valor de "59" sobre una escala de 100.
**Justificación**: Lo selecciono por presentar un desafío de diferentes escalas. No se pueden comparar o promediar directamente sin una transformación previa que los normalice a un rango común.

#### Elenco de actores (`actor / actors`):
**Estado de los datos**: Rotten Tomatoes incluye metadatos adicionales como una URL de imagen y enlaces sameAs a perfiles de celebridades. Por otro lado, eCartelera ofrece enlaces regionales a biografías en español mediante mainEntityOfPage.
**Justificación**: La selecciono por ser información complementaria. La integración permite construir un grafo de conocimiento más rico que el de cualquier fuente por separado, uniendo nodos bajo un identificador común.

#### Fecha de lanzamiento / Evento de publicación (`datePublished / releasedEvent`):
**Estado de los datos**: eCartelera especifica el "2020-12-18" para la localización "ES" (España). Las otras fuentes suelen registrar la fecha de estreno en EE. UU. o México, que suelen ser distintas.
**Justificación**: La selecciono porque los valores no coinciden debido al contexto (regionalidad). Requeriria una decisión de diseño: ¿guardamos todas las fechas por país o solo la primera a nivel mundial?.

### Ejercicio 2
Voy a seguir un formato de pseudocodigo como el que se vio la Clase-1 con el algoritmo de los premiados.

#### Algoritmo para Calificación Normalizada (`aggregateRating`)
 
```
total-rating := 0
source-count := 0
for all  movie-source  in  {RottenTomatoes, IMDb, eCartelera}
    value := get  ratingValue  from  movie-source:aggregateRating
    best := get  bestRating  from  movie-source:aggregateRating
     
    // Normalización a escala 0-100
    if  best == "10"
        norm-value := value * 10
    else
        norm-value := value
    end
     
    total-rating := total-rating + norm-value
    source-count := source-count + 1
end
final-average := total-rating / source-count
return  final-average
```

#### Algoritmo para Elenco Enriquecido (`actor`)

```
unified-cast := ∅
for all  movie-source  in  {RottenTomatoes, IMDb, eCartelera}
    actor-list := get  actor  from  movie-source
    for all  actor  in  actor-list
        name := get  name  of  actor
        if  name  not in  unified-cast
            create new node for  name
        end
        
        // Paso de complementariedad: agregar metadatos si existen
        if  actor  has  image  then  add  image  to  node(name)
        if  actor  has  sameAs  then  add  link  to  node(name)
        if  actor  has  mainEntityOfPage  then  add  link  to  node(name)
        
        add  node(name)  to  unified-cast
    end
end
return  unified-cast
```
#### Algoritmo para Estreno Mundial (`releasedEvent`)

```
earliest-date := “9999-12-31”
for all  movie-source  in  {RottenTomatoes, IMDb, eCartelera}
    event := get  releasedEvent  from  movie-source
    current-date := get  startDate  from  event
    
    // Decisión de diseño: conservar la fecha más antigua (World Premiere)
    if  current-date  <  earliest-date
        earliest-date := current-date
    end
end
add  movie,  schema:datePublished,  earliest-date  to  results
return  results
```

#### Decisiones de diseño tomadas:
**Normalización**: Se decidió llevar todos los ratings a una escala de 0 a 100 para evitar que la fuente con escala menor (eCartelera) pierda peso visual frente a las demás.
**Unicidad de actores**: Se asume que el nombre del actor es un identificador suficiente para la unión de nodos, aunque cabe destacar que en el libro de Aidan Hogan se advierte que en la Web real se necesitan URIs únicas para evitar ambigüedad.
**Prioridad de fecha**: Se optó por la fecha cronológica más antigua para definir el "estreno oficial", ignorando la procedencia regional en favor de la precedencia temporal.

### Ejercicio 3

Las situaciones que detecte son las siguientes:

#### Conflicto de identidad
**Situación**: Una fuente menciona a un actor con un error de escritura. Un ejemplo de esto es que el JSON-LD de eCartelera escribe "Kristen Wigg", mientras que Rotten Tomatoes escribe "Kristen Wiig". Una máquina tomaria esto como dos personas distintas.

**Tratamiento**: Una solución que creeria óptima es no confiar en las cadenas de texto y buscar un identificador unico (URI) global en una base como la que vimos en la teoria, Wikidata. Si ambas fuentes no lo proveen, usaria algo como lo que se propone en el libro de Aidan Hogan 

#### Conflicto de Cardinalidad en Valores "Únicos"
**Situación**: Una propiedad que lógicamente debería tener un solo valor, como la duración, presenta variaciones. eCartelera indica "PT2H31M" (151 min), pero otra fuente podría redondearlo o incluir créditos finales, arrojando 155 min.
**Tratamiento**: Ante la falta de una "verdad absoluta", aplicaria la Proveniencia (Provenance). En lugar de borrar un dato, la base de conocimiento debe almacenar el valor junto con su origen: `(Pelicula, schema:duration, "151 min", fuente:eCartelera)`. Esto permitiria que la aplicación final decida si mostrar un rango o priorizar la fuente más confiable.

#### Contradicción lógica por "desactualización"

**Situación**: Un valor puede cambiar con el tiempo, como bien podría ser el rating que le dan a la pelicula de Wonder Woman. eCartelera por ejemplo reporta un 7.6 basado en 20 votos, mientras que Rotten Tomatoes muestra las reseñas individuales más recientes que podrían alterar el promedio. 

**Tratamiento**: Se puede definir una regla que invalide promedios que no hayan sido actualizados en un periodo X, o priorizar la fuente que posea un `ratingCount` significativamente mayor para garantizar representatividad.

### Ejercicio 4

En este caso al no usar JSON-LD, pasariamos de una "Habitación China Simbólica" a una que se basaría en lenguaje natural, lo cual traeria desafios tales como:

- **Brecha semántica**: Los documentos HTML están diseñados para que los humanos lo lean, pero no para que las maquinas los entiendan. La información esta encapsulada en "texto plano".
- **Ambigüedad**: En el texto libre, un nombre puede referirse a múltiples cosas. Por ejemplo, como vimos en la teoría, Dublin puede ser una la capital de Irlanda o bien un pub.
- **Heterogeneidad estructural**: Hay muchas formas de decir lo mismo. Un ejemplo de esto sería como se especifica la duración de una pelicula por las diferentes fuentes (151 min vs 2h 31m). Sin las etiquetas de Schema.org, la máquina no sabría que fragmento de texto corresponde a que propiedad.

