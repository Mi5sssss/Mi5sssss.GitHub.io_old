---
layout: post
title: A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits
date: 2021-7-15
categories: Research
tags: [Research, EDA, Algorithm]
description: A quick solution to specific matrix in MCA.
---

`Some technical trouble was found in displaying formula.`

![2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/20210715/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits.6qw7kn29nvc0.png)

# *1* Introduction

​	Existing cross-array simulators, such as MNSIM, mainly skip the SPICE-level circuit simulation and use highly simplified behavioral-level models to simulate the characteristics of memristors and crossbar arrays. It essentially increases the simulation speed by sacrificing accuracy, and thus does not guarantee simulation accuracy and makes it difficult to perform design optimization for complex practical applications. Therefore, the main objective of this essay is to develop dedicated circuit simulation acceleration techniques and tools for the characteristics of **MCA** (Memristor Crossbar Array) using tensor-based technology to improve the simulation efficiency by one order of magnitude, and to realize the design and optimization of memristor neural networks based on circuit simulation. Specifically, we hope to make full use of the special topology of the memristor array and build a corresponding preconditioner based on the tensor to accelerate the core iterative linear system solution process in the simulation process and achieve the optimal balance of simulation speed and accuracy.

# *2* Architecture of MCA Circuit

## *2.1* *Physical MCA Circuit Architecture*

  Fig. 1. showcases a general architecture of MCA. The conductance of each memristor device store a matrix element or weight. Usually the voltage-controlled extended memristor is applied to the array since voltages with changeable input pulse lengths and amplitudes are easy-controlled by source meter, without requirement of high accuracy.

![clip_image002](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image002.5lqmerbw7ac0.gif)

`Fig. 1. A general MCA is shown with BL (bit line) and WL (word line).`

​	A voltage-controlled extended memristor is illustrated by the following equation:

