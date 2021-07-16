---
layout: post
title: An optimization algorithm for demosaicing - using DE1 FPGA and camera with LCD screen display
date: 2021-7-14
categories: Research
tags: [Research, FPGA, Algorithm]
description: An optimization algorithm for demosaicing - using DE1 FPGA and camera with LCD screen display
---

# An optimization algorithm for demosaicing - using DE1 FPGA and camera with LCD screen display

> *NOTE: This is the course project of **System on Chip***.

*Abstract*— This article is designed to be used as the course final report for the course System-on-Chip Design. During the project, we used a DE1 FPGA development board, model 5CSEMA5F31C6, to implement the demosaic function required by the course, and added median filtering and gamma correction after the demosaic algorithm to make the output image meet the requirements of the human eye.

*Keywords—demosaicing, FPGA, median filtering, gamma correction*

# I.   Introduction 

Demosaicing algorithm is a digital image processing used to reconstruct full-color images from incomplete color samples output from image sensors covered with color filter arrays (CFA). It is also called CFA interpolation or color reconstruction. The reconstructed image is usually accurate in uniformly colored areas, but has a loss of resolution (detail and sharpness) and has edge artifacts.

Most modern digital cameras use a single sensor coated with a color filter array to acquire images, so demosaicing is a necessary part of the color image pipeline to reconstruct images into a generally viewable format. Many digital cameras are also capable of storing images as raw files and allow users to remove them and demosaic them using professional image processing software, rather than using the camera's built-in firmware.

# II.  Demosacing Algorithm

## A.  Color Filter Array

A color dropout array is a mosaic of color filters in front of a photographic element. The most commonly used color filter array configuration in business is the Bayer filter, as shown in the figure. It has an odd number of columns consisting of alternating red (R) and green (G) filters, and an even number of columns consisting of alternating green (G) and blue (B) filters. The number of green filters is twice as many as red and blue, which is to simulate the higher sensitivity of the human eye to green light.

Because color sampling from the filter array naturally creates blending problems, optical anti-aliasing filters are usually configured between the sensor and the optical range of the lens in order to eliminate false color noise from interpolation and color blending.

Because each pixel on the sensor is behind the filter, the output is a matrix of pixel values, each representing the original intensity of one of the three filtered colors, so a demosaicing algorithm is needed to estimate the color levels of each pixel for various colors (Color levels), not just a fraction of a color.

## B.  Bilinear Interpolation

Green pixel interpolation: assigns the average of the upper, lower, left and right pixel values as the G-value of the interpolated pixels. Example: G8 = (G3 + G7 + G9 + G13)/ 4.

![图片2](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/An-optimization-algorithm-for-demosaicing---using-DE1-FPGA-and-camera-with-LCD-screen-display/图片2.3c1ejvsnkhe0.png)

`Fig. 1.  Bayer Layer example`

Interpolation of red/blue pixels at the green position: the average of two neighboring pixel values of the corresponding color is assigned to the interpolated pixel. Example: B7 = (B6 + B8)/ 2; R7 = (R2 + R12)/ 2.

Interpolation of red/blue pixels at blue/red positions: the average of four adjacent diagonal pixel values is assigned to the interpolated pixel. Example: R8 = (R2 + R4 + R12 + R14)/ 4; B12 = (B6 + B8 + B16 + B18)/ 4. The number of cases for this algorithm is the same as the number of cases for 3*3, which is also 4.

## C.  Optimazation of Bilinear Interpolation

In Fig. 1., if the first 4x4 area is taken instead of 3x3 pattern, it is easy to get the array. In this case, it is necessary to obtain the middle four-pixel luminance values, 7, 8,12, 13, where the algorithms for all four are calculated using bilinear interpolation. For example, R7=(R2+R12)/2, B7=(B6+B8)/2, and so on.

So, we can get all the information of the 2x2 pixels in the middle through the 4x4-pixel grid in this case, and the clock requirement matches the original requirement of two clocks to get the result.

## D.  Change of Buffer

At the same time, we modify the structure of the buffer while modifying the number of fetched pixel points. We changed the original 3-row linebuffer to a 4-row linebuffer and used one more linebuffer. two linebuffer are taking the brightness values of 1*4 vertical columns of this column and the brightness values of 1x4 vertical columns of pixels after two clocks, which are input to the second and fourth columns of the processing matrix in turn.

![图片3](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/An-optimization-algorithm-for-demosaicing---using-DE1-FPGA-and-camera-with-LCD-screen-display/图片3.ujkacynetj4.png)

`Fig. 2.  (a) Optimazation of Linebuffer and (b) processing matrix`

# III.  Post Processing

## A.  Median Filter

Median filtering is a nonlinear signal processing technique based on the statistical theory of ordering that can effectively suppress noise. The basic principle of median filtering is to replace the value of a point in a digital image or digital sequence with the median of the values of the points in a neighborhood of that point, so that the surrounding pixel values are close to the true value, thus eliminating isolated noise points.

In Fig. 1., for instance, after taking out the 4*4 pixels for the first time and getting all the information values of the middle 2*2 pixels, we sort the red, green and blue pixel information of the middle four pixels and take the average of the middle two values as the output of the red, green and blue colors. Where the sorting is implemented we are using combinational logic circuits, i.e., no clock consumption is required.

