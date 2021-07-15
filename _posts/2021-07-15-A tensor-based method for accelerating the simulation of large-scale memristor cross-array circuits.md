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

![clip_image002](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image002.5lqmerbw7ac0.gif)

Fig. 1. A general MCA is shown with BL (bit line) and WL (word line).



​     A voltage-controlled extended memristor is illustrated by the following equation:

![clip_image004](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image004.1k479rxt8ee8.gif)

![clip_image006](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image006.vaqmjeulsyo.gif)

![clip_image008](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image008.385dzdxldd00.gif)

​     Where the ![clip_image010](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image010.fh3f65gbmz.gif) is the resistance state of each memristor, and ![clip_image012](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image012.w5nnvhy7htc.gif) is the voltage that applied to both ends. ![clip_image014](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image014.2necb93mpyc0.gif) is the current that we can obtain from measurement. Conductance of memristor ![clip_image016](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image016.6250gf2avns0.gif) is a function related to ![clip_image010](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image010.fh3f65gbmz.gif) and ![clip_image012](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image012.w5nnvhy7htc.gif), which shows a similar property of Ohm's Law. The state equation ![clip_image018](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image018.80pnd6hoooo.gif) is a function ![clip_image020](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image020.6oga5holp5c0.gif) connected with ![clip_image010](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image010.fh3f65gbmz.gif) and ![clip_image012](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image012.w5nnvhy7htc.gif).

​     In Fig .1., analog input vector values are mapping to the voltage with alterable pulse lengths and amplitudes voltages, and the output vector values are represented as currents.

​     The operation of MCA is divided into two parts, writing process and reading process. As there are no insulation between reading and writing, crosstalk is existed in the circuit, which we would not pay attention to in this article.

 

## *B.*    *Numerical Description*

 

​     In a SPICE model, the crossbar array circuits are commonly illustrated by matrix.

​     In the MCA, due to the special characteristics of the circuit, the conductance in the circuit can be reduced to a matrix ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image016.gif) in the form of cross point, and since cross point can only form conductance and interact with adjacent cross point, this special characteristic leads to a matrix in the form of a tri-diagonal matrix.



| ![clip_image022](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image022.4lgsjken4q20.gif)  (a) | ![clip_image024](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image024.34wt1x0t4160.gif)  (b) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![clip_image026](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image026.u4zo6aw5te8.gif)  (c) | ![clip_image028](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image028.5xz5ezl7mtw0.gif)  (d) |



Fig. 2. (a) The general MCA with wire resistance concerned, forming ![clip_image030](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image030.5n4xr4x7wu80.gif) dimension crossbar structure. (b) The division of memristor.  (c) A division of row wire resistance, which can be represented as ![clip_image032](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image032.78s2e6cg5v00.gif). (d) A division of column wire resistance, which can be represented as ![clip_image034](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image034.3odo99ljz200.gif). 





Fig. 2(a) shows that in actual, wire conductance is concerned in case of large accuracy error. Fig 2(b), Fig 2(c) and Fig 2(d) showcase the separation of the general MCA structure. The ![clip_image032](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image032.78s2e6cg5v00.gif) and ![clip_image034](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image034.3odo99ljz200.gif) are the conductance of rows and columns, in the matrix form. Given an assumption that every wire resistance is identical, is represented in the graph as resistors. Hence, the ![clip_image032](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image032.78s2e6cg5v00.gif) and ![clip_image034](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image034.3odo99ljz200.gif) matrix are congruent, represented by ![clip_image016](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image016.6250gf2avns0.gif) instead.

We give our model by the following MNA (Modified Nodal Analysis) equation:

![clip_image040](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image040.1nyg1awb6bs0.gif)

where the ![clip_image042](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image042.2cef22425l3w.gif) and ![clip_image044](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image044.3rbtg58cme00.gif) in subscript represented top layer and bottom layer of an MCA. ![clip_image046](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image046.2bc7pmpwsbb4.gif) and ![clip_image048](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image048.rywpyleozls.gif) are function that related to voltage applied to the nodes from bottom and top wire, a set of nonlinear equations. ![clip_image050](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image050.pyqgb2xooqo.gif) is an array of the top nodal voltages arranged by rows, as well as ![clip_image052](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image052.53ouwnu7if80.gif).

