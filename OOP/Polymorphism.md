# Polymorphism 

Polymorphism allows methods or functions to process objects differently based on their class or data type. 
It enables a single function or method to operate on different types of objects or perform different tasks. 
There are two main types of polymorphism: compile-time (method overloading) and runtime (method overriding).


```typescript
class Shape {
  area(): number {
    return 0;
  }
}

class Circle extends Shape {
  constructor(public radius: number) {
    super();
  }

  // Override the area method
  area(): number {
    return Math.PI * this.radius * this.radius;
  }
}

class Rectangle extends Shape {
  constructor(public width: number, public height: number) {
    super();
  }

  // Override the area method
  area(): number {
    return this.width * this.height;
  }
}

const shapes: Shape[] = [new Circle(5), new Rectangle(10, 5)];

shapes.forEach(shape => {
  console.log(`Area: ${shape.area()}`);
});
// Output:
// Area: 78.53981633974483 (Circle)
// Area: 50 (Rectangle)
