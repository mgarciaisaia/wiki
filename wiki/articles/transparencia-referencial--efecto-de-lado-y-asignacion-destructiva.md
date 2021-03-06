---
layout: article
title: Transparencia referencial  efecto de lado y asignacion destructiva
---

Definiciones
------------

Operación:aplicar una función, evaluar un predicado, enviar un mensaje, etc.
Transparencia Referencial  
Hay transparencia referencial si al reemplazar una operación por su resultado se logra el mismo efecto.

Una definición alternativa dice: Hay transparencia referencial cuando al realizar una operación con los mismos valores siempre da el mismo resultado. Si bien esta parece más fácil de entender, no es tan precisa como la primera; puede ser útil para dar los primeros pasos, pero para el final hay que terminar de entender la otra.

<!-- -->

  
También se puede pensar en las propiedades necesarias para tener Transparencia Referencial.

Decimos que una operación tiene transparencia referencial si es:

-   Independiente: No dependen del estado de nada que este fuera de si misma
-   Sin estado/Stateless: No tiene un estado que se mantenga de llamada en llamada
-   Determinística: Siempre devuelven el mismo valor dados los mismos argumentos

Efecto de Lado/Colateral (Side Effect)  
Hay efecto de lado cuando un cambio de estado sobrevive a la realización de una operación. Por ejemplo, una operación puede modificar una variable global, modificar uno de sus argumentos, escribir datos a la pantalla o a un archivo, o hacer uso de otras operaciones que tienen efecto de lado.

Otra definición válida es:

  
Si le sacás una foto al sistema (llamémosla F1), después realizas la operación de tu interés, y le volvés a sacar una foto al sistema (F2). Si F1 y F2 son distintas =&gt; la operación que hiciste tiene efecto de lado.

<!-- -->

Asignación Destructiva  
Asignar destructivamente es reemplazar el valor de una variable por otro valor.

La [unificación](unificacion-y-pattern-matching.html) no se considera asignación (al momento de ligar no había ningún valor anterior, ¿sería más bien una inicialización?). Unificar es encontrar una sustitución capaz de igualar 2 términos. Cuando se efectiviza está sustitución hablamos de ligado de variables (tal valor se ligó a tal variable).

Ejemplos
--------

Cuando hablamos de que "algo" tiene transparencia referencial, efecto de lado o asignación destructiva, ese "algo" es la realización de una operación, de un lenguaje en particular o de un paradigma.

Estos tres conceptos suelen ir de la mano y si bien pueden darse relaciones entre ellas es saludable poder detectar la aparición de cada uno de ellos individualmente. Una relación que surge de la definición de transparencia referencial es que para ésta se dé, no puede haber efecto colateral, ya que si el estado del sistema se modifica, no es lo mismo ejecutar esa operación que reemplazar por el resultado.

A continuación mostramos algunos ejemplos en Smalltalk, ya que permite la aparición de todas estas características, para dejar más en claro de qué manera podemos identificarlas.

### Ejemplo 1: consulta no determinística

`Date today`

-   Transparencia Referencial: NO (Con cualquiera de las 2 definiciones de transparencia referencial)
-   Efecto de Lado: NO
-   Asignación Destructiva: NO

Evaluarlo con los mismos parámetros (o sea ninguno) en días distintos va a dar resultados distintos. Reemplazar la operación por el resultado una vez que cambia el día se rompe todo. Asignación destructiva y efecto de lado hay, pero en la CPU que actualiza la variable que indica el tiempo, no en el mensaje `today` que consulta ese valor (no se si es tan así, pero es a modo ilustrativo).

El efecto colateral de otra operación afecta a esta operación y le hace perder la transparencia referencial, a pesar de que esta operación por si misma NO tiene efecto de lado.

Ejemplos como este hacen que transparencia referencial y efecto colateral no sean conceptos opuestos.

### Ejemplo 2: método con efecto

