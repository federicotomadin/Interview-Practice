### Ejemplo

En la vida real tenemos claro que un elefante es un animal. Imaginemos que tenemos la clase `Animal` que representa un animal, y les damos a los animales la propiedad de andar y saltar:

```typescript
class Animal {
    walk() { /* ... */ }
    jump() { /* ... */ }
}
```

Y tenemos una parte del código donde recibimos un animal, y necesitamos que el animal salte:

```typescript
function jumpHole(a: Animal) {
    a.walk();
    a.jump();
    a.walk();
}
```

Ahora nos creamos un elefante. Pero claro, un elefante no puede saltar, así que decidimos lanzar una excepción para asegurarnos de detectarlos si esto ocurre:

```typescript
class Elephant extends Animal {
    jump() {
        throw new Error("Los elefantes no pueden saltar");
    }
}
```

Ahora en todos los sitios donde estemos usando `jumpHole()`, si el animal es un elefante, tendremos una excepción.

Mal asunto, ¿no?

### ¿Cómo lo solucionamos?

Aquí lo que tenemos que entender es que la abstracción que hemos decidido hacer no es correcta.

Hay animales que no saltan, así que estamos dando por ciertos casos que se pueden volver en nuestra contra.

Por tanto, las clases tienen que representar esos posibles estados inequívocamente, y las funciones usar las abstracciones que necesiten.

¿Qué podríamos hacer en este caso? Plantear un tipo de animal ligero que sí que puede saltar, mientras que damos por hecho que los animales en general no pueden hacerlo:

```typescript
class Animal {
    walk() { /* ... */ }
}

class LightweightAnimal extends Animal {
    jump() { /* ... */ }
}
```

Esto nos permite definir animales que sí pueden saltar y otros que no. Por ejemplo un perro y un elefante:

```typescript
class Dog extends LightweightAnimal { }

class Elephant extends Animal { }
```

Y la función de `jumpHole()` solo admitiría animales que pueden saltar:

```typescript
function jumpHole(a: LightweightAnimal) {
    a.walk();
    a.jump();
    a.walk();
}
```

Elegir las abstracciones correctas muchas veces no es fácil, pero tenemos que intentar limitar al máximo cuál es su alcance para no pedir más de lo que se necesita ni menos.

Esta es la solución que obtendríamos mediante herencia aplicando el Principio de Liskov, pero también se podría haber solucionado mediante composición.

La herencia nos puede generar una jerarquía de clases muy compleja si hay muchos tipos de animales, así que en función del problema hay que plantearse cuál merece la pena usar.

Esta segunda opción es la que veremos con el Principio de segregación de interfaces.
