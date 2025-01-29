# Part3-Exercise3


### Overall Strcuture and Changes and Justifications

##### Shifting from Object[] to a Shape Interface

The old code stored shape data in a generic Object[] array, which made the code messy and hard to maintain. We had to do a lot of casting ((Point) ps[i]) and used a big switch statement to figure out how to calculate area or boundaries.

In the new approach, I created a Shape interface with the methods area() and boundaries(). Each shape (triangle, quadrilateral, circle) knows how to compute its own area and boundaries. This avoids big switch statements and manual casting.

**Benefit**: Each shape is now responsible for its own logic, and we can add more shapes by simply writing a new class that implements Shape. No need to edit the rest of the code.

```java
public interface Shape {
    double area();
    Point[] boundaries();
}
```

##### readShape() Method

The original code used repeated lines that read types, read points, then stored them into Object[].

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
    ...
}
```
1. store two points to define this quadrilateral’s bounding box
2. area() calculates (width * height) based on the differences between p1 and p2
3. boundaries() returns the top-left and bottom-right corners


###### Circle Class
```java
public static class Circle implements Shape {
    private final Point center, perimeter;
    ...
}
```
1. store a center and one perimeter point.
2. area() uses pie * r^2
3. boundaries() returns the bounding box of the circle (leftmost, rightmost, topmost, bottommost points)

In the old code, a single function area(Object[] ps) was used for all the shape ccalculations. Similarly, boundaries(Object[] ps) also tried to handle every shape in one place.  

In my approach each shape has its own class, own fields, and own formula for area() and boundaries(), which is simpler to read and easier to develope if we want to add new shapes in the future.









