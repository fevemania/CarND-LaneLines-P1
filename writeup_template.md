# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My original pipeline consisted of 8 steps. 

First, I set all global parameters used in following function at the beginning.

Notably, it's important to put both image.shape and the calculation of vertices for roi in the process_img function. (Point 1)
Because the roi region need to change dynamically accordingly to the resolution of input image.

Then I convert the color space of original image from RGB to GRAYSCALE.
And then do guassian blur to reduce noise.
After that the canny algorithm come in to extract edges according two thresholds.

Notably, I was once try to split roi before do other matrix manipulations like blur or canny, but it seems like the canny algo will extract the edge of roi borders.


After selecting roi, It is time to draw line on it!

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by

(1) first, seperate points into two collections, point_in_positive_slope and point_in_negative_slope

    And the Tricky method I use is to set the positive boundary equals to 30, and negative boundary equals to -30, only slope above 30 would be added into positive collection, and the slopes below -30 added into negative collection.

(2) further, I seperate x and y of each of two collections, form px (the set of x coordinates belong to point_in_positive_slope),   
    py (the set of y coordinates belong to point_in_positive_slope), nx (the set of y coordinates belong to point_in_negative_slope), ny (the set of y coordinates belong to point_negative_slope).
    
(3) use np.polyfit to calculate average slope and intercept on negative and positive point collection. And This is how I extrapolate the line.

(4) then I need to find two point to form positive slope line and the other two point to form negative slope line.
    
    In order to draw full line with no broken point, I select two y coordicate to reversely calculate corespond x coordinate.
two y coordinate are: 

  (a) first y is at the botton of image 
  (b) second y times 0.6 from first y

After draw line on roi, use cv2.addWeighted method to linearly blend roi_with_line and original_img with RGB color space

If you'd like to include images to show how the pipeline works, here is how to include an image: 


![alt text][image1]


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when 

The biggest shortcoming I think is that color select criterion is hard code.
It really restrict to fit only on the simple situation the homework provided for us.

Imagine when the car pass through the lane with no street lamp in the dark night, maybe the color select will accidently ignore white or yellow lanes.

Another shortcomings might happen if there is the situation to change onto another lane, then my current pipeline will be broken.

Also when it comes to the other car right before our car, I believe the the average algorithm I use would definitely affected by it.

### 3. Suggest possible improvements to your pipeline

A possible improvement on the biggest shortcoming would be to totally give up color selection and consider only grayscale.
And maybe we should come up with another algo to fit in the dark night, such as to scan over each row of roi region to find out the point where the pixel intensity dramatically get high and low, that might be the bright white lane line or bright yellow lane line which in relation to dark ground, and accordingly find out the point collection on lane lines.

I personally have no idea about how to deal with lane tranfer problem.

For the last one, I think we should use computer vision and deep learning to first recognize the car in order to seperate it from our point collection algorithm.




Another potential improvement could be to ...
