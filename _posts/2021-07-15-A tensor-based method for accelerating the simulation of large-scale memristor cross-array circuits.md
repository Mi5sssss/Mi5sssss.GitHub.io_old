---
layout: post
title: A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits
date: 2021-7-15
categories: Research
tags: [Research, EDA, Algorithm]
description: A quick solution to specific matrix in MCA.
---

# I Introduction

 

​	Currently, computers with von Neumann architecture composed of traditional CMOS (Complementary Metal Oxide Semiconductor) chips often face the serious limitation of ‘memory wall’ when training neural networks, i.e., due to the large amount of data exchange between processor memory, which generates huge energy consumption, and slow computing speed, often unable to effectively handle real-time data.

​	So far, the main categories of memristor-based neural networks are memristor bridge synaptic neural networks and crossbar array neural network circuits. The weights of an MCA (memristor crossbar array) neural network are related to the conductance values of the memristors. Compared with the memristor bridge neural network circuit, the synaptic and overall network circuit structure of this type of neural network are simpler. Two memristors are used to form the crossbar array, and the synaptic weights are the difference of the two array memristor values, so that the weights can be positive, zero, or negative, which makes it easy to update the weights. The circuit structure of crossbar arrays is usually utilized, which allows interconnection of neurons and realization of neural networks with very large integration density. The overall neural network circuit uses a current mode approach to sum multiple output currents.

​	The MCA has a special structure that allows for the simulation of matrix multiplication and convolutional operations, which are the core of AI (artificial intelligence) algorithms, to achieve integrated storage and computation. The crossbar matrix circuit has a memory calculation function at the crossbar, where data storage and operation can be performed at the same point. However, considering the parasitic effect of capacitors, for very large-scale cross-matrix simulation, the commonly used general-purpose circuit simulator SPICE (Simulation Program with Integrated Circuit Emphasis) is very slow, and most of the simulations for cross-switched array circuits mainly focus on the estimation of switching levels such as area power consumption, not carrying out very accurate simulations, and cannot model their functions well in simulation calculations.

​     Existing cross-array simulators, such as MNSIM, mainly skip the SPICE-level circuit simulation and use highly simplified behavioral-level models to simulate the characteristics of memristors and crossbar arrays. It essentially increases the simulation speed by sacrificing accuracy, and thus does not guarantee simulation accuracy and makes it difficult to perform design optimization for complex practical applications. Therefore, the main objective of this essay is to develop dedicated circuit simulation acceleration techniques and tools for the characteristics of MCA using tensor-based technology to improve the simulation efficiency by one order of magnitude, and to realize the design and optimization of memristor neural networks based on circuit simulation. Specifically, we hope to make full use of the special topology of the memristor array and build a corresponding preconditioner based on the tensor to accelerate the core iterative linear system solution process in the simulation process and achieve the optimal balance of simulation speed and accuracy.

 

# II. Architecture of MCA Circuit

 

## *A.*    *Physical MCA Circuit Architecture*

 

  Fig. 1. showcases a general architecture of MCA. The conductance of each memristor device store a matrix element or weight. Usually the voltage-controlled extended memristor is applied to the array since voltages with changeable input pulse lengths and amplitudes are easy-controlled by source meter, without requirement of high accuracy.

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image002.gif)

Fig. 1. A general MCA is shown with BL (bit line) and WL (word line).



​     A voltage-controlled extended memristor is illustrated by the following equation:

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image004.gif)

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image006.gif)

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image008.gif)

​     Where the ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image010.gif) is the resistance state of each memristor, and ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image012.gif) is the voltage that applied to both ends. ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image014.gif) is the current that we can obtain from measurement. Conductance of memristor ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image016.gif) is a function related to ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image010.gif) and ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image012.gif), which shows a similar property of Ohm’s Law. The state equation ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image018.gif) is a function ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image020.gif) connected with ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image010.gif) and ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image012.gif).

​     In Fig .1., analog input vector values are mapping to the voltage with alterable pulse lengths and amplitudes voltages, and the output vector values are represented as currents.

​     The operation of MCA is divided into two parts, writing process and reading process. As there are no insulation between reading and writing, crosstalk is existed in the circuit, which we would not pay attention to in this article.

 

