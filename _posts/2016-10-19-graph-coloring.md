---
layout: post
title: "Graph Coloring"
date: 2016-10-19
---

There is nothing more I love in computer science than leveraging theoretical concepts for practical applications. Data structures like trees and stacks are often used in the industry, but there isn't too much discussion of them. Recently in CS 311 (Discrete Mathematics), we covered graph theory. Many objects in the real-world can be modeled as graphs, from social networks to even people shaking hands at a party. Within graph theory, there are a lot of fields of study, but one particular topic caught my attention - graph coloring. The name itself was peculiar, but the underlying concept was truly fascinating.

Graph coloring, as the name suggests, is the process of assigning colors to the vertices in a graph such that no two adjacent vertices share the same color. The chromatic number ($\chi{G}$) of a graph $G$ is the least amount of colors needed to validly color its vertices.

For instance, a complete graph $K_n$ on $n$ vertices has a chromatic number of $n$ because all pairs of vertices $u$ and $v$ have an edge incident to them. However, most graphs are not complete graphs, and so $\chi{G} \le n$.

Graph coloring has some interesting applications. Consider a scenario where a university has to schedule final exams such that no exams clash with each other. If a student is taking two classes and both classes have conflicting timings, this is a problem. Therefore, given a finite amount of final exam times, the university must resolve this problem using the least amount of times.

Each final exam can be a single vertex in a graph. If a student is in two classes, then an edge incident to two vertices will represent this fact. The graph below will model our scenario. There will be five CS classes: CS 1, CS 2, CS 3, CS 4, and CS 5. Using student data, the university has determined that CS 1 and CS 2, CS 1 and CS 3, CS 2 and CS 3, CS 2 and CS 4, CS 3 and CS 4, and CS 4 and CS 5 have students taking both of them.

