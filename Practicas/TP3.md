# Inmonotology - Ontología del dominio inmobiliario para el OVS

## Actividades

### Documentación

Para cada pregunta se indica el código SPARQL completo evaluado sobre el grafo de conocimiento provisto, junto con las dificultades encontradas durante su elaboración.

#### Pregunta 1: ¿Cuáles son los precios de todos los inmuebles?

```sparql
PREFIX inmo: <http://www.semanticweb.org/luciana/ontologies/2024/8/inmontology#>
PREFIX gr: <http://purl.org/goodrelations/v1#>
PREFIX pronto: <https://raw.githubusercontent.com/fdioguardi/pronto/main/ontology/pronto.owl#>

SELECT ?listing ?price
WHERE {
  ?listing a pronto:RealEstateListing ;
           inmo:hasFeature ?f .
  ?f a inmo:Price ;
     inmo:hasValue ?v .
  ?v gr:hasCurrencyValue ?price .
}
```

**Dificultades:** ninguna relevante para este caso. El patrón de navegación `Listing → hasFeature → Price → hasValue → UnitPriceSpecification → hasCurrencyValue` se corresponde directamente con la estructura documentada en la consigna.

#### Pregunta 2: ¿Cuáles son los precios de todos los inmuebles obtenidos por el scrapper?

```sparql
PREFIX inmo: <http://www.semanticweb.org/luciana/ontologies/2024/8/inmontology#>
PREFIX gr: <http://purl.org/goodrelations/v1#>
PREFIX pronto: <https://raw.githubusercontent.com/fdioguardi/pronto/main/ontology/pronto.owl#>

SELECT ?listing ?price
WHERE {
  ?listing a pronto:RealEstateListing ;
           inmo:hasFeature ?f .
  ?f a inmo:Price ;
     inmo:hasOrigin inmo:Scraper ;
     inmo:hasValue ?v .
  ?v gr:hasCurrencyValue ?price .
}
```

**Dificultades:** en un primer momento intenté filtrar el origen mediante una propiedad `hasScrapperValue`, siguiendo el ejemplo de modelado descripto en el enunciado. Sin embargo, al inspeccionar el `grafo_la_plata.ttl` real pude comprobar que dicha propiedad no existe en los datos, sino que el origen del valor se indica con la propiedad `inmo:hasOrigin` apuntando al individuo `inmo:Scraper`, mientras que `inmo:hasValue` se mantiene como propiedad genérica para acceder al valor.

Algo que me resulto raro es que el resultado de esta consulta coincidia exactamente con el de la Pregunta 1. Confirmé que no es un error de la consulta sino un reflejo de los datos: al agrupar los `Price` por su `hasOrigin` obtuve una única categoría (`Scraper`), es decir que el grafo provisto todavía no contiene valores de precio con origen de curación manual (`hasCurationValue`) ni de extracción NLP (`hasAVEValue`).

#### Pregunta 3: ¿Cuáles son las ciudades y sus nombres de las que hay información de inmuebles?

```sparql
PREFIX inmo: <http://www.semanticweb.org/luciana/ontologies/2024/8/inmontology#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?cityName
WHERE {
  ?re inmo:hasFeature ?f .
  ?f a inmo:Address ;
     inmo:hasValue ?addr .
  ?addr a inmo:PostalAddress ;
        inmo:city ?city .
  ?city rdfs:label ?cityName .
}
```

**Dificultades:** el resultado de esta consulta da una única ciudad ("La Plata"), lo cual inicialmente me generó la duda de si el grafo debía incluir también localidades como City Bell o Manuel B. Gonnet. Pude confirmar que esto es correcto y esperado: dichas localidades están modeladas como valores de la propiedad `inmo:neighborhood` (barrio/localidad), no de `inmo:city`. La propiedad `city` en este grafo corresponde al partido/ciudad de nivel superior (en este caso, La Plata), mientras que `neighborhood` captura la granularidad más fina. Verifiqué ejecutando una consulta análoga sobre `inmo:neighborhood`, la cual sí devuelve barrios como "City Bell" y "Manuel B. Gonnet".

#### Pregunta 4: ¿Cuáles son los precios de los inmuebles ubicados en La Plata (General Pueyrredon)?

```sparql
PREFIX inmo: <http://www.semanticweb.org/luciana/ontologies/2024/8/inmontology#>
PREFIX gr: <http://purl.org/goodrelations/v1#>
PREFIX pronto: <https://raw.githubusercontent.com/fdioguardi/pronto/main/ontology/pronto.owl#>
PREFIX sioc: <http://rdfs.org/sioc/ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?listing ?price
WHERE {
  ?listing a pronto:RealEstateListing ;
           sioc:about ?re ;
           inmo:hasFeature ?pf .
  ?pf a inmo:Price ;
      inmo:hasValue ?pv .
  ?pv gr:hasCurrencyValue ?price .

  ?re inmo:hasFeature ?af .
  ?af a inmo:Address ;
      inmo:hasValue ?av .
  ?av a inmo:PostalAddress ;
      inmo:city ?city .
  ?city rdfs:label ?cityName .
  FILTER(regex(?cityName, "La Plata", "i"))
}
```

