# Part3-Exercise3


### Overall Strcuture and Changes and Justifications

##### Shifting from Object[] to a Shape Interface

The old code stored shape data in a generic Object[] array, which made the code messy and hard to maintain. We had to do a lot of casting ((Point) ps[i]) and used a big switch statement to figure out how to calculate area or boundaries.

In the new approach, I created a Shape interface with the methods area() and boundaries(). Each shape (triangle, quadrilateral, circle) knows how to compute its own area and boundaries. This avoids big switch statements and manual casting. It also uses polymorphism: the main code can call shape.area() regardless of whether shape is a triangle, quadrilateral or circle. 

**Benefit**: Each shape is now responsible for its own logic, and we can add more shapes by simply writing a new class that implements Shape. No need to edit the rest of the code.

```java
public interface Shape {
    double area();
    Point[] boundaries();
}
```

##### readShape() Method

The original code used repeated readTypes, readPoints and then stored them into Object[]. Addtionaly, reading input and constructing shapes was mixed with the rest of the logic.

Now we only do these steps once in a dedicated method. We also don’t store points into an Object[] since we directly create a shape object of the right type. 

**Benefit**: The code is cleaner. If we add a new shape we just add a new branch in switch (type)

```java
static Shape readShape()
```

##### Individual Shape Classes

###### Triangle Class
```java
public static class Triangle implements Shape {
    private final Point p1, p2, p3;

    public Triangle(Point p1, Point p2, Point p3) {
        this.p1 = p1;
        this.p2 = p2;
        this.p3 = p3;
    }

    @Override
    public double area() {
        ...
    }

    @Override
    public Point[] boundaries() {
        ...
    }
}
```
1.  store three points (p1, p2, p3) for the triangle
2.  area() calculates the bounding box area, then takes half
3.  boundaries() returns two corners: the maximum corner (top-left) and the minimum corner (bottom-right)


###### Quadrilateral Class
```java
public static class Quadrilateral implements Shape {
    private final Point p1, p2;
    
    public Quadrilateral(Point p1, Point p2) {
            this.p1 = p1;
            this.p2 = p2;
    }

     @Override
        public double area(){
        ...
    }

     @Override
        public Point[] boundaries(){
        ...
    }
}
```
1. store two points to define this quadrilateral’s bounding box
2. area() calculates (width * height) based on the differences between p1 and p2
3. boundaries() returns the top-left and bottom-right corners


###### Circle Class
```java
public static class Circle implements Shape {
    private final Point center, perimeter;

    public Circle(Point center, Point perimeter) {
            this.center = center;
            this.perimeter = perimeter;
    }


    @Override
    public double area(){
        ...
    }

     @Override
        public Point[] boundaries(){
        ...
    }
}
```
1. store a center and one perimeter point.
2. area() uses pie * r^2
3. boundaries() returns the bounding box of the circle (leftmost, rightmost, topmost, bottommost points)

In the old code, a single function area(Object[] ps) was used for all the shape ccalculations. Similarly, boundaries(Object[] ps) also tried to handle every shape in one place.  

In my approach each shape has its own class, own fields, and own formula for area() and boundaries(), which is simpler to read and easier to develope if we want to add new shapes in the future.


##### Main Method

```java
public static void main() throws Exception {
    ...
    Shape s1 = readShape();
    Shape s2 = readShape();
    ...
}
```
We read two shapes from the user by calling readShape() twice. Moreover, this can easily be extended to N shapes if we want.

```java
List<Shape> shapes = new ArrayList<>();
if (s1 != null) shapes.add(s1);
if (s2 != null) shapes.add(s2);
```
In case the user typed an unknown shape, readShape() can return null, and we check this before adding to the list.


```java
double totalArea = 0.0;
for (Shape sh : shapes) {
    totalArea += sh.area();
}
System.out.printf("Sum of area covered by the shapes:\n%f\n\n", totalArea);
```
Each shape calculates its own area and we just loop over them and add up the results.

##### Combining Boundaries

```java
int xMax = Integer.MIN_VALUE;
int yMax = Integer.MIN_VALUE;
int xMin = Integer.MAX_VALUE;
int yMin = Integer.MAX_VALUE;

for (Shape sh : shapes) {
    Point[] b = sh.boundaries();
    Point topLeft = b[0];
    Point bottomRight = b[1];

    xMax = Math.max(xMax, topLeft.x);
    yMax = Math.max(yMax, topLeft.y);
    xMin = Math.min(xMin, bottomRight.x);
    yMin = Math.min(yMin, bottomRight.y);
}
```
Just like the original code, we merge bounding boxes. However, each shape provides its boundaries with a single call to sh.boundaries(). Additionaly, we no longer manually check what shape it is or do a big if-else. Everything is handled by the shape classes themselves.


