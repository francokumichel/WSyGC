# Actividad OWL

## Diseño de la ontología
El diseño de la ontología se realizará en base al articulo de [protegé](https://protege.stanford.edu/publications/ontology_development/ontology101.pdf).

### Paso 1: Determinar el dominio y alcance de la ontología
Hay diferentes puntos claves que debe resolver el diseño e implementación de la ontología, los cuales son:

- **¿Que dominio va a cubrir esta ontología?**. El dominio es el del mercado de la oferta inmobiliaria, donde se hará foco particular en dos tipos de inmuebles, que son `casas` y `lotes`. La ontología va a abarcar tanto características físicas como técnicas de los inmuebles con la información comercial de los avisos (precios, inmobiliarias, agentes, etc).
- **¿Para que vamos a usar esta ontología?**. Para construir una base de conocimiento que nos permita organizar, visualizar y consultar de forma estructurada la oferta inmobiliaria. Todo esto con el fin de facilitar el emparejamiento entre las necesidades de un cliente y los avisos disponibles en el mercado.
- **¿Quienes van a usar y mantener la ontología?**. En este caso esta hecja con fines didácticos, pero en un caso real los potenciales clientes en busca de viviendas serian los usuarios y los que la mantengan serían los agentes inmobiliarios que actualizan avisos y valores.

Algo que se plantea en el articulo de protegé es realizar preguntas de competencia a modo de "prueba de fuego" con el fin de determinar si la ontología tiene suficiente detalle. Basándome en el enunciado de la actividad propuesta la ontología debería responder las siguientes preguntas que las organizo en diferentes secciones:

- **Filtrado por servicios y dimensiones**: ¿Cuáles son los inmuebles con una superficie de lote mayor a X metros cuadrados qye se encuentran en un barrio con servicios de luz, gas y cloacas?
- **Búsqueda por perfil de vivienda**: ¿Que inmobiliaria posee casas que coincidan con un perfil específico (ej: 3 dormitorios y mas de $80m^2$)?
- **Categorización por valor y zona**: ¿Qué casas en el barrio de Manuel B. Gonnet califican como "baratas" (superficie > $90m^2$ y valor < $120.000$ USD)?
- **Responsabilidad comercial**: ¿Quién es el agente inmobiliario responsable de un aviso determinado y a qué inmobiliaria pertenece?

## Paso 2: Considerar reutilizar ontologías existentes
Si bien hay ontologías ya existentes, como por ejemplo GEO(WGS84 POS) para representar la ubicación geográfica mediante latitud y longitud, para fines de aprendizaje voy a asumir que no existen ontologías relevantes y comenzaré desde cero para dominar el proceso de diseño.

## Paso 3: Enumerar terminos importantes 
Basandome en los requerimientos específicos de la actividad, la lista de términos esenciales es la siguiente:

- **Terminos relacionados a un inmueble**: Inmueble, Casa, Lote, Dirección postal, Latitud y Longitud, Barrio, Dimensiones, Habitaciones, Superficie cubierta, Servicios, Amenities
- **Términos relacionados con la oferta comercial**: Aviso inmobiliario, Valor, Moneda, Monto, Inmobiliaria, Agente inmobiliario
- **Otros términos de interés**: Perfil de vivienda (podrian ser criterios especificos de viviendas), Rango de precios ("barato" o "caro" según el monto), Ubicación relativa (como podría ser "es vecino de" para los barrios)

## Paso 4: Definir clases y la jerarquia de clases
La jerarquía de clases que se propone es la siguiente:

- `Inmueble`: Clase raíz
    - `Casa` es sublclase de `Inmueble`. Es un tipo de inmueble que posee habitaciones y superficie cubierta.
    - `Lote` es subclase de `Inmueble`. Serían terrenos baldíos definidos por sus dimensiones.
- `Aviso`: Representa la publicación comercial del inmueble, vinculando valor, inmobiliaria y agente.
- `Organizacion`:
    - `Inmobiliaria`: subclase de `Organizacion`. Sería la empresa que publica los avisos.
- `Persona`
    - `Agente`: subclase de `Persona`. Individuo responsable de la oferta
- `ConceptoGeografico`
    - `Barrio`: clase para representar las zonas
- `Servicio`: Clase para representar elementos como luz, gas o cloacas
- `CasasBaratasDeGonnet`: clase definida. Se establecería mediante un axioma de equivalencia que combine:
    - Ser parte de la clase `Casa`
    - Tener una relación con el `Barrio` "Manuel B. Gonnet"
    - Tener una `superficieCubierta` > 90
    - Tener un `montoValor` < 120000

## Paso 5: Definir propiedades de clases (Slots o roles en el libro)
Para este paso vamos a dividir las propiedades en *relaciones entre clases* y *atributos de datos*

### Object properties
- `ubicadoEnBarrio`
    - **Dominio**: `Inmueble`; **Rango**: `Barrio`
    - **Lógica**: Permite saber en qué zona esta la casa o el lote.
- `ofreceInmueble`
    - **Dominio**: `Aviso`; **Rango**: `Inmueble`
- `publicadoPor`
    - **Dominio**: `Aviso`; **Rango**: `Inmobiliaria`
- `tieneAgenteResponsable`
    - **Dominio**: `Aviso`; **Rango**: `Agente`
    - **Característica**: Funcional. Un aviso tiene un único agente responsable.
- `tieneServicio`
    - **Dominio**: `Inmueble`; **Rango**: `Servicio`
- `esVecinoDe`
    - **Dominio**: `Barrio`; **Rango**: `Barrio`
    - **Característica**: Simétrica. Si el barrio A es vecino del barrio B, el B necesariamente es vecino del A.
- `esParteDe`
    - **Dominio**: `ConceptoGeografico`; **Rango**: `ConceptoGeografico`
    - **Característica**: Transitiva. Si un sector es parte de Gonnet y Gonnet es parte de La Plata, el sector es parte de La Plata.

### Datatype properties
- Propiedades de `Inmueble`:
    - `direccionPostal`: Tipo `String`
    - `latitud` y `longitud`: Tipo `decimal`
- Propiedades específicas de `Casa`
    - `cantidadHabitaciones`: Tipo `Integer`
    - `superficieCubierta`: Tipo `Integer`
- Propiedades específicas de `Lote`
    - `ladoLote` : Tipo `decimal`
- Propiedades de `Aviso`
    - `montoValor`: Tipo `decimal`
    - `monedaValor`: Tipo `String` (EJ: USD, ARS)

## Paso 6: Definir las facetas de las propiedades
### Cardinalidad de las propiedades
- **Cardinalidad única (Single Cardinality)**: `direccionPostal`, `montoValor`, `tieneAgenteResponsable`, `ubicadoEnBarrio`
- **Cardinalidad Múltiple (Multiple Cardinality)**: `tieneServicio`, `publicaAviso`
### Tipos de valor
- **String**: `direccionPostal`
- **Decimal**: `latitud`, `longitud`, `montoValor`, `largoTerreno`, `anchoTerreno`
- **Integer**: `cantidadHabitaciones`, `superficieCubierta`
- **Enumerated**: para `monedaValor` podemos restringir los valores permitidos a por ejemplo `{USD, ARS}`
- **Instance**: `ubicadoEnBarrio`, `tieneAgenteResponsable`
### Relaciones inversas

- `tieneAgenteResponsable` (Aviso -> Agente) es inversa de `gestionaAviso` (Agente -> Aviso)
- `publicadoPor` (Aviso -> Inmobiliaria) es inversa de `publicaAviso` (Inmobiliaria -> Aviso)
- `ofreceInmueble` (Aviso -> Inmueble) es inversa de `SeOfertaEn` (Inmueble -> Aviso)

### Valores por defecto
Se podría setear como valor por defecto "USD" para `monedaValor`, ya que usualmente en los avisos publican el precio en dolares.

## Paso 7: Crear instancias
