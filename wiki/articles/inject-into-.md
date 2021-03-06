---
layout: article
title: Inject into 
---

El **inject:into:** es un solo mensaje que entienden todas las colecciones, recibe 2 parámetros.

`  `

Utilidad
--------

La idea de este mensaje es realizar una operación de acumulación sobre todos los elementos de una colección.

Parámetros del **inject:into:**
-------------------------------

El primero parámetro del **inject:into:** (valorInicialAcumulador) representa el objeto al cual va a apuntar el acumulador antes de empezar a evaluar el bloque (el 2do parámetro del **inject:into:**) sobre cada elemento de la colección.

El segundo parámetro del **inject:into:** (bloqueConDosParametros) es un bloque que va a ser evaluado sobre cada elemento de la colección y el acumulador actual.

bloqueConDosParametros = \[ :acumuladorActual :elementoDeLaColeccion | <acá escribimos el resultado que representa al nuevo valor del acumulador > \]

El **inject:into:** retorna el valor final del acumulador

Ejemplos
--------

### Sumatoria de una colección de números

Si col es una colección de números y queremos obtener su sumatoria podemos hacer

`  col inject: 0 into: [ :sumatoria :numero | sumatoria + numero ]`

valorInicialAcumulador = 0

acumulador = sumatoria

elementoDeLaColeccion = numero

operación que retorna el nuevo valor del acumulador = sumatoria + numero

### Productoria de una colección de números

Si col es una colección de números y queremos obtener su productoria podemos hacer

`  col inject: 1 into: [ :productoria :numero | productoria * numero ]`

valorInicialAcumulador = 1

acumulador = productoria

elementoDeLaColeccion = numero

operación que retorna el nuevo valor del acumulador = productoria \* numero

### Máximo de una colección de números

Si col es una colección de números y queremos obtener el máximo podemos hacer

`  col inject: col anyOne into: [ :maximo :numero | maximo max: numero ]`

valorInicialAcumulador = col anyOne (algún elemento de la colección)

acumulador = maximo

elementoDeLaColeccion = numero

operación que retorna el nuevo valor del acumulador = maximo max: numero

### La altura más alta de una colección de personas

Si col es una colección de personas y queremos obtener la altura más alta

`  col inject: col anyOne altura into: [ :maximaAltura :persona | maximaAltura max: persona altura ]`

valorInicialAcumulador = col anyOne altura (alguna altura de alguna persona de la colección)

acumulador = maximaAltura

elementoDeLaColeccion = persona

operación que retorna el nuevo valor del acumulador = maximaAltura max: persona altura

### La persona más alta de una colección de personas

Si col es una colección de personas y queremos obtener la persona más alta

`  col inject: col anyOne into: [ :personaMasAlta :persona | personaMasAlta altura > persona altura ifTrue: [personaMasAlta] ifFalse: [persona] ]`

valorInicialAcumulador = col anyOne (alguna persona de la colección)

acumulador = personaMasAlta

elementoDeLaColeccion = persona

operación que retorna el nuevo valor del acumulador = personaMasAlta altura &gt; persona altura ifTrue: \[personaMasAlta\] ifFalse: \[persona\]

Nota: si asumimos que persona es instancia de la clase Persona podemos hacer lo siguiente

`  col inject: col anyOne into: [ :personaMasAlta :persona | personaMasAlta personaMasAlta: persona ]`

`  Persona >> personaMasAlta: otraPersona`
`    ^self altura > otraPersona altura`
`         ifTrue: [ self ] ifFalse: [ otraPersona ]`

### Aplanando colecciones

Si col es una colección donde cada uno de sus elementos son colecciones y quiero obtener una colección con todos los elementos de sus subcolecciones puedo hacer

`  col inject: Bag new into: [ :unaColeccionConTodosLosElementos :unaColeccion | unaColeccionConTodosLosElementos addAll: unaColeccion. unaColeccionConTodosLosElementos ] "Hay que retornar explícitamente unaColeccionConTodosLosElementos"`

Si no nos interesan los elementos repetidos podemos escribir

`  col inject: Set new into: [ :unaColeccionConTodosLosElementos :unaColeccion | unaColeccionConTodosLosElementos union: unaColeccion ]`

Al menos en Pharo recordar que:

- \#union: retonarna una instancia de Set

- \#add, \#addAll:, \#remove:, \#removeAll: retornan el objeto que recibieron como parámetro **NO** retornan al objeto receptor del mensaje