`#LaColeccionConEfectoDeLado`
`>>add: unElemento`
`  "El add: siempre devuelve lo que se agrega. Acá se redefine para avisarle `
`   al elemento que fue agregado (no se me ocurrió nada mejor)"`
`  unElemento teAgregaronEn: self.`
`  ^super add: unElemento.`

-   Transparencia Referencial: NO (si te toma la definición alternativa SI, por eso dicha definición no es correcta). En un final, si se da un caso como este y están en duda, justifiquen por qué sí o no.
-   Efecto colateral: SI, porque la colección, luego de recibir el mensaje add: se modifica.
-   Asignación destructiva: no se la ve directamente en éste método, si bien puede estar presente en `teAgregaronEn:` o en `add:`. Probablemente no tenga mucho sentido hablar de asignación destructiva en este ejemplo.

Asumiendo que los parámetros siempre entienden el mensaje `#teAgregaronEn:` no importa cuantas veces se realicen estás operaciones siempre devuelven el parámetro.

`LaColeccionConEfectoDeLado new add: 4. "Devuelve 4"`
`LaColeccionConEfectoDeLado new add: pepita. "Devuelve pepita"`

Pero obviamente no es lo mismo escribir

`(LaColeccionConEfectoDeLado new add: 4) que esto (4)`

Acá se puede ver la importancia de utilizar la definición correcta de transparencia referencial.

### Ejemplo 3: método de consulta determinística

`#Number`
` >>factorial`
`    | resultado |`
`    resultado := 1.`
`    self < 0 ifTrue: [ self error: 'Como que no va pedirle el factorial a un número negativo' ].`
`    1 to: self do: [ :indice | resultado := resultado * indice ].`
`    ^resultado`

-   Transparencia Referencial: SI (con las 2 definiciones)
-   Efecto colateral: NO
-   Asignaciones Destructivas: SI

### Ejemplo 4: El efecto colateral dependiendo del contexto

`#Collection`
`select: aBlock`
`   | newCollection |`
`   newCollection := self species new.`
`   self do: [:each | (aBlock value: each) ifTrue: [newCollection add: each]].`
`   ^newCollection`

Sabemos que el método `select:` no tiene efecto colateral en sí mismo, porque no modifica la colección original (`self`), sino que devuelve una nueva de su mismo tipo.

Sin embargo podemos ver dentro de `select:` efectos colaterales en la asignación de `newCollection` y en el `add:`. Ambos pueden ser considerados como efectos colaterales dentro de la ejecución del método, pero quien usa `select:` no se da cuenta de eso y para él no tiene efecto de lado. Es decir, las asignaciones destructivas de variables locales *no presuponen* un efecto colateral para el sistema visto como un todo, ya que esos cambios de estado no perduran más allá de la ejecución del método. Sí podría analizarse como efecto *colateral* dentro del método. Probablemente en un método tan pequeño como este no tenga importancia ese tipo de análisis, pero en el caso de algoritmos más complejos podría cobrar valor (y asumiendo que no sea posible partir un algoritmo complejo en operaciones más pequeñas que simplifiquen justamente el análisis, pero eso ya es otra cuestión).

Por otra parte, aunque el `select:` no genera efectos de lado, no nos garantiza que el bloque que viene como parámetro no pueda tenerlo, por lo que uno debe tener cuidado con eso.

¿Por qué nos interesa pensar en estos conceptos?
------------------------------------------------

Estos son algunos ejemplos concretos sobre cómo la existencia o no de efecto, asignación destructiva y transparencia referencial afectan a la hora de programar.

**Separar la lógica que hace cosas de la que consulta:** Muy seguido vemos métodos (o procedimientos, dependiendo del paradigma) que tienen efecto y a su vez retornan algún valor relacionado con el mismo, estas prácticas pueden llevar a confusiones que producen un funcionamiento erróneo del sistema, sobre todo cuando el nombre del método elegido no denota que existe un efecto asociado a su ejecución. Es una buena práctica tener separada la lógica que realiza modificaciones sobre el sistema de los que sólo pretenden obtener el resultado de una consulta, que nuestros métodos tengan un único objetivo, lo cual simplifica su uso y la elección de un nombre suficientemente representativo.

