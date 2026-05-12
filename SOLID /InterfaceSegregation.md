## ¿Cómo detectar que estamos violando el Principio de segregación de interfaces?

Como comentaba en los párrafos anteriores, si al implementar una interfaz ves que uno o varios de los métodos no tienen sentido y te hace falta dejarlos vacíos o lanzar excepciones, es muy probable que estés violando este principio.

Si la interfaz forma parte de tu código, divídela en varias interfaces que definan comportamientos más específicos.

Recuerda que no pasa nada porque una clase ahora necesite implementar varias interfaces. El punto importante es que use todos los métodos definidos por esas interfaces.

### Ejemplo

Imagina que tienes una tienda de CDs de música, y que tienes modelados tus productos de esta manera:

```typescript
interface Product {
    name: string;
    stock: number;
    numberOfDisks: number;
    releaseDate: number;
}

class CD implements Product {
    name: string;
    stock: number;
    numberOfDisks: number;
    releaseDate: number;

    constructor(name: string, stock: number, numberOfDisks: number, releaseDate: number) {
        this.name = name;
        this.stock = stock;
        this.numberOfDisks = numberOfDisks;
        this.releaseDate = releaseDate;
    }
}
```

El producto tiene una serie de propiedades que nuestra clase `CD` sobrescribirá de algún modo.

Pero ahora has decidido ampliar mercado, y empezar a vender DVDs también.

El problema es que para los DVDs necesitas almacenar también la clasificación por edades, porque tienes que asegurarte de que no vendes películas no adecuadas según la edad del cliente.

Lo más directo sería simplemente añadir la nueva propiedad a la interfaz:

```typescript
interface Product {
    ...
    recommendedAge: number;
}
```

¿Qué ocurre ahora con los CDs? Que se ven obligados a implementar `recommendedAge`, pero no van a saber qué hacer con ello, así que lanzarán una excepción:

```typescript
class CD implements Product {
    ...
    get recommendedAge(): number {
        throw new Error("UnsupportedOperationException");
    }
}
```

Con todos los problemas asociados que hemos visto antes.

Además, se forma una dependencia muy fea, en la que cada vez que añadimos algo a `Product`, nos vemos obligados a modificar `CD` con cosas que no necesita.

Podríamos hacer algo tal que así:

```typescript
interface DVD extends Product {
    recommendedAge: number;
}
```

Y hacer que nuestras clases extiendan de aquí.

Esto solucionaría el problema a corto plazo, pero hay algunas cosas que pueden seguir sin funcionar demasiado bien.

Por ejemplo, si hay otro producto que necesite categorización por edades, necesitaremos repetir parte de esta interfaz.

Además, esto no nos permitiría realizar operaciones comunes a productos que tengan esta característica.

La alternativa es segregar las interfaces, y que cada clase utilice las que necesite. Tendríamos por tanto una interfaz nueva:

```typescript
interface AgeAware {
    recommendedAge: number;
}
```

Y ahora nuestra clase `DVD` implementará las dos interfaces:

```typescript
class CD implements Product {
    ...
}

class DVD implements Product, AgeAware {
    ...
    recommendedAge: number;

    constructor(name: string, stock: number, numberOfDisks: number, releaseDate: number, recommendedAge: number) {
        this.name = name;
        this.stock = stock;
        this.numberOfDisks = numberOfDisks;
        this.releaseDate = releaseDate;
        this.recommendedAge = recommendedAge;
    }
}
```

La ventaja de esta solución es que ahora podemos tener código `AgeAware`, y todas las clases que implementen esta interfaz podrían participar en código común.

Imagina que no vendes sólo productos, sino también actividades, que necesitarían una interfaz diferente.

Estas actividades también podrían implementar la interfaz `AgeAware`, y podríamos tener código como el siguiente, independientemente del tipo de producto o servicio que vendamos:

```typescript
function checkUserCanBuy(user: User, ageAware: AgeAware): boolean {
    return user.age >= ageAware.recommendedAge;
}
```

### ¿Qué hacer con código antiguo?

Si ya tienes código que utiliza fat interfaces, la solución puede ser utilizar el patrón de diseño “Adapter”. El patrón Adapter nos permite convertir unas interfaces en otras, por lo que puedes usar adaptadores que conviertan la interfaz antigua en las nuevas.