## B.  Gamma Correction

In TVs and graphic monitors, the electron beam that occurs in the picture tube and the brightness of the image it generates does not change linearly with the input voltage of the tube; the electron flow changes according to an exponential curve compared to the input voltage, which is greater than the exponent of the electron beam. This means that the signal in the dark area is darker than the actual situation, while the light area is higher than the actual situation. Therefore, to reproduce the picture captured by the camera, the TV and monitor must be gamma compensated. This gamma correction can also be done by the camera. We gamma compensation for the entire TV system, the purpose is to make the camera according to the incident light brightness and the brightness of the picture tube symmetry and the output signal, so the image signal should be introduced an opposite nonlinear distortion, that is, with the TV system gamma curve corresponding to the camera gamma curve, its value should be 1/γ, we call the camera gamma value. The gamma value of the TV system is about 2.2, so the TV system's camera nonlinear compensation gamma value of 0.45. The gamma value of the color picture tube is 2.8, its image signal correction index should be 1/2.8 = 0.35, but due to the impact of stray light inside and outside the tube, the contrast and saturation of the reproduced image are reduced, so the gamma value of the color camera is still mostly used 0.45. In In practice, we can adjust the gamma value within a certain range according to the actual situation, in order to obtain the best results.

The so-called gamma correction is to edit the gamma curve of the image, in order to non-linear tone editing method of the image, detect the dark and light parts of the image signal, and make the ratio of the two increase, so as to improve the image contrast effect. Computer graphics field is used to this screen output voltage and the corresponding luminance conversion relationship curve, called the gamma curve.

In our code, we set the gamma values to 2.2 and 1.4, respectively, and did not use a resource-intensive algorithm like a lookup table, because we wanted to use the keys to achieve different gamma values between the conversion on the LCD screen to get different gamma values.

## C.  Implementation of WUJIAN100

Demosaicing module                   

We used the wujian100 output enable signal when installing the module we designed. Fig. 3. Give the information of how we use wujian100.

![图片4](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/An-optimization-algorithm-for-demosaicing---using-DE1-FPGA-and-camera-with-LCD-screen-display/图片4.6lq42yyj6rk0.png)

`Fig. 3.  (a) Optimazation of Linebuffer and (b) processing matrix`

 

 

# IV.  Result

In Fig.4., the image displayed by the proximity interpolation algorithm is shown. Comparing the algorithm using bilinear interpolation in Fig. 5., the use of the bilinear interpolation algorithm with median filtering improves the edge jaggedness effect very well. The same can be seen more clearly in the image with gamma correction (Fig.6.), and the dark information in the image with gamma correction is much clearer.

![图片5](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/An-optimization-algorithm-for-demosaicing---using-DE1-FPGA-and-camera-with-LCD-screen-display/图片5.2gjhqm52nndw.jpg)

`Fig. 4.  Nearest Neighbor Replication`

![图片6](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/An-optimization-algorithm-for-demosaicing---using-DE1-FPGA-and-camera-with-LCD-screen-display/图片6.6e1x6quoq500.jpg)

`Fig. 5.  Bilinear Interpolation and Median filter (no Gamma Correction)`

![图片7](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/An-optimization-algorithm-for-demosaicing---using-DE1-FPGA-and-camera-with-LCD-screen-display/图片7.tctvwhiw2sg.jpg)

`Fig. 6.  Bilinear Interpolation and Median filter (Gamma = 2.2)`

In order to see more clearly, we chose to use the camera to illuminate the edges at the same location where the features are compared, and it can be seen that the images obtained using our algorithm can eliminate the jagged effect better than the original proximity interpolation algorithm. It can also be seen that the median filtering reduces a lot of the noise information in the bright areas (Fig. 7.).

| ![图片8](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/An-optimization-algorithm-for-demosaicing---using-DE1-FPGA-and-camera-with-LCD-screen-display/图片8.1pjcviabprts.jpg) | ![图片9](https://cdn.jsdelivr.net/gh/Mi5sssss/blog_image@main/An-optimization-algorithm-for-demosaicing---using-DE1-FPGA-and-camera-with-LCD-screen-display/图片9.43orysol4600.jpg) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |

 `Fig. 7.     (a) Original (b) our result`

# V.  Conclusion

In our improved algorithm, the 3x3 buffer is removed and replaced by a 4x4 computation matrix, and the line buffer structure is modified to use two line buffer to perform the data fetching operation in parallel.

After executing the bilinear interpolation algorithm, we added the sorting and median filtering for the four values after the obtained results to receive the corresponding primary output red, green and blue signals, which were then transmitted to the LCD screen for display. In each processing (including whether to use the algorithm of bilinear interpolation, whether to use median filtering, whether to use gamma correction and the difference between taking the value of 2.2 and 1.4 for gamma) we use switches and buttons as the signal for switching. We finally use the wujian100 as an enable signal to connect the keys to the module.

##### Acknowledgment

·     Prof. An Fengwei, for brilliant teaching and patient tutorial

·     Every TA for excellent instruction and homework correction

**
**

 