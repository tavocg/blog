---
title: 'Apoderarse de no inicializar valores'
date: '2026-01-18'
tags: ['golang']
---

El concepto de inicialización es prácticamente un dogma, y en muchos casos innecesario. Con esto no quiero decir que prefiero comportamiento indefinido, sino que la _[Convención sobre configuración](https://rubyonrails.org/doctrine/es#convention-over-configuration)_ es una filosofía poderosa que ahorra líneas de código sin necesidad de perder control sobre el comportamiento de un programa.

## Zero values

Por supuesto, no cualquier lenguaje se presta para esta filosofía. Golang aplica este concepto en forma del concepto de [Zero values](https://go.dev/tour/basics/12). Por definición:

> Variables declared without an explicit initial value are given their zero value.

<br>
<div align="center">

| Tipo | valor |
|-|-|
| Numérico: `int`, `int64`, `float64`, etc | `0` |
| Booleano: `bool` | `false` |
| String: `string` | `""` |

</div>

Esto aplica también para los tipos cuyo tipo subyacente es alguno de los anteriormente mencionados, por ejemplo:

```go
type CtxKey string

func foo() {
  var k CtxKey

  fmt.Printf("%v", k)
  // Output: ""
}
```

Además, agregando que datos de tipo slice se inicializan como el slice del mismo tipo sin elementos. Todo esto en la práctica significa que:

```go
func main() {
	// será inicializado como:
	var i int       // 0
	var f []float64 // []
	var b bool      // false
	var s string    // ""
	fmt.Printf("%v %v %v %q\n", i, f, b, s)
	// Output: 0 [] false ""
}
```

## Implicaciones

La mejor manera de comprender por qué este comportamiento nos ahorra configuración en favor de la convención, es con un ejemplo. Imaginemos que estamos modelando datos biológicos de animales para un sistema de clasificación científica. No todos los animales comparten las mismas características.

- Algunos tienen colmillos
- Algunos son acuáticos
- Algunos poseen un hábitat específico que vale la pena registrar
- Otros simplemente no aplican a ciertas categorías

En Go, podemos expresar este modelo de forma directa:

```go
type Animal struct {
	Name       string
	IsActuatic bool
	Habitat    string
}
```

Ahora podemos crear instancias parciales sin ningún tipo de inicialización explícita

```go
elephant := Animal{
	Name:     "Elefante africano",
	Habitat:  "Sabana",
}

worm := Animal{
	Name: "Lombriz de tierra",
}
```

Notar que no existen valores inválidos, por ejemplo, si luego queremos comprobar si un animal tiene colmillos o no:

```go
// Nunca declaramos que la lombriz fuera o no acuática
if worm.IsActuatic {
  // Animal actuático
}
```

De la misma forma, si queremos ver el hábitat del animal en un reporte generado:

```go
func Report(a Animal) {
  fmt.Printf("REPORTE\nNombre: %s\nAcuático: %v\nHábitat: %s\n", a.Name, a.IsActuatic, a.Habitat)

  // ...o si queremos omitir irrelevantes

  fmt.Printf("REPORTE\n")

  // siempre imprimir el nombre,
  // si lo dejamos vacío simplemente mostrará:
  // Name:
  fmt.Printf("Nombre: %s\n", a.Name)

  if a.IsActuatic {
    fmt.Printf("Acuático: sí\n")
  }

  if a.Habitat != "" {
    fmt.Printf("Hábitat: %s\n", a.Habitat)
  }
}
```

De esta manera, podemos ver reportes de animales que incluyen exclusivamente los datos que requerimos, esto es útil porque podemos configurar cómo queremos manejar los datos sin necesidad de manejar errores o excepciones. Además, si se requiere manejo explícito de alguna condición se puede manejar sin necesidad de "contaminar" el resto.

```go
// Caso en el que necesitamos definir un valor por defecto
func Report(a Animal) {
  // ...
  if a.Habitat != "" {
    fmt.Printf("Hábitat: desconocido\n")
  }
  // ...
}

// Caso en el que necesitamos obligatoriamente un error,
// pero únicamente en el caso en el que el nombre está vacío
func Report(a Animal) error {
  // ...
  if a.Name != "" {
    return fmt.Errorf("empty name")
  }
  // ...
}
```

### Ejemplo en Python

Veamos la contraparte en un lenguaje popularmente utilizado por ser sencillo

```py
class Animal:
    def __init__(
        self,
        name,
        is_acuatic=False,
        habitat=None,
    ):
        self.name = name
        self.is_acuatic = is_acuatic
        self.habitat = habitat

# ...

elephant = Animal(
    name="Elephant",
    habitat="Savanna",
)

worm = Animal(
    name="Worm",
)

# ...

if animal.habitat is None:
    habitat = "desconocido"
```

A pesar de que:

- Debimos declarar explícitamente valores por defecto
- Los consumidores del objeto deben comprobar None
- Mayor verbosidad y comprobaciones adicionales en comparación con Go

...todavía mantiene la ventaja de que puestos estos rodines es difícil hasta crear métodos que vayan a resultar en exceptiones o en manejo innecesario de errores.

### Similarmente en C++

```cpp
#include <iostream>
#include <string>

struct Animal {
    std::string name;
    bool isAcuatic = false; // Valor por defecto explícito
    std::string habitat = ""; // Valor por defecto explícito
};

int main() {
    Animal elephant{"Elephant", false, "Savanna"};
    Animal worm{"Worm"}; // isAcuatic=false, habitat=""

    std::cout << "Elephant: " << elephant.name
              << ", Acuático: " << elephant.isAcuatic
              << ", Hábitat: " << elephant.habitat << std::endl;

    std::cout << "Worm: " << worm.name
              << ", Acuático: " << worm.isAcuatic
              << ", Hábitat: " << worm.habitat << std::endl;

    return 0;
}
```

A pesar de que, sin inicialización explícita, bool y std::string podrían contener basura (comportamiento indefinido), podemos ver una clara ventaja en comparación con otras implementaciones

## Una mala implementación

```cpp
#include <iostream>
#include <string>
#include <stdexcept>

struct Animal {
    std::string name;
    bool isAcuatic;
    std::string habitat;

    // Constructor "estricto": falla si cualquier campo es inválido
    Animal(const std::string& n, bool acuatic, const std::string& h) {
        if (n.empty()) {
            throw std::invalid_argument("Name cannot be empty");
        }
        if (h.empty()) {
            throw std::invalid_argument("Habitat cannot be empty");
        }
        name = n;
        isAcuatic = acuatic;
        habitat = h;
    }
};

int main() {
    try {
        Animal worm("", false, ""); // lanza excepción
    } catch (const std::exception& e) {
        std::cout << "Error creating animal: " << e.what() << std::endl;
    }

    try {
        Animal elephant("Elephant", false, "Savanna"); // funciona
    } catch (const std::exception& e) {
        std::cout << e.what() << std::endl;
    }

    return 0;
}
```

Problemas de esta implementación

- CADA VEZ que se crea un objeto hay que manejar try/catch
- Esto contamina todo el flujo de creación de datos con manejo de errores innecesario
- Menos expresivo y más verboso
- No es obvio a simple vista qué ramas podrían o no fallar, donde usando zero values (o valores por defecto en su ausencia) podemos simplemente omitir el manejo de errores

Si en el futuro se agregan nuevos campos, cada constructor y llamada debe actualizarse. Mientras que en la implementación previa en Go basta con agregar a la estructura de datos el atributo deseado, por ejemplo, agregando `shell_hardness float64` a la definición del struct, y por defecto todas las instancias de `Animal` tendrán dureza de caparazón 0, la cual se puede agregar a instancias previas o nuevas de las instancias de este tipo.
