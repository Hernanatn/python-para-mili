# Objetos. *Parte 1)*
**Estructura General**

Todo en Python, en última instacia, es un objeto que hereda direca o indirectamente de la clase `object`. Esto tiene agunas propiedades particulares, la más importante respecto de a POO, refiere a las implicancias que tiene sobre como los objetos son alocados en memoria y como se implementan Clases.

Las clases se declaran con la palabra clave `class`, pueden poseer atributos y las funciones internas -métodos- siempre llevan como primer argumento una referencia a la instancia, `self`.

## Declaración de Clases
```Python
class MiClase():
    def __init__(self) -> None:
        self.atributo = ...
        ...
    def metodo(self):
        ...
```

## Atributos

### Atributos de Instancia y Atributos de Clase
Las clases en Python pueden tener tanto atributos vinculados a una instancia particular como a la clase en sí.
Los atributos de cada instancia se declaran dentro de un método y llevan una referencia a `self`, mientras que los atributos de clase se declaran por fuera de los métodos.

```Python
class MiCLase():
    atributoDeClase : int = 1

    def __init__(self, attr : str) -> None:
        self.atributoDeInstancia = attr
        ...
```
Tanto la clase en sí como todos los objetos de tipo **MiClase** poseen el atributo **atributoDeClase**, cuyo valor es 1. Mientras que el valor del **atributoDeInstancia** dependerá de cada instancia concreta.

Los atributos de clase resultan útiles, por ejemplo, cuando implementando un patrón _"Singleton"_
```Python
class MiSingleton():
    _unicaInstancia = None

    def __new__(cls,*posicionales,**nominales):
        if cls._unicaInstancia is None:
            cls._unicaInstancia = super().__new__(cls,*posicionales,**nominales)
        return cls._unicaInstancia
```
En el ejemplo anterior, el **constructor** de la instancia primero revisa si la variables _unicaInstancia de la Clase es None, y solo construye una nueva en ese caso. Si ya existe una _unicaInstancia creada previamente, devuelve esa en lugar de instancias una nueva, garantizando el *"Singleton"*.

### Atributos "privados" vía *name mangling*
En Python todos los atributos y métodos de una clase son públicos. Siempre.
Existe, sin embargo, una comodida del compilador que permite simular la funcionalidad de protección de variables.
Esta funcionalidad se llama *"name mangling"* y se utiliza prefijando la variable o método con dos __ guiones bajos.

```Python
class MiClasePublica():

    def __init__(self, attr : str) -> None:
        self.nombre = attr
        ...

...
instancia : MiClasePublica = MiClasePublica("juan")
print(instancia.nombre)
# >>> juan

class MiClasePrivada():

    def __init__(self, attr : str) -> None:
        self.__nombre = attr
        ...

...
instancia : MiClasePrivada = MiClasePrivada("pablo")
print(instancia.__nombre)

# >>> AttributeError object of type 'MiClasePrivada' does not have a '__nombre' attribute.
```
Pareciera ser que ese *name mangling* efectivamente "privatiza" el atributo nombre y lo torna inaccesible por fuera de la clase. Pero no, lo único que hace es, en *tiempo de compilación*, cambiar el nombre de la variable, agregando como prejifo el nombre de la clase. Y podemos seguir accediendo a la variable siempre que lo hagamos con su nombre *mangleado*.

```Python
class MiClasePrivada():
    ...

instancia : MiClasePrivada = MiClasePrivada("pablo")
print(instancia._MiClasePrivada__nombre)

# >>> pablo
```

### Atributos y objetos en memoria `__slots__` vs `__dict__`
Por defecto, todos los objetos de Python implementan el atributo `__dict__` que es un diccionario de todos los atributos y métodos "públicos" de la instancia.  
Este diccionario se almacena en el *heap* y es referenciado por un puntero en el *stack frame* del programa (ver vi. Esquema de Objetos en Memoria).

El diccionario es una estructura de datos variable y dinámica. Esto implica, al menos, dos cuestiones:
+La alocción de memoria del diccionario debe ser dinámica
+La estructura intera del objeto, i.e. los atributos que *puede* tener, pueden variar luego de su inicialización.

```Python
class MiClaseDict():    

    def __init__(self, nombre : str, edad : int):
        self.nombre = nombre
        self.edad = edad 

dicto : MiClaseDict = MiClaseDict("juan",5)

print(dicto.__dict__)
# >>> {'nombre': 'juan', 'edad': 5}

dicto.nuevoAtributo = "nuevo atributo" #es perfectamente legal agregar atributos en tiempo de ejecución
print(dicto.__dict__)
# >>> {'nombre': 'juan', 'edad': 5, 'nuevoAtributo': 'nuevoAtributo'}
```

La implementación de `__dict__` puede suponer penalidades de rendimiento, como así permitir anti-patrones como la modificación de las clases en curso de ejecución. Existe, sin embargo, una alternativa.

Python permite declarar el atributo de clase `__slots__`. Este reemplaza al diccionario por una tupla de `str` la cual contiene los nombres de los atributos de la clase. La tupla es una estructa inmutable, y el alocador de memoria de Python aprovecha esto cambiando la estructura del objeto en memoria, guardando en el `stack` un puntero por cada atributo (los cuales viven en el `heap`).