## *B.*    *Numerical Description*

 

​     In a SPICE model, the crossbar array circuits are commonly illustrated by matrix.

​     In the MCA, due to the special characteristics of the circuit, the conductance in the circuit can be reduced to a matrix ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image016.gif) in the form of cross point, and since cross point can only form conductance and interact with adjacent cross point, this special characteristic leads to a matrix in the form of a tri-diagonal matrix.



| ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image022.gif)  (a) | ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image024.gif)  (b) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![图表  描述已自动生成](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image026.gif)  (c) | ![图片包含 截图, 灯光, 照片  描述已自动生成](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image028.gif)  (d) |



Fig. 2. (a) The general MCA with wire resistance concerned, forming ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image030.gif) dimension crossbar structure. (b) The division of memristor.  (c) A division of row wire resistance, which can be represented as ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image032.gif). (d) A division of column wire resistance, which can be represented as ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image034.gif). 





Fig. 2(a) shows that in actual, wire conductance is concerned in case of large accuracy error. Fig 2(b), Fig 2(c) and Fig 2(d) showcase the separation of the general MCA structure. The ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image036.gif) and ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image038.gif) are the conductance of rows and columns, in the matrix form. Given an assumption that every wire resistance is identical, is represented in the graph as resistors. Hence, the ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image036.gif) and ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image038.gif) matrix are congruent, represented by ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image016.gif) instead.

We give our model by the following MNA (Modified Nodal Analysis) equation:

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image040.gif)

where the ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image042.gif) and ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image044.gif) in subscript represented top layer and bottom layer of an MCA. ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image046.gif) and ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image048.gif) are function that related to voltage applied to the nodes from bottom and top wire, a set of nonlinear equations. ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image050.gif) is an array of the top nodal voltages arranged by rows, as well as ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image052.gif).

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image054.gif)

The ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image056.gif) and ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image058.gif) represent the nodal voltage of the ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image060.gif) row and ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image062.gif) column on the top and bottom grid, respectively.

The G represent the wire conductance matrix in MNA.

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image064.gif)

 

Additionally,

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image066.gif)

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image068.gif)

where ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image070.gif) and ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image072.gif) are matrix of ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image074.gif) dimension, since ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image076.gif) is identity matrix of ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image078.gif).

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image080.gif) and ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image082.gif) are the nonideal properties in the circuit of top layer and bottom layer, usually the writing and reading noise and thermal noise.

 

## *C.*   *Coupling Model*

 

Since both layer’s nodal voltage in MCA will affect the memristor respectively, it is a non-linear relationship, so we couple the two equations. Combining the two equations into one matrix better for machine solving. A coupled matrix equation is proposed here:

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image084.gif)***![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image086.gif) ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image088.gif) ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image090.gif)

 

To approach the result of equation

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image092.gif)

Newton-Raphson method is applied, in every iteration:

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image094.gif) ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image096.gif)

The Jacobian matrix of ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image098.gif) is ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image100.gif):

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image102.gif)

Then,

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image104.gif)

Hence the equation becomes to:

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image106.gif)

Further get:

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image108.gif)

In this function, to show more intuitively in ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image110.gif) every step of iteration, we make the simplification of the following symbols:

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image112.gif)

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image114.gif)

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image116.gif)

The equation of our primary concern has been transformed into:

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image118.gif)

 

![图形用户界面, 图表  描述已自动生成](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image120.gif)

Fig. 3. Schematic diagram of the shape of matrix A when ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image122.gif). It should be noted that the dimension of the matrix here is ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image124.gif)

 

# III. The Accelerator Design on Solution

 

In this section, we mainly focus on using the iterative solution method of GMRES (Generalized minimal residual method) to solve the problem of solving the sparse diagonal matrix. Fig. 3 show a case when the crossbar dimension n is 8, forming a sparse band matrix with dimension 128. Typically, the band matrix we can approximately treat as a seven-diagonal-band matrix, thought there are some vacancies. A preconditioner ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image126.gif) is required for the GMRES methods, for better solving process. 

![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image128.gif)

