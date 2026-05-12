# Inheritance 

Inheritance is a mechanism where a new class inherits the properties and methods of an existing class. It helps in code reusability and establishing a relationship between classes.

```typescript
class Vehicle {
  protected brand: string;

  constructor(brand: string) {
    this.brand = brand;
  }

  public drive(): void {
    console.log(`${this.brand} is driving.`);
  }
}

class Car extends Vehicle {
  private model: string;

  constructor(brand: string, model: string) {
    super(brand); // Call the constructor of the parent class
    this.model = model;
  }

  public displayInfo(): void {
    console.log(`${this.brand} ${this.model}`);
  }
}

const car = new Car('Toyota', 'Corolla');
car.drive();         // Outputs: Toyota is driving.
car.displayInfo();   // Outputs: Toyota Corolla