![](http://i.imgur.com/W80cmQu.png)

Now, we must use an algorithm to color the graph. There are many algorithms out there for coloring graphs, some better than others. However, graph coloring itself is a very challenging problem because there can be exponential possibilities for unique colorings for a very large graph. Of course, this algorithm might not take too much time for a university to schedule its exams, but the time complexity is around $O(n!)$.

I wrote a small algorithm to color a graph in Java. The pseudocode is outlined below if you don't want to take a look at the actual code.

```text
for each vertex in graph:
  if neighbors have no color:
    assign vertex a color
    add color to list
  else:
    get a copy of the color list
    filter with colors neighbors have
    if there are possible colors:
      randomly assign color to vertex
    else:
      generate a new color
      add the new color to list
      assign color to vertex
```

There are a lot of algorithms to do this - some start with the vertices of the highest degree and others group the vertices in some unique manner. This algorithm definitely works for small graphs, but I doubt it will be useful for larger ones. If you're interested, check out the implementation of this algorithm in Java.

`Vertex.java`

```java
public class Vertex<E> {

    private E data;
    private int color;

    public Vertex(E data) {
        this.data = data;
        this.color = 0;
    }

    public E getData() {
        return this.data;
    }

    public void setColor(int color) {
        this.color = color;
    }

    public int getColor() {
        return this.color;
    }

    @Override
    public boolean equals(Object o) {
        if (o == null || !(o instanceof Vertex)) {
            return false;
        }

        Vertex<?> other = (Vertex<?>) o;

        return this.data.equals(other.data) &&
                this.color == other.color;
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append("V(");
        sb.append(this.data.toString());
        sb.append(", ");
        sb.append(this.color);
        sb.append(")");
        return sb.toString();
    }
}
```

`Edge.java`

```java
public class Edge<E> {

    private Vertex<E> v1;
    private Vertex<E> v2;

    public Edge(Vertex<E> v1, Vertex<E> v2) {
        this.v1 = v1;
        this.v2 = v2;
    }

    public Vertex<E> getFirstVertex() {
        return this.v1;
    }

    public Vertex<E> getSecondVertex() {
        return this.v2;
    }
}
```

`Graph.java`

```java
import java.util.Map;
import java.util.HashMap;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.Random;

public class Graph<E> {

    private Map<Vertex<E>, ArrayList<Vertex<E>>> map;

    public Graph(ArrayList<Edge<E>> edges) {
        map = new HashMap<Vertex<E>, ArrayList<Vertex<E>>>();

        Iterator<Edge<E>> it = edges.iterator();
        while (it.hasNext()) {
            Edge<E> edge = it.next();
            Vertex<E> v1 = edge.getFirstVertex();
            Vertex<E> v2 = edge.getSecondVertex();

            if (!this.map.containsKey(v1)) {
                this.map.put(v1, new ArrayList<Vertex<E>>());
            }

            if (!this.map.containsKey(v2)) {
                this.map.put(v2, new ArrayList<Vertex<E>>());
            }
        }

        it = edges.iterator();
        while (it.hasNext()) {
            Edge<E> edge = it.next();
            Vertex<E> v1 = edge.getFirstVertex();
            Vertex<E> v2 = edge.getSecondVertex();

            this.map.get(v1).add(v2);
            this.map.get(v2).add(v1);
        }
    }

    public ArrayList<Vertex<E>> getVertices() {
        ArrayList<Vertex<E>> vertices = new ArrayList<Vertex<E>>();
        Iterator<Vertex<E>> it = this.map.keySet().iterator();

        while (it.hasNext()) {
            Vertex<E> vertex = it.next();
            vertices.add(vertex);
        }

        return vertices;
    }

    public void color() {
        ArrayList<Integer> colors = new ArrayList<Integer>();
        Iterator<Vertex<E>> it = this.map.keySet().iterator();
        Random random = new Random();

        while (it.hasNext()) {
            Vertex<E> vertex = it.next();
            ArrayList<Vertex<E>> neighbors = this.map.get(vertex);

            boolean neighborsColored = false;
            for (Vertex<E> neighbor : neighbors) {
                if (neighbor.getColor() != 0) neighborsColored = true;
            }

            if (!neighborsColored) {
                if (colors.size() == 0) {
                    colors.add(1);
                    vertex.setColor(colors.get(0));
                } else {
                    int colorIndex = random.nextInt(colors.size());
                    int color = colors.get(colorIndex);
                    vertex.setColor(color);
                }

            } else {
                ArrayList<Integer> poss = new ArrayList<Integer>();
                for (Integer color : colors) poss.add(color);

                for (Vertex<E> neighbor : neighbors) {
                    int color = neighbor.getColor();
                    if (color != 0 && colors.contains(color)) {
                        poss.remove(new Integer(color));
                    }
                }

                if (poss.size() != 0) {
                    int colorIndex = random.nextInt(poss.size());
                    int color = poss.get(colorIndex);
                    vertex.setColor(color);
                } else {
                    int newColor = colors.get(colors.size() - 1) + 1;
                    colors.add(newColor);
                    vertex.setColor(newColor);
                }
            }
        }

    }

    @Override
    public String toString() {
        return this.map.toString();
    }
}
```

`Main.java`

```java
import java.util.ArrayList;

public class Coloring {

    public static void main(String[] args) {
        Vertex<Integer> v1 = new Vertex<Integer>(1);
        Vertex<Integer> v2 = new Vertex<Integer>(2);
        Vertex<Integer> v3 = new Vertex<Integer>(3);
        Vertex<Integer> v4 = new Vertex<Integer>(4);
        Vertex<Integer> v5 = new Vertex<Integer>(5);

        Edge<Integer> e1 = new Edge<Integer>(v1, v2);
        Edge<Integer> e2 = new Edge<Integer>(v1, v3);
        Edge<Integer> e3 = new Edge<Integer>(v2, v3);
        Edge<Integer> e4 = new Edge<Integer>(v2, v4);
        Edge<Integer> e5 = new Edge<Integer>(v3, v4);
        Edge<Integer> e6 = new Edge<Integer>(v4, v5);

        ArrayList<Edge<Integer>> edges = new ArrayList<Edge<Integer>>();
        edges.add(e1);
        edges.add(e2);
        edges.add(e3);
        edges.add(e4);
        edges.add(e5);
        edges.add(e6);

        Graph<Integer> graph = new Graph<Integer>(edges);
        graph.color();

        for (Vertex<Integer> vertex : graph.getVertices()) {
            System.out.println(vertex);
        }
    }
}
```

Let's color the graph! I created `Vertex` and `Edge` objects in the driver class and called the `color` method on the graph, which colors the graph. The output is as follows:

```java
V(1, 1)
V(4, 1)
V(3, 2)
V(5, 2)
V(2, 3)
```

This means that color 1 is assigned to CS 1 and CS 4, color 2 is assigned to CS 3 and CS 5, and color 3 is assigned to CS 2. Graphically, the colored graph looks like this:

![](http://i.imgur.com/1jYNarp.png)

If each color represents some time slot, then the final exams for these five CS classes will not conflict. That wraps up graph coloring - if I come up with any better algorithms in the future, I'll make sure to share them!