```Python
class MiClaseSlots():
    __slots__ = ("nombre","edad") # 

    def __init__(self, nombre : str, edad : int):
        self.nombre = nombre
        self.edad = edad 

sloto : MiClaseSlots = MiClaseSlots("pablo",7)

print(sloto.__slots__)
# >>> ('nombre','edad')

print(sloto) 
sloto.nuevoAtributo = "nuevo atributo" #es ilegal modificar una tupla y por lo tanto no se pueden agregar nuevos atributos
# >>> AttributeError: 'MiClaseSlots' object has no attribute 'nuevoAtributo'
```
> *Nota: una tupla en python pesa ~40bytes, mientras que un diccionario pesa ~232bytes. Lo que representa una diferencia espacial de 580%.*  
*La magnitud absoluta no es tanta, pero en un programa que, por ejemplo, en el curso de ejecución instancie 1.000.000 de objetos (lo cual no resulta tan disparatado para un programa de mediana complejidad como puede ser un webscrapper) la implementación de `__dict__` implica 192 Mb de memoria extra utilizados tan sólo para instanciar aquellos objetos.* 

## Métodos
### Métodos de Instancia
Los métodos de instancia son las funciones vinculadas al objeto que toman como primer argumento una referencia a la instancia, `self`.
```Python
class Numero():
    def sumar(self,otroNumero : int):
        return self += otro
```
### Métodos de Clase
Además de métodos de instancia, las clases pueden tener métodos en sí mismas. Estos métodos no dependen de ninguna instancias en particular ni pueden acceder a otros atributos que no sean los de la clase.  
Los métodos de clase se indican con el decorador `@classmethod` y toman como primer argumento una referencia a la clase, `cls`.

```Python
class strFormateada(str):
    def __init__(self, cadena : str, formato : str):
        self.data = cadena.format(formato)
        ...
    
    @classmethod
    def crearDesdeCSV(cls, rutaCsv : str) -> strFormateada:
        with open(ruta, ...) as csv:
            cadena, formato = csv.read().split(",")
            return strFormateada(cadena,formato)

cadenaNueva = strFormateada.crearDesdeCSV("ruta/al/archivo.csv")
```

## Métodos 'Mágicos' (*__dunder__*)
### Construcción e Instanciación: __init()__ vs __new()__
<div style="text-align: justify">
Si bien los objetos en python se instancian con el método "mágico" <code>__init__()</code>. Este no es en verdad un constructor. En verdad, el interprete, justo antes de llamar <code>__init__()</code> invoca otro método "mágico", <code>__new__()</code>, que es el verdadero encargado de crear y retornar el objeto. Por eso<code>__init()__</code> retorna <code>None</code>.
</div><br>
<div style="text-align: justify">
Dado que Python no expone al programador las operaciones de alocación de memoria, en general no es necesario utilizar <code>__new__()</code>, sinembargo esto puede ser útil en el patrón *"Singleton"* o, por ejemplo, para la implementación de Clases del usuario que dependen de <code>secuencias inmutables</code> como las tuplas, ya que estas son inmutables desde el instante en que son creadas.
</div><br>
<div style="text-align: justify">
Si intentáramos crear una tupla de <code>str</code> cuyos miembro fueran todos mayúsculos indistintamente de si fueron cargados en mayúscula o en mínuscula utilizando <code>__init__</code>, recibiríamos un error
</div><br>

```Python
from typing import Iterable

class TuplaMayus(tuple):
    def __init__(self, iterable : Iterable) -> None:
        for indice, miembro in enumerate(iterable):
            self[indice] = miembro.upper()
...

#>>> TypeError 'TuplaMayus' object does not support item assignment

```
<div style="text-align: justify">
Esto sucede por que la tupla, al ser una colección inmutable no permite modificar sus elementos una vez que fue creada. En su lugar se puede usar <code>__new__</code>:
</div>

```Python
from typing import Iterable

class TuplaMayus(tuple):
    def __new__(cls, iterable : Iterable) -> TuplaMayus:
        iterableMayus : Iterable = (e.upper() for e in iterable)
        return super().__new__(cls,iterableMayus)
```
> *Nota: `__new__ siempre debe retornar una instancia de la clase, es el constructor`*

### Métodos @overload
<div style="text-align: justify">
</div><br>
Python "vainilla" no permite la sobrecarga de métodos y funciones, sin embargo, la biblioteca estándar "overloading" incluye el decorador con aquella funcionalidad.

```Python
from overloading import overload
from collections import Iterable
class Perro:
    
    @overload
    def alimentar(comida : int):       
        self.hambre -= comida

    @overload
    def alimentar(comidas : Iterable[int]):
        comida : int
        for comida in comidas:
            alimentar(comida)


# El compilador expande el overload a un único método "alimentar" cuya implementación es masomenos:  
def alimentar(comida : int | Interable[int]):
    if isinstance(comida,Iterable):
        c : int
        for c in comida:
            self.hambre -= c
    else:
        self.hambre -= comida
```

## Herencia, `super().__init__()`
<div style="text-align: justify">
Python permite que clases hereden a otras, absorviendo sus atributos y métodos. Sin embargo, para que un objeto de una clase *sea* de tipo de su clase madre, debe, al momento de instanciacion, instanciar tambien con el método de su madre. Esto se logra a traves de la referencia que todas las clases tiene a su clase inmediata superior, <code>super()</code> 
</div>

```Python
class ClaseMadre():

class ClaseHija(claseMadre):
    def __init__(self):
        super().__init__(self):

``` 
*Nota todas las clases heredan implícitamente a `object`*


## `type`, `super()`, *`__class__`*, *`__base__`*, *`__subclass__`*, `metaclass=`

###

### Creación de Tipos en tiempo de Ejecución