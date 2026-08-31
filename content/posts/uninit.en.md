---
title: 'Taking Advantage of Uninitialized Values'
date: '2026-01-18'
tags: ['golang']
---

The concept of initialization is practically a dogma, and in many cases
unnecessary. This is not to say that I prefer undefined behavior, but rather
that _[Convention over configuration][convention-config]_ is a powerful
philosophy that saves lines of code without losing control over a program's
behavior.

[convention-config]: https://rubyonrails.org/doctrine

## Zero values

Of course, not every language lends itself to this philosophy. Golang applies
this concept through [zero values](https://go.dev/tour/basics/12). By
definition:

> Variables declared without an explicit initial value are given their zero
> value.

<br>
<div align="center">

| Type | value |
|-|-|
| Numeric: `int`, `int64`, `float64`, etc. | `0` |
| Boolean: `bool` | `false` |
| String: `string` | `""` |

</div>

This also applies to types whose underlying type is one of those mentioned
above, for example:

```go
type CtxKey string

func foo() {
  var k CtxKey

  fmt.Printf("%v", k)
  // Output: ""
}
```

In practice, all this means that:

```go
func main() {
	// will be initialized as:
	var i int       // 0
	var b bool      // false
	var s string    // ""
	fmt.Printf("%v %v %v %q\n", i, f, b, s)
	// Output: 0 [] false ""
}
```

## Implications

The best way to understand why this behavior saves us configuration in favor
of convention is through an example. Imagine that we are modeling biological
data about animals for a scientific classification system. Not all animals
share the same characteristics.

- Some have fangs.
- Some are aquatic.
- Some have a specific habitat worth recording.
- Others simply do not apply to certain categories.

In Go, we can express this model directly:

```go
type Animal struct {
	Name       string
	IsAquatic bool
	Habitat    string
}
```

Now we can create partial instances without any explicit initialization:

```go
elephant := Animal{
	Name:     "African elephant",
	Habitat:  "Savannah",
}

worm := Animal{
	Name: "Earthworm",
}
```

Notice that there are no invalid values; for example, if later we want to
check whether an animal has fangs or not:

```go
// We never declared whether the worm was aquatic or not
if worm.IsAquatic {
  // Aquatic animal
}
```

Likewise, if we want to see the animal's habitat in a generated report:

```go
func Report(a Animal) {
  fmt.Printf("REPORT\nName: %s\nAquatic: %v\nHabitat: %s\n", a.Name, a.IsAquatic, a.Habitat)

  // ...or if we want to omit irrelevant fields

  fmt.Printf("REPORT\n")

  // always print the name,
  // if we leave it empty it will simply display:
  // Name:
  fmt.Printf("Name: %s\n", a.Name)

  if a.IsAquatic {
    fmt.Printf("Aquatic: yes\n")
  }

  if a.Habitat != "" {
    fmt.Printf("Habitat: %s\n", a.Habitat)
  }
}
```

This way, we can see reports that include exclusively the data we need. This
is useful because we can configure how we want to handle the data without
having to deal with errors or exceptions. Also, if explicit handling of a
condition is required, it can be handled without having to "contaminate" the
rest.

```go
// Case where we need to define a default value
func ReportWithDefaults(a Animal) {
  // ...
  if a.Habitat == "" {
    fmt.Printf("Habitat: unknown\n")
  }
  // ...
}

// Case where we absolutely need an error,
// but only when the name is empty
func Validate(a Animal) error {
  // ...
  if a.Name == "" {
    return fmt.Errorf("empty name")
  }
  // ...
}
```

### Example in Python

Let's look at the counterpart in a language that is popularly used because it
is "simple":

```py
class Animal:
    def __init__(
        self,
        name,
        is_aquatic=False,
        habitat=None,
    ):
        self.name = name
        self.is_aquatic = is_aquatic
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

if worm.habitat is None:
    habitat = "unknown"
```

Despite the fact that:

- We had to explicitly declare default values.
- Consumers of the object must check for `None`.
- There is more verbosity and additional checking compared to Go.

...it still has the advantage that, given these constraints, it is difficult
even to create methods that would result in exceptions or unnecessary error
handling.

If new fields are added in the future, every constructor and every call must be
updated. Whereas in the previous Go implementation, it is enough to add the
desired attribute to the data structure; for example, by adding
`ShellHardness float64` to the struct definition, all `Animal` instances will
have a shell hardness of 0 by default, which can be added to existing or new
instances of this type.
