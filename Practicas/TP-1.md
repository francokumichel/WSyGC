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