### Full code
```java

package fi.utu.tech.exercise3;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class Exercise3 {
    public Exercise3() {
        System.out.println("Exercise 3");
    }

    
    // record for storing coordinates
    public record Point(int x, int y) {}

    //define two methods for any shape: area and boundaries
    public interface Shape {
        double area();
        Point[] boundaries();
    }


    // read user input and create the corresponding shape
    static Shape readShape() {
        // ask the user what shape they wan
        System.out.print("Enter the pattern type (triangle, quadrilateral, circle): ");
        String type = new Scanner(System.in).next();

        // read the first two points 
        System.out.print("Enter the x-coordinate of the point: ");
        int x1 = new Scanner(System.in).nextInt();
        System.out.print("Enter the y-coordinate of the point: ");
        int y1 = new Scanner(System.in).nextInt();
        Point p1 = new Point(x1, y1);

        System.out.print("Enter the x-coordinate of the point: ");
        int x2 = new Scanner(System.in).nextInt();
        System.out.print("Enter the y-coordinate of the point: ");
        int y2 = new Scanner(System.in).nextInt();
        Point p2 = new Point(x2, y2);

        switch (type) {
            case "triangle" -> {
                // need a third point for a triangle
                System.out.print("Enter the x-coordinate of the point: ");
                int x3 = new Scanner(System.in).nextInt();
                System.out.print("Enter the y-coordinate of the point: ");
                int y3 = new Scanner(System.in).nextInt();
                Point p3 = new Point(x3, y3);
                return new Triangle(p1, p2, p3);
            }
            case "quadrilateral" -> {
                return new Quadrilateral(p1, p2);
            }
            case "circle" -> {
                return new Circle(p1, p2);
            }
            default -> {
                System.out.println("Unknown shape type: " + type);
                return null;
            }
        }
    }

    // Triangle shape
    public static class Triangle implements Shape {
        private final Point p1, p2, p3;

        // store three points
        public Triangle(Point p1, Point p2, Point p3) {
            this.p1 = p1;
            this.p2 = p2;
            this.p3 = p3;
        }

        @Override
        public double area() {

            int xMax = Math.max(p1.x, Math.max(p2.x, p3.x));
            int xMin = Math.min(p1.x, Math.min(p2.x, p3.x));
            int yMax = Math.max(p1.y, Math.max(p2.y, p3.y));
            int yMin = Math.min(p1.y, Math.min(p2.y, p3.y));

            return (double)( (xMax - xMin) * (yMax - yMin) ) / 2.0;
        }

        @Override
        public Point[] boundaries() {

            int xMax = Math.max(p1.x, Math.max(p2.x, p3.x));
            int xMin = Math.min(p1.x, Math.min(p2.x, p3.x));
            int yMax = Math.max(p1.y, Math.max(p2.y, p3.y));
            int yMin = Math.min(p1.y, Math.min(p2.y, p3.y));

            return new Point[] { new Point(xMax, yMax), new Point(xMin, yMin) };
        }
    }

    //Quadrilateral shape
    public static class Quadrilateral implements Shape {
        private final Point p1, p2;

        //store two points 
        public Quadrilateral(Point p1, Point p2) {
            this.p1 = p1;
            this.p2 = p2;
        }

        @Override
        public double area() {

            int xMax = Math.max(p1.x, p2.x);
            int xMin = Math.min(p1.x, p2.x);
            int yMax = Math.max(p1.y, p2.y);
            int yMin = Math.min(p1.y, p2.y);

            return (xMax - xMin) * (yMax - yMin);
        }

        @Override
        public Point[] boundaries() {
            int xMax = Math.max(p1.x, p2.x);
            int xMin = Math.min(p1.x, p2.x);
            int yMax = Math.max(p1.y, p2.y);
            int yMin = Math.min(p1.y, p2.y);
            return new Point[] { new Point(xMax, yMax), new Point(xMin, yMin) };
        }
    }


    //Circle shape
    public static class Circle implements Shape {
        private final Point center, perimeter;
        //store a center and one perimeter pint
        public Circle(Point center, Point perimeter) {
            this.center = center;
            this.perimeter = perimeter;
        }

        @Override
        public double area() {

            int dx = perimeter.x - center.x;
            int dy = perimeter.y - center.y;
            double r2 = dx*dx + dy*dy; 
            return Math.PI * r2;
        }

        @Override
        public Point[] boundaries() {

            int dx = perimeter.x - center.x;
            int dy = perimeter.y - center.y;

            int xMax = center.x + Math.abs(dx);
            int xMin = center.x - Math.abs(dx);
            int yMax = center.y + Math.abs(dy);
            int yMin = center.y - Math.abs(dy);

            return new Point[] { new Point(xMax, yMax), new Point(xMin, yMin) };
        }
    }

    public static void main() throws Exception {

        // read two shapes
        System.out.println("A circle is defined by a centre and a perimeter point, the others by corner points");
        Shape s1 = readShape();
        Shape s2 = readShape();

        // store the shapes
        // if user typed unknown shapes, we may get null
        List<Shape> shapes = new ArrayList<>();
        if (s1 != null) shapes.add(s1);
        if (s2 != null) shapes.add(s2);

        // sum area
        double totalArea = 0.0;
        for (Shape sh : shapes) {
            totalArea += sh.area();
        }
        System.out.printf("Sum of area covered by the patterns:\n%f\n\n", totalArea);

        int xMax = Integer.MIN_VALUE;
        int yMax = Integer.MIN_VALUE;
        int xMin = Integer.MAX_VALUE;
        int yMin = Integer.MAX_VALUE;

        for (Shape sh : shapes) {
            Point[] b = sh.boundaries();
            Point topLeft = b[0];
            Point bottomRight = b[1];

            xMax = Math.max(xMax, topLeft.x);
            yMax = Math.max(yMax, topLeft.y);
            xMin = Math.min(xMin, bottomRight.x);
            yMin = Math.min(yMin, bottomRight.y);
        }

        assert(xMax >= xMin);
        assert(yMax >= yMin);

        System.out.printf("The common boundaries of the shapes:\n(%d, %d) x (%d, %d)\n\n",
                xMin, yMin, xMax, yMax);
    }
}


```



