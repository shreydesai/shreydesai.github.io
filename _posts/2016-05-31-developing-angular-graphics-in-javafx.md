---
layout: post
title: "Developing Angular Graphics in JavaFX"
date: 2016-05-31
---

For my AP Computer Science final project, my partner and I developed a simple one player, arcade-shooter game called [Dots](https://github.com/shreydesai/dots) in Java on top of the JavaFX GUI framework. The project required a lot of GUI techniques, multithreading and concurrency knowledge, and surprisingly, a lot of trigonometry and algebra. One of the math heavy components of the project was the angular motion of the bullets - the object that hit the enemies to destroy them.

<img src="http://i.imgur.com/kCIVVys.png" width="50%">

We wanted the bullet to start from the player and move in the direction of wherever the user clicked on the screen, so it seemed like the bullet was traveling towards the mouse.

## Determining the path

As stated above, the bullet had to take a certain path. The `Circle` object that represented the player had $x$ and $y$ coordinates and the mouse (when pressed) also logged $x$ and $y$ coordinates.

<img src="http://i.imgur.com/GPFBpsm.png" width="50%">

The bullet was a constant $C$ away from the player and the hope was that at some interval $i$, the bullet would move in this determined path. Because we were no longer dealing with linear movement and instead two-dimensional movement, trigonometry and vectors suddenly became more relevant.

## Calculating $\theta$

In order to find the initial coordinates of the bullet, we used similar triangles in order to find the angle between the circle and mouse element. This angle would then be used to calculate the vector components of the bullet and the coordinates would be subsequently found.

<img src="http://i.imgur.com/hHqaVHR.png" width="50%">

Three lines were drawn - one from $cx$ to $px$, one from $cy$ to $py$, and one from the center of the bullet element perpendicular to the line below it. In order to determine the vector components of the smaller triangle, $\theta$ had to be calculated with respect to the larger horizontal and vertical components.

Many web and GUI coordinate systems are different from the traditional Cartesian coordinate system in that $(0,0)$ starts from the top right, with the positive $y$ direction going south and the positive $x$ direction going east. This is very important for the calculation of the components because otherwise it simply will not work.

The large horizontal component was the difference between the ending and starting $x$ coordinate, or $px - cx$. The large vertical component was the same with the $y$ coordinate, or $cy - py$.

The angle could be found with:

$$\theta = tan^{-1}(\frac{y}{x})$$

Substituting the horizontal and vertical components would yield:

$$\theta = tan^{-1}(\frac{px - cx}{cy - py})$$

This angle could then be used to determine the vector components of the smaller triangle.

## Finding the coordinates

Because $\theta$ was known this time, the components of the bullet vector could be derived from the angle with the $sin(x)$ and $cos(x)$ functions. Given that the length between the player and bullet was some constant $C$, the horizontal component would be $C \times cos\theta$ and the vertical component would be $C \times sin\theta$.

<img src="http://i.imgur.com/euDJ3fF.png" width="50%">

Finally, given that the coordinates of the initial position of the bullet was $(nx,ny)$ and the vector components were known, $nx$ and $ny$ could be calculated. Note that the JavaFX coordinate system would need to be taken into consideration here.

$$nx = cx + C \times cos\theta$$
$$ny = cy - C \times sin\theta$$

## Conclusion

That simple equation above took a lot of drawing on whiteboards and head banging, but I'm glad we figured it out. The end product looked super exciting and it definitely looked better than a black line solely moving horizontally. I also brushed up some of my vector algebra, which was also a plus. The method for the equation looked much cleaner and it was well integrated with the rest of the code.

```java
public Line getLine() {
    double cx = coordinates[0];
    double cy = coordinates[1];
    double px = coordinates[2];
    double py = coordinates[3];
    double componentX, componentY;

    right = px > cx;

    if (right) {
        componentX = px - cx;
        componentY = cy - py;
        theta = Math.atan(componentY / componentX);

        return new Line(
            x, cy,
            cx + 10 * Math.cos(theta),
            cy - 10 * Math.sin(theta)
        );
    } else {
        componentX = cx - px;
        componentY = py - cy;
        theta = Math.atan(componentY / componentX);
        return new Line(
            cx, cy,
            cx - 10 * Math.cos(theta),
            cy + 10 * Math.sin(theta)
        );
    }
}
```