**Dificultades:** nada adicional a las que ya describí en la Pregunta 3 respecto al alcance de `city`. Aclaro que el grafo al ser solo para La Plata, ajuste el filtro para esta ciudad.

#### Pregunta 5: ¿Cuáles son los precios del mes de febrero de los inmuebles ubicados en La Plata?

```sparql
PREFIX inmo: <http://www.semanticweb.org/luciana/ontologies/2024/8/inmontology#>
PREFIX gr: <http://purl.org/goodrelations/v1#>
PREFIX pronto: <https://raw.githubusercontent.com/fdioguardi/pronto/main/ontology/pronto.owl#>
PREFIX sioc: <http://rdfs.org/sioc/ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX time: <http://www.w3.org/2006/time#>

SELECT ?listing ?price
WHERE {
  ?listing sioc:about ?re ;
           inmo:hasFeature ?pf .
  ?pf a inmo:Price ;
      inmo:hasValue ?pv ;
      time:hasTime ?t .
  ?pv gr:hasCurrencyValue ?price .
  ?t time:inXSDDateTimeStamp ?date .
  FILTER(MONTH(?date) = 2)

  ?re inmo:hasFeature ?af .
  ?af a inmo:Address ;
      inmo:hasValue ?av .
  ?av a inmo:PostalAddress ;
      inmo:city ?city .
  ?city rdfs:label ?cityName .
  FILTER(regex(?cityName, "La Plata", "i"))
}
```

**Dificultades:** lo que más me complicó de esta consulta fue distinguir entre dos nociones de fecha presentes en el grafo:

- `dc:date`, propiedad sobre el `RealEstateListing`, que indica la fecha de **publicación** del aviso.
- `time:hasTime` sobre cada `Feature` (en este caso, `Price`), que indica la fecha en la que el **scraper capturó** ese valor puntual.

Al filtrar por mes usando `time:hasTime` sobre la Feature `Price` obtuve resultados únicamente para febrero y marzo, mientras que `dc:date` sobre los listings presenta datos distribuidos en los doce meses del año. Dado estó, interpreté que la pregunta refiere al momento en que se registró el valor del precio (y no a la fecha de publicación del aviso), por lo que opte por filtrar sobre `time:hasTime` de la Feature `Price`. Luego valide esto ejecutando consultas de diagnóstico que confirmaron que el rango temporal cubierto por el proceso de scraping de características (`time:hasTime`) es acotado a febrero-marzo, a diferencia de la fecha de publicación de los avisos.

#### Pregunta 6: ¿Cuál es el stock de oferta durante el mes de febrero en todo el grafo?

```sparql
PREFIX pronto: <https://raw.githubusercontent.com/fdioguardi/pronto/main/ontology/pronto.owl#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>

SELECT (COUNT(DISTINCT ?listing) AS ?stockFebrero)
WHERE {
  ?listing a pronto:RealEstateListing ;
           dc:date ?date .
  FILTER(MONTH(?date) = 2)
}
```

**Dificultades:** la consigna advierte que en el grafo existen avisos que son co-referencia entre sí, vinculados mediante `owl:sameAs`, lo cual debe considerarse al contar el stock para no duplicar inmuebles. Primero lo que hice fue utilizar únicamente `COUNT(DISTINCT ?listing)`, asumiendo que esto resolvía el problema de duplicados.

Sin embargo note que `DISTINCT` solo elimina filas sintácticamente idénticas, y **no fusiona** dos URIs distintas relacionadas por `owl:sameAs` (que, aunque representen el mismo inmueble en el mundo real, son valores diferentes desde el punto de vista de SPARQL). Por lo tanto, `DISTINCT` por sí solo no resuelve, en términos generales, la deduplicación por co-referencia.

Para determinar si esto representaba un problema real en los datos, ejecute la siguiente consulta de verificación:

```sparql
PREFIX owl: <http://www.w3.org/2002/07/owl#>

SELECT (COUNT(*) AS ?cantidad)
WHERE {
  ?a owl:sameAs ?b .
}
```

El resultado fue `0`, pudiendo confirmar que el `grafo_la_plata.ttl` provisto **no tiene tripletas `owl:sameAs`**. Por lo tanto, para este grafo en particular, `COUNT(DISTINCT ?listing)` es suficiente y no introduce duplicados por co-referencia.

Además se utilizó `dc:date` (fecha de publicación) en lugar de `time:hasTime` de alguna Feature particular, ya que la pregunta refiere al "stock de oferta" del grafo en general y no a una característica puntual, por lo que la fecha de publicación del aviso es la interpretación más adecuada para esta consulta en particular (a diferencia de las Preguntas 5 y 7).

#### Pregunta 7: ¿Cuál es el promedio de precios de inmuebles ubicados en La Plata durante el mes de febrero?