**Optimizaciones:** Tener asegurada la transparencia referencial permite hacer optimizaciones como las que tiene el motor de Haskell que afectan globalmente a los programas construidos con el mismo. La [evaluación perezosa o lazy](estrategias-de-evaluacion-lazy-evaluation.html) es posible gracias a esta característica. También lo podemos ver en Prolog que para buscar soluciones utiliza el mecanismo de [Backtracking](backtracking.html) de modo que se puedan encontrar múltiples respuestas a una consulta, así como descartar los caminos por los cuales no sea posible hayar alguna, de una forma eficiente.

Otro ejemplo viene de la mano del procesamiento en paralelo. Si tenemos un conjunto con millones de elementos y queremos filtrarlo por un criterio, ¿no puedo dividir ese conjunto en varios más pequeños, filtrarlos por separado en distintos procesadores y juntar sus resultados? Si yo aseguro que evaluar el criterio de filtrado sobre cada elemento no va a provocar ningún efecto que pueda alterar mi resultado final, esta optimización podría permitir aprovechar mucho mejor el hardware disponible. Si te interesa el tema acá hay algo para leer al respecto: [Scala - Parallel Collection Framework](http://infoscience.epfl.ch/record/150220/files/pc.pdf)

**[Testing](testing.html):** El testeo unitario se basa en la premisa de que cada test sea independiente del otro y eso se logra controlando que el estado del sistema antes y después de correr cada test sea el mismo, por ese motivo es importante mantener el efecto controlado y poder revertir aquellos cambios que sobrevivan a la ejecución de cada test particular. También la transparencia referencial es importante para el testeo unitario ya que testear el resultado de una operación que depende de algo no determinístico no es viable y hace falta usar estrategias de testeo más avanzadas ([Mock Objects](http://es.wikipedia.org/wiki/Objeto_simulado)) para evitar este tipo de dependencia.

Preguntas frecuentes
--------------------

Leí una definición de Transparencia Referencial: “Hay transparencia referencial si al reemplazar una operación por su resultado se logra el mismo efecto”

Con este criterio, aquí sí habría transparencia referencial:

`int a=1;`
`int c;`
`c=a++;`

Ya que es lo mismo que hacer esto:

`int a=1;`
`int c;`
`c=1;`

Ya que el efecto en la variable c es el mismo: va a valer 1.

Pero para mí no es así, ya que no va a haber transparencia referencial, porque si bien se logra el mismo efecto con respecto a c, el sistema cambia (la variable a se incrementa en una unidad).

¿Es correcto afirmar entonces que si no hay Efecto de Lado entonces tengo garantizada la Transparencia referencial y viceversa?

### Varias respuestas

Relación entre efecto colateral y transparencia referencial:  
Ojo, las definiciones no son intercambiables. Seguramente hay sistemas con efecto de lado y que además logran transparencia referencial (cualquier sistema donde no se usen variables globales o estáticas, y no tenga funciones de entrada salida). En smalltalk no es dificil de conseguir. En c++ tampoco (siempre que uses const para marcar las cosas que querés que sean inmutables).

Sin embargo, **ausencia de efecto de lado implica transparencia referencial**, simplemente por el hecho de que el sistema no te permite cambiar el estado de las variables en un contexto dado.

En el ejemplo dado no hay transparencia referencial, es correcta la interpretación. La expresión a++ tiene el efecto colateral de modificar el valor de a, por lo tanto no puede tener transparencia referencial.

Efecto:Tal vez la duda viene por la idea de *efecto*.  
El efecto no es solamente el resultado, incluye todas las posibles consecuencias de evaluar una expresión, en particular lo que llamamos efecto de lado (o mejor dicho colateral).

Entonces si hay efecto colateral, hay efecto (y no puede haber transparencia referencial).


