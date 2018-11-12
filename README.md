## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.


The code for this step is contained within the first 3 code cells of the IPython Notebook Advanced_Lane_Project.ipynb. 

A number of images of chessboards, taken from different angles and viewpoints are used as inputs to the OpenCV functions 
`findChessboardCorners` and `calibrateCamera`. The idea here is that, we can use the outputs from `findChessboardCorners` function, which maps the internal chessboard corners to image points, as input into the `calibrateCamera` function. This gives us the camera calibration and distortion coefficients. We can now use the OpenCV function `Undistort` to undo the effects of distortion on any image

![Camera Calibration][output_images/Camera_Calibration.png]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The effects of applying `undistort` function are subtle but we can see that the image has changed, by looking at the image of the hood. 
![Undistorted_Road][output_images/Camera_Calibration_Road.png]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.


Gradient Thresholds 

We use different kinds of gradient thresholds 

* Apply sobel filters along the X axis, mostly because the lane lines are vertical 
* Apply directional gradient thresholds


Color Transforms

* We filter along the R and G channels to isolate Yellow lines
* S Channel filter to separate yellow and white lines from the above image
* L channel filter to discard edges because of shadows


Here's an example of my output for this step.

![Thresholded Binary Image][output_images/Thresholded_Binary_Road.png]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I did a perspective transform to adjust the veiwing angle of images. In this case, I defined the points manually in the original image and then did a transform from the top down

I choose the source points manually by examining the previous cell

Destination points are chosen such that we have a top down view of the selected points

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 220, 720      | 320, 720      | 
| 1150, 720     | 800, 720      |
| 575, 480      | 320, 1        |
| 750, 480      | 800, 1        |


I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![Perspective_Transform][output_images/Perspective_Transform.png]



#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?


Using the perspective transformed image, we see that the first half of the perspective warped image is likely to have the lane lines. It is shown below

![First Half of the Transformed Image][output_images/IdentifyLanes1.png]

We then use the image above, and we take the sum of all values in each row. This gives us the sum of concentration of pixels in each row

We see that the peak in the first half is the left lane and the second half is the right lane. 

![Histogram Output][output_images/IdentifyLanes2.png]

I then used the sliding window search using the output from the histogram as the starting position of my right and left lanes. I then used 10 windows, 100 pixels width each. 

The x and y coordinates of all non zero pixels were found and a polynomial was fit. Using this, the lane lines were drawn. 

![Sliding Window][output_images/IdentifyLanes3.png]

I then searched around previously detected lanelines. This was because, consecutive frames are likely to have lane lines in similar places. Using a margin of 50 pixels, we search the previously detected lane lines. 

![Search Previous][output_images/IdentifyLanes4.png]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](https://youtu.be/YBSi50AnDxk)



---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Some of the issues I had to encounter were the following 

* More information, and not just lane coordinates from other frames can be used in the other frames to increase robustness
* This would fail if there was a road crossing, Rain or Snow
* Tail gating a car would almost certainly disrupt the line detection
* The color and gradient thresholding were not straight forward and there was a lot of experimenting that had to be done
