# Sun Detection Workshop Documentation
 
**Project:** Project SunbYte SOL workshop on computer vision

**Files covered:** `sun_detection_workshop.ipynb`

**Language / libraries:** Python 3.10+, OpenCV, NumPy, Matplotlib

## Workshop Introduction

The purpose of the workshop is to give students an introduction to building a computer vision pipleline one step at a time using Google Colab. 

Students will be provided with an image of the sun from a previous SunbYte mission and they will have to work out:

**1. Where** the sun is by calculating its pixel coordinates and radius.

**2. How far** it is from the centre of the frame in pixels and in degrees.

**3. How far away the sun is** 

## The pipeline students will be following


| Step | What happens | Why |
|---|---|---|
| 1 | Greyscale + blur | Blurring to hide noise and sunspots |
| 2 | Threshold | Split the image into "bright" and "not bright" |
| 3 | Clean the mask | Remove specks, fill holes |
| 4 | Find contours, fit a circle | Turn bright pixels into candidate shapes |
| 5 | Check it's round | Reject clouds, reflections and lens flare |
| 6 | Find coordinates | Using contour method find centre |

## Steps we will be taking

### **1.** Import `sun.jpg` to use in the workshop

### **2.** Setting up imports and defining a helper function *show()* that unpacks the image that are being passed and giving them titles.

- The students will need to run each section one at a time, as methods and variables are repeated throughout the code sections

### **3. Greyscale and Blurring:**  We will first convert the image to greyscale so we can easily detect the sun in later steps. 

Following along, we will add a **Gaussian blur**, which smooths out camera noise and averages each pixel with its neighbours. 

- Students will change the `BLUR_SIZE` to see the difference between having it set to small and large values, as well as why we cannot have it be an even number.

- **The Challenge** is to for them to print out the brightest value pixel by using `blurred.max()` in an if statement to check whether its in the 0-255 scale. This block should be wrapped in a `try()` and should raise a `RunTimeError` if the value is outside the pixel range.


### **4. Thresholding**, which turns the greyscale image into a black and white mask, leading to the image only having one bright sun and a black background.

- Students will get an introduction to **Otsu's method** which looks at the brightness histogram picks the value that best separates the dark and bright groups of pixels.

- The red line on the histogram shows the cut-off for the black pixels, everything to the **right** becomes white in the mask and everything to the **left** becomes black

-  The students are tasked to change the value of the constant `THREHSOLD` and observe what happens to the mask.


### **5. Cleaning up the mask:** 
The mask may contains sunspots and noise from the camera. To fix this we use two **morphological** operations 

- **Opening:** revomes small white specks
- **Closing:** fills in small back holes

We use a 5x5 ellipse shaped **Kernel**: a small shape that slides over the image.

- **Challenge:** Write a `show()` function to present the `raw mask`, `opened mask`, and the `mask after closing`, with their corresponding titles on the screen.

- The end result should be 3 images with tites: "Raw mask", "Opened mask", "Mask after closing"


### **6. Finding contours:** 
A contour is simply the outline of a white region in the mask. For each contour we measure:

- area: the number of pixels inside it

- perimeter: the length of its outline

- minimum enclosing circle: the smallest circle that fits around it

- **Challenge:** Students will need to claculate the area using contourArea(c), perimeter with arcLength(c, True) (True means the circle is closed) and minimum enclosing circle with minEnclosingCircle(c)


### **7. Check circularity**

To check circularity we must compare the area with the perimeter:

$$\text{circularity} = \frac{4\pi \times \text{area}}{\text{perimeter}^2}$$

A perfect circle scores **1.0**.

We will also calculate the **fill ratio** to see how much of the enclosing circle the shape fills

$$\text{fill} = \frac{\text{area}}{\pi r^2}$$

- The first largest contour that passes both tests is our sun. 

- To get its centre, we use the **centroid**, which is the average position of all its pixels.

- **Challenge:** Students need to calculate `circularity` and `fill` using the values `area` and `perimeter` from the step above.

### **8. Calculating the sun's coordinates** 

Image coordinates are not maths coordinates. In an image, (0, 0) is the top-left corner, x increases to the right and y increases downwards.

For the telescope, we convert to centre-based coordinates with *y* pointing **up**, as it needs to know where the sun is relative to the **centre** of the frame.

We define:
- dx = how far **right** of the centre
- dy = how far **above** the centre

- **Challenge:** The sudents must find `dx` and `dy` using `cx = w / 2` and `cy = h / 2`



## How we will run the workshop

- 1. In University Computer labs with provided computers

- 2. In a classroom using students own computers

- 3. Remotely with pre-recorded demonstation 











 