![](https://latex.codecogs.com/svg.image?2&plus;2=3*x*%5Cfrac%7B4%7D%7B2%7D)， ![](https://latex.codecogs.com/svg.image?2&plus;2=3)
$$
i=G\left(x,v\right)v
$$

$$
G\left(x,0\right)\neq\infty
$$

$$
\frac{dx}{dt}=g(x,v)
$$

​	Where the $x$ is the resistance state of each memristor, and $v$ is the voltage that applied to both ends.  $i$ is the current that we can obtain from measurement. Conductance of memristor $G$ is a function related to $x$ and $v$ , which shows a similar property of Ohm’s Law. The state equation $\frac{dx}{dt}$ is a function $g$ connected with $x$ and $v$.

​	In Fig .1., analog input vector values are mapping to the voltage with alterable pulse lengths and amplitudes voltages, and the output vector values are represented as currents.

​	The operation of MCA is divided into two parts, writing process and reading process. As there are no insulation between reading and writing, crosstalk is existed in the circuit, which we would not pay attention to in this article.

## *2.2*    *SPICE Model Build*

​	In a SPICE model, the crossbar array circuits are commonly illustrated by matrix.

​	In the MCA, due to the special characteristics of the circuit, the conductance in the circuit can be reduced to a matrix $G$ in the form of cross point, and since cross point can only form conductance and interact with adjacent cross point, this special characteristic leads to a matrix in the form of a tri-diagonal matrix.

| ![clip_image022](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image022.4lgsjken4q20.gif)  (a) | ![clip_image024](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image024.34wt1x0t4160.gif)  (b) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![clip_image026](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image026.u4zo6aw5te8.gif)  (c) | ![clip_image028](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image028.5xz5ezl7mtw0.gif)  (d) |

Fig. 2. (a) The general MCA with wire resistance concerned, forming $N*N$ dimension crossbar structure. (b) The division of memristor.  (c) A division of row wire resistance, which can be represented as $G_r$. (d) A division of column wire resistance, which can be represented as $G_c$. 



​	We give our model by the following MNA (Modified Nodal Analysis) equation:
$$
G_t*V_t+I_t(V_t,V_b)-Y_t=0
$$

$$
G_b*V_b-I_b(V_t,V_b)-Y_b=0
$$

where the $t$ and $b$ in subscript represented top layer and bottom layer of an MCA. $I_t$ and $I_b$ are function that related to voltage applied to the nodes from bottom and top wire, a set of nonlinear equations. $V_t$ is an array of the top nodal voltages arranged by rows, as well as $V_b$.
$$
V_t=\left[\begin{matrix}v_{t1,1}\\v_{t1,2}\\\vdots\\v_{t1,n-1}\\v_{t1,n}\\v_{t2,1}\\\vdots\\v_{t2,n}\\\vdots\\v_{tn,n}\\\end{matrix}\right],\ V_b=\left[\begin{matrix}v_{b1,1}\\v_{b1,2}\\\vdots\\v_{b1,n-1}\\v_{b1,n}\\v_{b2,1}\\\vdots\\v_{b2,n}\\\vdots\\v_{bn,n}\\\end{matrix}\right]
$$
The $V_{ti,j}$ and $V_{bi,j}$ represent the nodal voltage of the $i^{th}$ row and $j^{th}$ column on the top and bottom grid, respectively.

The G represent the wire conductance matrix in MNA.
$$
G=\left[\begin{matrix}\begin{matrix}2g&-g\\-g&2g\\\end{matrix}&\cdots&0\\\vdots&\ddots&\vdots\\0&\cdots&\begin{matrix}2g&-g\\-g&2g\\\end{matrix}\\\end{matrix}\right]_{n\ast n}
$$
Additionally,
$$
G_t=I\bigotimes G
$$

$$
G_b=G\bigotimes I
$$

where $G_t$ and $G_b$ are quasi-tri-diagonal matrix.

​	$Y_t$ and $Y_b$ are the non-ideal properties in the circuit of top layer and bottom layer, usually the writing and reading noise and thermal noise.

## *2.3*    *Coupling Model*

​	Since both layer’s nodal voltage in **MCA** will affect the memristor respectively, it is a non-linear relationship, so we couple the two equations. Combining the two equations into one matrix better for machine solving. A coupled matrix equation is proposed here:
$$
F=\ \left[\begin{matrix}G_t&0\\0&G_b\\\end{matrix}\right]*\ \left[\begin{matrix}V_t\\V_b\\\end{matrix}\right]+ \left[\begin{matrix}I_t\\{-I}_b\\\end{matrix}\right]- \left[\begin{matrix}Y_t\\Y_b\\\end{matrix}\right]
$$
To approach the result of equation
$$
F=0
$$
Newton-Raphson method is applied, in every iteration:
$$
\vec{V_{n+1}}=\vec{V_n}- F\left(\vec{V}\right)/J\left(\vec{V}\right)
$$
The Jacobian matrix of $F$ and $F'$ 
$$
F^\prime=\left[\begin{matrix}G_t&0\\0&G_b\\\end{matrix}\right]+\left[\begin{matrix}\frac{\partial I_t}{\partial V_t}&\frac{\partial I_t}{\partial V_b}\\\frac{-\partial I_b}{\partial V_t}&\frac{-\partial I_b}{\partial V_b}\\\end{matrix}\right]=J
$$
Then,
$$
J=\ \left[\begin{matrix}G_t+\frac{\partial I_t}{\partial V_t}&\frac{\partial I_t}{\partial V_b}\\\frac{-\partial I_b}{\partial V_t}&G_b+\frac{-\partial I_b}{\partial V_b}\\\end{matrix}\right]
$$
Hence the equation becomes to:
$$
\vec{V_{n+1}}-\vec{V_n}=\ J^{-1}\left(\vec{V_n}\right)\ast\left(-F\left(\vec{V_n}\right)\right)
$$
Further get:
$$
\ J\left(\vec{V_n}\right)\ast\left(\vec{V_{n+1}}-\vec{V_n}\right)=\left(-F\left(\vec{V_n}\right)\right)
$$
​	In this function, to show more intuitively in every step of iteration, we make the simplification of the following symbols:
$$
A=J\left(\vec{V_n}\right)
$$

$$
x=\left(\vec{V_{n+1}}-\vec{V_n}\right)
$$

$$
b=\left(-F\left(\vec{V_n}\right)\right)
$$

​	The equation of our primary concern has been transformed into:
$$
Ax=b
$$
![clip_image120](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image120.7f1cod8o2hg0.gif)



Fig. 3. Schematic diagram of the shape of matrix $A$ when $n=8$.  It should be noted that the dimension of the matrix here is $\left(2\ast n^2\right)\ast\left(2\ast n^2\right)$.

