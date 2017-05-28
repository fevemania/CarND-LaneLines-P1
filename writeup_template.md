# **Finding Lane Lines on the Road** 

## Writeup Template

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"
[image2]: ./demo/roi_before_canny_1.png "roi_before_canny_1"
[image3]: ./demo/roi_before_canny_2.png "roi_before_canny_2"
[image4]: ./demo/roi_after_canny_1.png "roi_after_canny_1"
[image5]: ./demo/roi_after_canny_2.png "roi_after_canny_2"
[image6]: ./demo/apply_grayscale_1.png "apply_grayscale_1"
[image7]: ./demo/apply_grayscale_2.png "apply_grayscale_2"
[image8]: ./demo/apply_color_select_1.png "apply_color_select_1"
[image9]: ./demo/apply_color_select_2.png "apply_color_select_2"

---

### Reflection

### Q1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.
---
### A1. My pipeline consisted of 8 steps. 

(1) First, I set all global parameters used in following function at the beginning.

**Notably, it's important to put both image.shape and the calculation of vertices for roi in the process_img function.
Because the roi region should change according to the resolution of input image; for instance, test_image and previous two video both are 960*540, but
challenge.mp4 is 1280*720**

    color_roi = np.dstack((roi, roi, roi))
    return weighted_img(edges_with_lines, color_roi)

(2) Then I convert the color space of original image from RGB to GRAYSCALE. (Or to the particular range of our color select).

(3) Do guassian blur to reduce noise.

(4) After that the canny algorithm come in to extract edges according two thresholds.

**Notably, I was once try to split roi before do other matrix manipulations like blur or canny, but it seems like the canny algo will extract the edge of roi borders.**

![image2]
![image3]
![image4]
![image5]

(5) After selecting roi, It is time to draw line on it!

*** In order to draw a single line on the left and right lanes, I modified the draw_lines() function by ***

* first, seperate points into two collections, point_in_positive_slope and point_in_negative_slope

And the Tricky method I use is to set the positive boundary equals to 30, and negative boundary equals to -30, only slope above 30 would be added into positive collection, and the slopes below -30 added into negative collection.

    # manually select the slope boundary (tricky method)
    positive_degree_bound = 30 / 180 * np.pi
    negative_degree_bound = (-30) / 180 * np.pi
    

* further, I seperate x and y of each of two collections, form px (the set of x coordinates belong to point_in_positive_slope),   
    py (the set of y coordinates belong to point_in_positive_slope), nx (the set of y coordinates belong to point_in_negative_slope), ny (the set of y coordinates belong to point_negative_slope).
    
* use np.polyfit to calculate average slope and intercept on negative and positive point collection. And This is how I extrapolate the line.

* then I need to find two point to form positive slope line and the other two point to form negative slope line.
    
    In order to draw full line with no broken point, I select two y coordicate to reversely calculate corespond x coordinate.
two y coordinate are: 

  (a) first y is at the botton of image 
  (b) second y times 0.6 from first y

(6) After draw line on roi, use cv2.addWeighted method to linearly blend roi_with_line and original_img with RGB color space

<!--![alt text][image1]-->


### 2. Identify potential shortcomings with your current pipeline

The biggest shortcoming I think is that color select criterion is hard code.
Though it dramatically improve the stablity of two lane lines, it really restrict to fit only on the simple situation the homework provided for us.

* Following picture show the situation of applying grayscale and colorselect on challenge.mp4.

![image6]
![image7]
![image8]
![image9]

Imagine when the car pass through the lane with no street lamp in the dark night, maybe the color select will accidently ignore white or yellow lanes.

Another shortcomings might happen if there is the situation to change onto another lane, then my current pipeline will be broken.

Also when it comes to the other car right before our car, I believe the the average algorithm I use would definitely affected by it.

### 3. Suggest possible improvements to your pipeline

A possible improvement on the biggest shortcoming would be to totally give up color selection and consider only grayscale.
And maybe we should come up with another algo to fit in the dark night, such as to scan over each row of roi region to find out the point where the pixel intensity dramatically get high and low, that might be the bright white lane line or bright yellow lane line which in relation to dark ground, and accordingly find out the point collection on lane lines.

I personally have no idea about how to deal with lane tranfer problem.

For the last one, I think we should use computer vision and deep learning to first recognize the car in order to seperate it from our point collection algorithm.
