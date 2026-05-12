```typescript
class Vehicle {
    constructor(public wheelCount: number, public maxSpeed: number) {}

    print() {
        console.log(`wheelCount=${this.wheelCount}, maxSpeed=${this.maxSpeed}`);
    }
}
```

Aunque a primera vista puede parecer una clase de lo más razonable, en seguida podemos detectar que estamos mezclando dos conceptos muy diferentes: la lógica de negocio y la lógica de presentación. Este código nos puede dar problemas en muchas situaciones distintas:

- En el caso de que queramos presentar el resultado de distinta manera, necesitamos cambiar una clase que especifica la forma que tienen los datos. Ahora mismo estamos imprimiendo por pantalla, pero imagina que necesitas que se renderice en un HTML. Tanto la estructura (seguramente quieras que la función devuelva el HTML), como la implementación cambiarían completamente.
- Si queremos mostrar el mismo dato de dos formas distintas, no tenemos la opción si sólo tenemos un método `print()`.
- Para testear esta clase, no podemos hacerlo sin los efectos de lado que suponen el imprimir por consola.

Hay casos como este que se ven muy claros, pero muchas veces los detalles serán más sutiles y probablemente no los detectarás a la primera. No tengas miedo de refactorizar lo que haga falta para que se ajuste a lo que necesites.

Una solución muy simple sería crear una clase que se encargue de imprimir:

```typescript
class VehiclePrinter {
    print(vehicle: Vehicle) {
        console.log(`wheelCount=${vehicle.wheelCount}, maxSpeed=${vehicle.maxSpeed}`);
    }
}
```

Si necesitases distintas variaciones para presentar la misma clase de forma diferente (por ejemplo, texto plano y HTML), siempre puedes crear una interfaz y crear implementaciones específicas. Pero ese es un tema diferente.

Otro ejemplo que nos podemos encontrar a menudo es el de objetos a los que les añadimos el método `save()`. Una vez más, la capa de lógica y la de persistencia deberían permanecer separadas. Seguramente hablaremos mucho de esto en futuros artículos.
