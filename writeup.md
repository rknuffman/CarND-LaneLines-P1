# **Finding Lane Lines on the Road**

## Writeup (rknuffman)

---

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./test_images_output/gray.jpg "Grayscale"
[image2]: ./test_images_output/blur.jpg "Gaussian Blur"
[image3]: ./test_images_output/edges.jpg "Canny edge detector"
[image4]: ./test_images_output/masked.jpg "Region of interest"
[image5]: ./test_images_output/solidWhiteCurve.jpg "Lanes"

---

### Reflection

### 1. Pipeline

My pipeline consisted of 6 steps:
* **Gray**: Convert the RGB input images to grayscale.
* **Blur**: Smooth the grayscale images using a gaussian blur with 9x9 kernel (selected through experimentation and perceived performance improvements).
* **Edge**: Run the smoothed image through a Canny edge detector with lower and upper thresholds of 5 and 100 (manually tuned, again).
* **Mask**: Focus on edges within a trapezoidal region of interest at the bottom-center of the image.  This masks irrelevant edges detected throughout portions of the image which should not contain the road.
* **Lines**: Use Hough transform to identify potential line segments within the masked edge image.  Parameterization included rho=1, theta=pi/180, a minimum of 40 votes, minimum length of 20 and maximum gap of 120 pixels.
* **Extrapolate**: Extend identified segments to display continuous lines to the left and right of the current lane.  This step relied on calculating the slope and intercept of each segment, filtering out unexpected segments (e.g. slopes close to zero), grouping left and right segments (positive vs. negative slope), and averaging the slope and intercept of each side.  

In order to draw a single line on the left and right lanes, I modified the draw_lines() function to utilize extrapolated points from the final step of the pipeline above, extending from slightly more than halfway down the image (62%) to the bottom of the image.  

![][image1]

![][image2]

![][image3]

![][image4]

![][image5]


### 2. Potential Shortcomings

Naturally, segmented lane markings pose more of a challenge than solid markings.  Because of the perspective of the camera capturing the images, there is a repetitive point where a segment has just moved out of the field of view, and remaining line segments are still distant.  The maximum line gap of the Hough transform was set at an extremely large value to help potentially connect the distant segments, but it will not always result in a detected segment along that side of the road.  

Additionally, lane markings can't be assumed to have a uniform appearance.  The challenge video is a good example, where lane markings are obscured by shadows or varying light conditions.  Scenarios with similar complications include varying weather conditions, different times of day, other vehicles obscuring lane markings (perhaps through lane changes), etc.  Some rural roads have no markings, or center markings only.  


### 3. Possible Improvements

There's certainly the possibility that a more sophisticated method could outperform the simple averaging of left and right lane parameters.  Perhaps the hough transform could be avoided entirely and replaced with an OLS regression of each half of the masked edge image.  

It would be interesting to see a weighted update, where line parameters from previous frames are given some weight when updating.  While this would have the effect of smoothing some of the fluctuations, there is a massive danger in preventing your lane finder from reacting quick enough to changing environments (like curves).  