```sparql
PREFIX inmo: <http://www.semanticweb.org/luciana/ontologies/2024/8/inmontology#>
PREFIX gr: <http://purl.org/goodrelations/v1#>
PREFIX pronto: <https://raw.githubusercontent.com/fdioguardi/pronto/main/ontology/pronto.owl#>
PREFIX sioc: <http://rdfs.org/sioc/ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX time: <http://www.w3.org/2006/time#>

SELECT (AVG(?price) AS ?promedio)
WHERE {
  ?listing sioc:about ?re ;
           inmo:hasFeature ?pf .
  ?pf a inmo:Price ;
      inmo:hasValue ?pv ;
      time:hasTime/time:inXSDDateTimeStamp ?date .
  ?pv gr:hasCurrencyValue ?price .
  FILTER(MONTH(?date) = 2)

  ?re inmo:hasFeature ?af .
  ?af a inmo:Address ;
      inmo:hasValue ?av .
  ?av a inmo:PostalAddress ;
      inmo:city/rdfs:label ?cityName .
  FILTER(regex(?cityName, "La Plata", "i"))
}
```

**Dificultades:** similar a lo dicho en la Pregunta 5 respecto a la elección de `time:hasTime` sobre la Feature `Price` como criterio temporal. Adicionalmente, se debe tener en cuenta que todos los precios provienen de la moneda USD, por lo que no fue necesario aplicar conversión de divisas para que el promedio sea comparable; de existir precios en otras monedas, sería necesario filtrar por `gr:hasCurrency` o normalizar los valores antes de promediar.

### Nuevas preguntas

Propongo 10 preguntas adicionales para realizar sobre el grafo de conocimiento, junto con una breve explicación de qué se espera obtener con cada una.

**1) ¿Cuántos avisos hay por función de negocio (venta vs. alquiler)?**

Se espera obtener un conteo agrupado según `gr:hasBusinessFunction` (`gr:Sell` / `gr:LeaseOut`, u otras funciones que existan en el grafo), para conocer la composición general de la oferta: qué proporción del grafo corresponde a inmuebles en venta y cuál a alquiler.

**2) ¿Cuál es el precio promedio según el tipo de inmueble (casa, departamento, local, terreno, etc.)?**

Se espera un promedio de precios agrupado por la clase concreta del `RealEstate` (subclase de `io:Casa`, `io:Departamento`, `io:Terreno`, `io:PH`, etc.), para comparar el valor de mercado entre distintos tipos de propiedades.

**3) ¿Existen inmuebles cuyo valor de precio difiere entre el valor del scraper y el valor curado manualmente?**

Se espera identificar casos donde una misma `Feature` de tipo `Price` tenga más de un origen (`hasOrigin` distinto) con valores numéricos diferentes entre sí, evidenciando discrepancias que justamente motivaron el proceso de curación manual que se menciona en la descripción de la ontología.

**4) ¿Cuáles son las cuentas (`sioc:UserAccount`) que publicaron más avisos en el grafo?**

Se espera un conteo de avisos agrupado por `sioc:has_creator`, ordenado de mayor a menor, para identificar los agentes/inmobiliarias con mayor actividad de publicación.

**5) ¿Cuál es el precio promedio, mínimo y máximo por barrio (`neighborhood`) dentro de La Plata?**

A diferencia de la Pregunta 4 original (que agrupa por ciudad), se espera una granularidad más fina, agrupando por `inmo:neighborhood`, para detectar diferencias de precio entre zonas como City Bell, Gonnet, el centro, etc.

**6) ¿Cuántos inmuebles no tienen dirección (`Address`) registrada?**

Usando `OPTIONAL` para la Feature de tipo `Address` y filtrando los casos donde no está presente (`FILTER(!BOUND(...))`), se espera detectar el volumen de inmuebles con datos incompletos, relevante para estimar la calidad/cobertura del proceso de scraping.

**7) ¿Cómo evolucionó el precio de un mismo inmueble a lo largo del tiempo, considerando distintas capturas del scraper?**

Para inmuebles cuya `Feature` de tipo `Price` tenga más de un valor con distinto `time:hasTime`, se espera obtener una serie temporal de precios por inmueble, para identificar si hubo variaciones (aumentos, descuentos) durante el período de scraping.

**8) ¿Cuál es la distribución de monedas (`gr:hasCurrency`) utilizadas en los precios del grafo?**

Se espera un conteo agrupado por `gr:hasCurrency` (por ejemplo, "USD" vs. otras monedas si existieran), para confirmar si todos los precios están expresados en la misma moneda o si es necesario normalizar antes de hacer comparaciones agregadas.

**9) ¿Cuáles son los 10 inmuebles de mayor precio en el grafo, y en qué ciudad/barrio se ubican?**

Combinando el patrón de precio con el de dirección, ordenando de forma descendente y limitando a 10 resultados (`ORDER BY DESC(?price) LIMIT 10`), se espera obtener un ranking de las propiedades más caras junto con su ubicación.

**10) ¿Qué proporción de los avisos fue publicada (`dc:date`) en un mes distinto al de la captura de sus características (`time:hasTime`)?**

Retomando la distinción identificada entre fecha de publicación y fecha de scraping, se espera cuantificar cuántos avisos tienen features cuya fecha de captura no coincide con el mes de publicación del listing, lo cual daría indicios sobre cuánto tiempo permanecen vigentes los avisos scrapeados respecto a su fecha original de publicación.