![clip_image054](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image054.14zef8ubhykg.gif)

The ![clip_image056](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image056.zmwiyttjgpc.gif) and ![clip_image058](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image058.cjh1mchqeds.gif) represent the nodal voltage of the ![clip_image060](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image060.33phdsxdqu60.gif) row and ![clip_image062](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image062.1myzxxq8f61s.gif) column on the top and bottom grid, respectively.

The G represent the wire conductance matrix in MNA.

![clip_image064](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image064.1v0dttc53wio.gif)

 

Additionally,

![clip_image066](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image066.68gnt8fq63c0.gif)

![clip_image068](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image068.5vm58u6am3o.gif)

where ![clip_image070](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image070.25vaaxk5hf8g.gif) and ![clip_image072](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image072.4gliyxalifm0.gif) are matrix of ![clip_image074](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image074.50ctwv5bzgc0.gif) dimension, since ![clip_image076](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image076.4wbw3wi34ec0.gif) is identity matrix of ![clip_image078](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image078.672qnal9ouc0.gif).

![clip_image080](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image080.2mi8qlrxfd00.gif) and ![clip_image082](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image082.29j4nun1xg2s.gif) are the nonideal properties in the circuit of top layer and bottom layer, usually the writing and reading noise and thermal noise.

 

## *C.*   *Coupling Model*

 

Since both layer’s nodal voltage in MCA will affect the memristor respectively, it is a non-linear relationship, so we couple the two equations. Combining the two equations into one matrix better for machine solving. A coupled matrix equation is proposed here:

![clip_image084](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image084.1f81snebcfi8.gif)*![clip_image086](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image086.bdq398jaetk.gif) ![clip_image088](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image088.qb53339g70w.gif) ![clip_image090](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image090.5r4hxkgfguc0.gif)

 

To approach the result of equation

![clip_image092](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image092.41wod3x8m7q0.gif)

Newton-Raphson method is applied, in every iteration:

![clip_image094](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image094.9nldr96nt0k.gif) ![clip_image096](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image096.4kcora2meg8.gif)

The Jacobian matrix of ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image098.gif) is ![img](img/2021-07-15-A tensor-based method for accelerating the simulation of large-scale memristor cross-array circuits/clip_image100.gif):

![clip_image102](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image102.5boypm9sp6c0.gif)

Then,

![clip_image104](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image104.7b5qab64ti00.gif)

Hence the equation becomes to:

![clip_image106](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image106.5j3wekc98po0.gif)

Further get:

![clip_image108](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image108.4a1pej6o6qq0.gif)

In this function, to show more intuitively in ![clip_image110](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image110.7y1xww4vnig.gif) every step of iteration, we make the simplification of the following symbols:

![clip_image112](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image112.6izugu7ffn00.gif)

![clip_image114](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image114.7co6ia3kg840.gif)

![clip_image116](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image116.6pky45ho9b80.gif)

The equation of our primary concern has been transformed into:

![clip_image118](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image118.3sv947iybv80.gif)

 

![clip_image120](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image120.7f1cod8o2hg0.gif)

Fig. 3. Schematic diagram of the shape of matrix A when ![clip_image122](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image122.6l2rzjxz21w0.gif). It should be noted that the dimension of the matrix here is ![clip_image124](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image124.53tz8cjhick0.gif)

 

# III. The Accelerator Design on Solution

 

In this section, we mainly focus on using the iterative solution method of GMRES (Generalized minimal residual method) to solve the problem of solving the sparse diagonal matrix. Fig. 3 show a case when the crossbar dimension n is 8, forming a sparse band matrix with dimension 128. Typically, the band matrix we can approximately treat as a seven-diagonal-band matrix, thought there are some vacancies. A preconditioner ![clip_image126](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image126.56o6mupluns0.gif) is required for the GMRES methods, for better solving process. 

![clip_image128](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/2021-07-15-A-tensor-based-method-for-accelerating-the-simulation-of-large-scale-memristor-cross-array-circuits/clip_image128.12570qo75ng0.gif)

