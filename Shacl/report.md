
# Reporte de Validación SHACL para la Ontología de los Premios Oscar

## 1. Introducción y Objetivo

Este documento detalla el proceso de validación del Grafo de Conocimiento (`Knowledge Graph`) de los Premios Oscar. El objetivo principal es garantizar la **calidad, consistencia e integridad** de los datos poblados según la ontología `oscar` y los vocabularios reutilizados (`schema`, `dbo`, `foaf`).

Para ello, se han definido un conjunto de reglas en SHACL (`shapes.ttl`) que actúan como un sistema de validación automático. A continuación, se explican estas reglas, la estrategia de datos para la evaluación y el análisis de los resultados obtenidos.

## 2. Descripción de las Restricciones SHACL Implementadas (`shapes.ttl`)

Se han implementado **6 SHACL Shapes** principales, cubriendo un amplio espectro de tipos de restricciones como se exige en el enunciado de la práctica:

1.  **`oscar:NominationShape`:** Es la Shape más compleja y agrupa múltiples restricciones sobre la clase central `oscar:Nomination`.
    *   **Cardinalidad y Tipo:** Valida que cada nominación tenga exactamente una persona (`sh:minCount 1`, `sh:maxCount 1`) y una película (`sh:minCount 1`), y que ambas sean entidades URI (`sh:class`).
    *   **Tipo de Dato (`sh:datatype`):** Obliga a que la propiedad `oscar:isWinner` sea de tipo `xsd:boolean`.
2.  **`schema:EventYearShape`:**
    *   **Rango Numérico (`sh:minInclusive`):** Asegura la integridad histórica de los datos, forzando a que la propiedad `oscar:year` sea un número entero igual o superior a 1927, año de la primera ceremonia.
3.  **`schema:EventCeremonyShape`:**
    *   **Rango Numérico (`sh:minInclusive`):** Valida la consistencia de los identificadores, exigiendo que `oscar:ceremonyNumber` sea 1 o mayor.
4.  **`schema:PersonNameShape`:**
    *   **Restricción de Cadena de Texto (`sh:minLength`):** Garantiza que todos los individuos de la clase `schema:Person` tengan un nombre (`schema:name`) que no esté vacío.
5.  **`schema:PersonDemographicsShape`:**
    *   **Enumeraciones (`sh:in`):** Para los atributos demográficos como `dbo:religion`, `dbo:race` y `dbo:sexualOrientation`, se ha creado un **vocabulario controlado** a partir de los valores únicos extraídos del dataset original. Esto previene la introducción de datos sucios o erróneos.
6.  **`oscar:CharacterShape`:**
    *   **Shape Cerrada (`sh:closed`):** Es una restricción que prohíbe añadir propiedades a la clase `oscar:Character` que no estén explícitamente definidas en la Shape (`schema:name`). Esto garantiza que la estructura de la clase se mantenga simple y no se contamine con atributos no previstos.

## 3. Estrategia de Muestreo de Datos (`data.ttl`)

Para llevar a cabo una validación completa, el archivo `data.ttl` se ha dividido en dos secciones:

*   **Datos Positivos:** Se han instanciado todos los conceptos y relaciones (`data:Oscars1928`, `data:CharlesChaplin`, `data:Nomination1`, etc.) siguiendo rigurosamente las reglas definidas en `shapes.ttl`.
*   **Datos Negativos:** Se han creado deliberadamente instancias "rotas" para poner a prueba cada una de las 6 Shapes. Por ejemplo:
    *   `data:BadEvent` tiene un año anterior a 1927 (`1920`).
    *   `data:BadPerson` tiene una religión inventada ("Jedi") que no está en la lista `sh:in`.
    *   `data:BadNomination` tiene `oscar:isWinner` como un string ("Sí, ganó") en vez de un booleano.
    *   `data:BadCharacter` incluye una propiedad no permitida (`oscar:weapon`) para probar la regla `sh:closed`.

## 4. Análisis del Reporte de Validación (`report.ttl`)

Tras ejecutar el validador `pySHACL` con el comando `pyshacl -s shapes.ttl -f turtle -o report.ttl data.ttl`, se generó un reporte de validación que confirma que **el grafo de datos no es conforme (`sh:conforms false`)**.

El análisis del reporte (`report.ttl`) revela que el sistema ha detectado correctamente todas las violaciones que habíamos introducido:

*   El validador detectó el error de rango de valor en `data:BadEvent`, ya que `1920` es menor que el `sh:minInclusive 1927` definido.
*   En `data:BadPerson`, el validador falló debido a la restricción `sh:in`, reportando que el valor "Jedi" no pertenecía a la lista de religiones permitidas.
*   En `data:BadNomination`, el validador reportó la violación de `sh:DatatypeConstraintComponent` porque el valor "Sí, ganó" no es `xsd:boolean`.
*   En `data:BadCharacter`, el validador reportó la violación de `sh:ClosedConstraintComponent`, indicando que la propiedad `oscar:weapon` no está permitida en esta Shape cerrada.

