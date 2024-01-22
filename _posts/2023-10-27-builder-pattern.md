---
title: "Builder pattern"
excerpt: "Separar la construcción de un objeto complejo de su representación para que un mismo proceso de construcción pueda crear diferentes representaciones."
toc: true
toc_sticky: true
author: "El viejo aprendiz"
---
 > Separate the construction of a complex object from its representation so that the same construction process can create different representations. — [GoF](https://en.wikipedia.org/wiki/Design_Patterns)

# El problema a resolver
Imaginemos que necesitamos modelar la entidad `Person`, que posee las propiedades `name`, `age`, `town`, `workplace` y `school`. Sobre estas propiedades han de cumplirse de forma invariante las siguientes condiciones:
- Las propiedades `name` y `edad` son obligatorias, esto es, no podrán ser `null`.
- El propiedad `municipio` es opcional, esto es, podrá ser informada con una cadena de texto o dejarse sin informar, tomando entonces el valor `null`.
- La propiedad `workplace` es opcional solo si la persona es mayor de edad, si es menor de edad, será  obligatoriamente `null`.
- La propiedad `school` es opcional solo si la persona es menor de edad, si es mayor, será obligatoriamente `null`.

# Una solución inmediata
Como hemos comentado antes, **siempre** que exista un objeto de la clase `Person`, cumplirá las condiciones, es decir, nunca podrá existir un objeto que no cumpla las condiciones. Por lo tanto, para no permitir que se puedan crear objetos que no cumplan las condiciones, podríamos validar todas las condiciones en el constructor de nuestra clase, de forma que al intentar crear un objeto que no cumpliera alguna de las condiciones nos saltara una excepción, por ejemplo:

```kotlin
class Person(
    private val name: String?,
    private val age: Int?,
    private val town: String?,
    private val workplace: String?,
    private val school: String?,
) {

    init {
        val fullAge = 18
        requireNotNull(name){"'name' is mandatory"}
        requireNotNull(age){"'age' is mandatory"}
        if (age < fullAge){
            require(workplace == null){
                "'workplace' must not be reported when one is underage"
            }
        }
        if (age >= fullAge){
            require(school == null){
                "'school' must not be reported when one is of legal age"
            }
        }
    }

}
```

# La solución basada en el patrón
Y aunque esto sea una solución totalmente funcional, es posible que queramos ir un paso mas allá, y ofrecer solo formas válidas de instanciar nuestra clase, esto es, no ofrecer formas de instanciación que puedan acabar provocando una excepción. Si conseguimos esto, habremos hecho la vida mas fácil a los programadores instanciadores de nuestra clase, y al tiempo, nuestro sistema será mas robusto al haber acotado el número de formas posibles en las que se pueden generar errores en tiempo de ejecución. Sin embargo, nos encontramos ante un nuevo desafío, el de presentar tantas formas de instanciar un objeto como combinaciones válidas de parámetros y argumentos existan.

## Analizando los requisitos
Si hacemos un análisis exhaustivo de los requisitos podemos comprobar que existen 8 combinaciones válidas de parámetros, siendo todas las restantes opciones no válidas. En la tabla de abajo, donde cada columna representa un parámetro de construcción, están representadas todas las posibles combinaciones de argumentos y, cada fila representa un subconjunto de la totalidad. Las filas rojas representan combinaciones no válidas y debemos interpretar los valores de la tabla de la siguiente manera:

- "_not reported_" indica que se pretende instanciar el objeto sin pasar valor (o pasando el valor null) en el parámetro representado por la columna correspondiente.
- "_reported_" indica que se pretende instanciar el objeto pasando un valor distinto de null en el parámetro representado por la columna correspondiente.
- "_minor_" solo se utiliza en la columna `age` e indica que se pretende instanciar el objeto pasando un valor numérico, no nulo, menor de 18.
- "_major_" solo se utiliza en la columna `age` e indica que se pretende instanciar el objeto pasando un valor numérico, no nulo, mayor o igual a 18.
- "--" representa cualquier valor, es decir, un valor nulo o no nulo.


<table>
    <thead>
        <tr align="center">
            <th>Combinación</th>
            <th>name</th>
            <th>age</th>
            <th>town</th>
            <th>workplace</th>
            <th>school</th>
        </tr>
    </thead>
    <tbody align="center">
        <tr style="background-color: #f87168">
            <td>1</td>
            <td>not reported</td>
            <td>--</td>
            <td>--</td>
            <td>--</td>
            <td>--</td>
        </tr>
        <tr style="background-color: #f87168">
            <td>2</td>        
            <td>reported</td>
            <td>not reported</td>
            <td>--</td>
            <td>--</td>
            <td>--</td>
        </tr>
        <tr>
            <td>3</td>        
            <td>reported</td>
            <td>minor</td>
            <td>not reported</td>
            <td>not reported</td>
            <td>not reported</td>
        </tr>        
        <tr>
            <td>4</td>        
            <td>reported</td>
            <td>minor</td>
            <td>not reported</td>
            <td>not reported</td>
            <td>reported</td>
        </tr>        
        <tr>
            <td>5</td>        
            <td>reported</td>
            <td>minor</td>
            <td>reported</td>
            <td>not reported</td>
            <td>not reported</td>
        </tr>           
        <tr>
            <td>6</td>        
            <td>reported</td>
            <td>minor</td>
            <td>reported</td>
            <td>not reported</td>
            <td>reported</td>
        </tr>                
        <tr style="background-color: #f87168">
            <td>7</td>        
            <td>reported</td>
            <td>minor</td>
            <td>--</td>
            <td>reported</td>
            <td>--</td>
        </tr>                
        <tr>
            <td>8</td>        
            <td>reported</td>
            <td>major</td>
            <td>not reported</td>
            <td>not reported</td>
            <td>not reported</td>
        </tr>                    
        <tr>
            <td>9</td>        
            <td>reported</td>
            <td>major</td>
            <td>not reported</td>
            <td>reported</td>
            <td>not reported</td>
        </tr>           
        <tr>
            <td>10</td>        
            <td>reported</td>
            <td>major</td>
            <td>reported</td>
            <td>not reported</td>
            <td>not reported</td>
        </tr>                
        <tr>
            <td>11</td>        
            <td>reported</td>
            <td>major</td>
            <td>reported</td>
            <td>reported</td>
            <td>not informed</td>
        </tr>                
        <tr style="background-color: #f87168">
            <td>12</td>
            <td>reported</td>
            <td>major</td>
            <td>--</td>
            <td>--</td>
            <td>reported</td>
        </tr>                                                                                
    </tbody>
</table>


## El código
Pues bien, el builder pattern nos ayuda a modelar todos estos requisitos en el código sin necesidad de crear constructores para cada una de las combinaciones (si es que esto fuera posible para el caso que tuviéramos entre manos). Además de permitirnos ir guiando al programador durante la construcción de objetos solo mostrando las opciones que son válidas:

```kotlin
class Person private constructor (
    private val name: String, val town: String?){

    var age: Int? = null; private set
    var workplace: String? = null; private set
    var school: String? = null; private set

    companion object {

        fun builder(name: String, town: String? = null): Builder 
            = Builder(name, town)

        class Builder(name: String, town: String?){
            private val person: Person = Person(name, town)

            fun setMajor(age: Int): MajorBuilder{
                if (age < 18) error("age has to be older than 18")
                person.age = age
                return MajorBuilder(person)
            }

            fun setMinor(age: Int): MinorBuilder{
                if (age >= 18) error("age has to be younger than 18")
                person.age = age
                return MinorBuilder(person)
            }

        }

        class MajorBuilder(val person: Person){
            fun setWorkplace(workplace: String): MajorBuilder{
                person.workplace = workplace
                return this
            }

            fun build() = person
        }

        class MinorBuilder(val person: Person){
            fun setSchool(school: String): MinorBuilder{
                person.school = school
                return this
            }

            fun build() = person
        }
    }
}
```

### Explicación
Si analizamos el código, vemos que el constructor de `Person` es privado, esto es, no podrá ser instanciado de forma ordinaria, solo podrá intanciarse desde dentro de la propia clase. Por otra parte, vemos que este constructor privado tiene solamente dos parámetros, `name` y `town`. ¿Por qué solo estos dos parámetros? Bien, por que son los únicos parámetros donde los requisitos que han cumplirse son independintes del resto de parámetros, quiero decir, las condiciones que han de cumplir estos dos parámetros no dependen del valor otros parámetros y tampoco las condiciones de otros parámetros se ven afectadas por el valor de éstos. Tal vez todo vaya adquiriendo más sentido conforme avancemos en el análisis.

Además, también observamos que la clase `Person` posee el resto de propiedades: `age`,`workplace` y `school`, que pueden ser leidas de forma pública pero solo pueden ser modificadas desde dentro de la clase.

Como vemos en el código, no ofrecemos un contrucutor público, sin embargo, ofrecemos un método publico estático llamado `builder` que aceptará los dos parámetros de los que hablábamos antes y nos ayudará con la instanciación de la clase. Al llamar a este método estático, lo que obtenemos no es un objeto de tipo `Person`, si no un objeto de tipo `Builder`, el cual posee internamente una instancia de `Person` todavía en construcción, por el momento, solo se le han inicializado las propiedades `name` y `town`.

Este objeto `Builder` devuelto nos ofrece dos métodos, que representan los dos posibles caminos de instanciación que queremos seguir, es decir, a tavés de los métodos `setMajor` y `setMinor` nos obliga a decidir en este momento si queremos instanciar una persona mayor o menor de edad teniendo que aportar obligatoriamente un valor para la edad, a su vez, la llamada a uno de estos métodos, establecerá el valor correspondiente en la propiedad `age` de la instancia `Person` en construcción y nos devolverá otro builder, un `MajorBuilder` o un `MinorBuilder`, dependiendo del método al que hayamas invocado. De forma que un `MajorBuilder` nos ofrecerá la opción de dar un valor para `workplace` pero no tendremos forma de dar un valor para `school`, puesto que las personas mayores de edad no han de informar de `school`, y de forma equivalente `MinorBuilder` nos permite dar un valor a `school` pero no nos deja dar un valor a `workplace`. Ambos builders, `MajorBuilder` y `MinorBuilder`, nos ofrecerán la posibilidad, a parte de establecer valores a los campos pertinentes, de instanciar finalmente el objeto `Person` a través de su método `build()`.

### Instanciaciones válidas
Un ejemplo de instanciación de cada una de las 8 combinaciones válidas de la tabla anterior serían las siguientes:


```kotlin
fun main(){

    val combi3 = Person.builder("Juan")
        .setMinor(15)
        .build()

    val combi4 = Person.builder("Juan")
        .setMinor(15)
        .setSchool("C.P. Gabriela Mistral")
        .build()

    val combi5 = Person.builder("Juan","Valencia")
        .setMinor(15)
        .build()

    val combi6 = Person.builder("Juan","Valencia")
        .setMinor(15)
        .setSchool("C.P. Gabriela Mistral")
        .build()

    val combi8 = Person.builder("Juan")
        .setMajor(35)
        .build()

    val combi9 = Person.builder("Juan")
        .setMajor(35)
        .setWorkplace("My office")
        .build()

    val combi10 = Person.builder("Juan","Valencia")
        .setMajor(35)
        .build()

    val combi11 = Person.builder("Juan","Valencia")
        .setMajor(35)
        .setWorkplace("My office")
        .build()
}
```

### Instanciaciones inválidas
Provocando error de compilación el intento de crear una instancia mal parametrizada, cualquiera de estas combinaciones no compilará:

```kotlin
val combi1 = Person.builder().build() 
// NO COMPILA. No value passed for parameter 'name'
```

```kotlin
val combi2 = Person.builder("Juan").build()
// NO COMPILA. Unresolved reference: build
```

```kotlin
val combi2 = Person.builder("Juan").build()
// NO COMPILA. Unresolved reference: build
```

```kotlin
val combi7 = Person.builder("Juan")
    .setMinor(15)
    .setWorkplace("My office")
    .build()
// NO COMPILA. Unresolved reference: setWorkplace
```

```kotlin
val combi12 = Person.builder("Juan")
        .setMajor(35)
        .setSchool("C.P. Gabriela Mistral")
        .build()
// NO COMPILA. Unresolved reference: setSchool
```
