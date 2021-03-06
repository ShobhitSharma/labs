### Topics

**Data Types**

- stack
- queue
- bag
- union find
- priority queue

**Sorting**

- quicksort
- mergesort
- heapsort
- radix sorts

**Searching**
- BST
- red-black BST
- hash table

**Graphs**

- BFS
- DFS
- Orun
- Kruskal
- Dijkstra

**Strings**

- KMP
- regular expressions
- TST
- Huffman
- LZW

**Advanced**

- B-tree
- suffix array
- maxflow


## Why Study Algorithms

- Intellectual stimulation
- Solve problems
- Fun and profit
- etc

## Prerequisites

- Know some programming
- Knowledge of Java
- High school math


## Dynamic Connectivity

### Steps to solve

- Model the problem
- Find and algo to solve it
- Fast enough? Fits in mem?
- If not, figure out why
- Find a way to address the problem
- Iterate until satisfied

### Problem

Given a set of N objects

- Union command: connect to objects
- Find/connected query: is there a path connecting two objects

![](http://i.imgur.com/y3YfA2c.jpg)

### Applications

- Pixels
- Networking
- Social network
- etc

### Modelling the relation

We assume 'is connected to' is an equivalence relation

- Reflexive: *p* is connected to *p*
- Symmetric: if *p* is connected to *q*, then *q* is connected to *p*
- Transistive: if *p* is connected to *q* and *q* is connected to *r*, then *p* is connected to *r*

### Goal

Design efficient data structure for union-find

- Number of objects N can be huge
- Number of operations M can be huge
- Find queries and union commands may be intermixed

### Dynamic-connectivity client

- Read in number of objects N from stdin
- Repeat
  - read in pair of integers from stdin
  - if they are not yet connected, connect them and print out pair

**input.txt**

```
10
4 3
3 8
6 5
9 4
2 1
8 9
5 0
7 2
6 1
1 0
6 7
```

**reader.java**
```java
public static void main(String[] args)
{
    int N = StdIn.readInt();
    UF uf = new UF(N);
    while(!StdIn.isEmpty())
    {
        int p = StdIn.readInt();
        int q = StdIn.readInt();
        if(!uf.connected(p, q))
        {
            uf.union(p, q);
            StdOut.println(p + ' ' + q);
        }
    }
}
```


### Question

**How many connected components result after perfoming the following sequence:**
```
1-2  3-4  5-6  7-8  7-9  2-8  0-5  1-9
```
**Answer:** 3


## Quick Find

### Data Structure

- integer array `id[]` of size N
- interpretation: p and q are connected iff they are have the same id

**Find**

Check if p and q have the same id

**Union**

To merge components containing p and q, change all entries whose id equals `id[p]` to `id[q]`


```java
public class QuickFindUF
{
  private id[] id;

  public QuickFindUF(int N)
  {
     id = new int[N];
     for(int i=0; i<N; i++)
     {
      id[i] = i;
     }
  }

  public boolean connected(int p, int q)
  {
      return id[p] == id[q];
  }

  public void union(int p, int q)
  {
      int pid = id[p];
      int qid = id[q];
      for(int i=0; i<id.length; i++)
      {
        if(id[i] == pid)
        {
          id[i] = qid;
        }
      }
  }

}
```

**Cost Model**

Defect: Union too expensive O(N^2)

### Quiz

**What is the maximum number of `id[]` array entries that can change during one call to union when using the quick-find data structure on N elements**

**Answer:** N-1 (In worst case, all entries except id[p] are set to id[p])


## Quick Union

### Data Structure (forest)

- Integer array id[] of size N
- Interpretation: id[i] is parent of i
- Root of i is id[id[id[...]]]

Find: Check if p and q have the same root

Union: To merge components containing p and q set the id of p's to the id of  q's root


### Question

**Suppose that in a quick-union data structure on 10 elements that the id[] array is listed below. What are the roots of 3 and 7?**

```
0 9 6 5 4 2 6 1 0 5
```

**Answer:** 6 and 6
```
3->5->2->6
7->1->9->5->2->6
```

**Quick Union**
```java
public class QuickFindUF
{
  private id[] id;

  public QuickFindUF(int N)
  {
     id = new int[N];
     for(int i=0; i<N; i++)
     {
      id[i] = i;
     }
  }

  private int root(int i)
  {
    while(i != id[i])
    {
      i = id[i];
    }
    return i;
  }

  public boolean connected(int p, int q)
  {
    return root(p) == root(q);
  }

  public void union(int p, int q)
  {
    int i = root(p);
    int j = root(q);
    id[i] = j;
  }

}
```

Defect: Trees can get tall. Find too expensive O(N)

### Quiz

**What is the maximum number of array accesses during a find operation when using the quick-union data structure on N elements?**

**Answer:** Linear

## Quick Union Improvements

### Weighted quick-union

- Modify quick-union to avoid tall trees
- Keep track of size of each tree
- Balance by linking of smaller tree to root of larger tree

```java
public class QuickFindUF
{
  private id[] id;
  public sz[] id;

  public QuickFindUF(int N)
  {
     id = new int[N];
     for(int i=0; i<N; i++)
     {
      id[i] = i;
     }
  }

  private int root(int i)
  {
    while(i != id[i])
    {
      i = id[i];
    }
    return i;
  }

  public boolean connected(int p, int q)
  {
    return root(p) == root(q);
  }

  public void union(int p, int q)
  {
    int i = root(p);
    int j = root(q);
    if (sz[i] < sz[j])
    {
      id[i] = j;
      sz[j] += sz[i];
    }
    else
    {
      id[j] = i;
      sz[i] += sz[j];
    }
  }

}
```
Running time: lg(N)

### Quick union with path compression

Just after computing the root of p, set the id of each examined node to point to that root

```java
public class QuickFindUF
{
  private id[] id;
  public sz[] id;

  public QuickFindUF(int N)
  {
     id = new int[N];
     for(int i=0; i<N; i++)
     {
      id[i] = i;
     }
  }

  private int root(int i)
  {
    while(i != id[i])
    {
      id[i] = id[id[i]]; // flatten the tree
      i = id[i];
    }
    return i;
  }

  public boolean connected(int p, int q)
  {
    return root(p) == root(q);
  }

  public void union(int p, int q)
  {
    int i = root(p);
    int j = root(q);
    if (sz[i] < sz[j])
    {
      id[i] = j;
      sz[j] += sz[i];
    }
    else
    {
      id[j] = i;
      sz[i] += sz[j];
    }
  }

}
```

Runtime: O(N+Mlg*N) where lg* is the number of times you need to take the lg to get 1 (~5)

### Question

**Suppose that i[] array during the weighted quick union algortihm is listed below. Which id[] entry changes when we apply the union operation to 3 and 6?**

![](http://i49.tinypic.com/308uz9x.png)

**Answer:** id[8]

## Applications

- Percolation
- Dynamic Connectivity
- Least common ancestor
- Equivalence of finite state automata
- Matlab's bwlabel() function in image processing
- etc

## Percolation

A model of many physical systems

- N by N grid of sets
- Each site is open with probability p or blocked 1 - p
- System percolates iff top and botom are connected by open sites

![](http://i.imgur.com/wzvRqhD.png)

###Percolation phase model

When N is large, theory guarantees a sharp threshold p*
- p > p*: almost certainly percolates
- p < p* almost certainly does not percolate

What is the value of p*?
~0.592746


### Monte Carlo Simulation

- Initialize by N by N whole grid to be blocked
- Declare random sites open util top connected to bottom
- Vacancy percentage estimates p*

**How to check if N by N system percolates?**

- Create an object for each site and name them 0 to N^2 - 1
- Sites are in same component if connected by open sites
- Percolates iff any site on bottom row is connected to site on top row

Clever Trick: Introduce 2 virtual sites (and connections to top and bottom)
- Percolates iff virtual top site is connected to virtual bottom site

### Question

**When opening one new site in the percolation simulation, how many times is union() called?**

**Answer:** 0, 1, 2, 3 or 4

