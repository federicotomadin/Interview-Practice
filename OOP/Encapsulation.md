# Encapsulation 

Abstraction is the process of hiding the complex implementation details and showing only the essential features of an object. It allows a programmer to focus on the interactions at a higher level without needing to understand all the complex inner workings.


```typescript

abstract class Animal {
  constructor(public name: string) {}

  // Abstract method (must be implemented by derived classes)
  abstract makeSound(): void;

  move(): void {
    console.log(`${this.name} moves.`);
  }
}

class Dog extends Animal {
  makeSound(): void {
    console.log('Woof! Woof!');
  }
}

const dog = new Dog('Buddy');
dog.makeSound(); // Woof! Woof!
dog.move(); // Buddy moves.
