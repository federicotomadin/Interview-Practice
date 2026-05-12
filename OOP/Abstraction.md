# Abstraction 

This is the concept of bundling data \(attributes\) and methods \(functions\) that operate on the data into a single unit or class. Encapsulation restricts direct access to some of an object’s components, which means the internal state of an object can only be changed through its methods, ensuring controlled access and maintaining integrity.

```typescript
class Car {
  private speed: number; // private property

  constructor(speed: number) {
    this.speed = speed; // encapsulated data
  }

  // Public method to access private data
  public accelerate(increment: number): void {
    this.speed += increment;
    console.log(`The car accelerates to ${this.speed} km/h.`);
  }

  // Public method to get the speed
  public getSpeed(): number {
    return this.speed;
  }
}

const myCar = new Car(50);
myCar.accelerate(20); // The car accelerates to 70 km/h.
console.log(myCar.getSpeed()); // 70
// myCar.speed = 10
