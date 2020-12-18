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

[//]: # (Image References)

[image1]: ./examples/undistort_output_mine.png "Undistorted"
[image2]: ./examples/straight_lines2.png "Road Transformed"
[image3]: ./examples/straight_lines2_thresh.png "Binary Example"
[image4]: ./examples/warped_straight_lines_mine.png "Warp Example"
[image5]: ./examples/first_fit.jpg "Fit Visual"
[image6]: ./examples/full_vis.png "Output"
[video1]: ./output_images/challenge.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb".  

OpenCV built in function cv2.findChessboardCorners locates the chessboard points. OpenCV's calibrateCamera assumes that the chessboard is flat and runs a regression for the 'plumb_bob' camera model using the points from the undistorted frames as inputs, the real world object corners as the predicted variable to find the distortion matrix. 

I applied this distortion correction to the test images using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]
Actually I can't really tell the difference, but I used opencv undistort to map the distorted image to undistorted one using the calculated distortion matrix again at line #137 of the jupyter notebook.
#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color (HSV) and gradient (sobel) thresholds to generate a binary image (thresholding steps at lines #104 through #136 in the notebook.
Here is an example
![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `corners_unwarp`, which appears in lines #211 through #214 and is used to warp another perspective to the image found by
the `get_perspective_matrix()`. This function takes as inputs image dimensions and source points (`p1,p2,p3,p4`) and maps them to a hardcoded destination rectangle. 
I chose the hardcoded source points in lines #168 to #186 by manually clicking on the straight lines image.

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 550, 464      | 0, 0          | 
| 743, 461      | 1280, 0       |
| 1279, 685     | 1280, 720     |
| 115, 685      | 0, 720        |


I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.
But to be honest, the src, dst from the example md looked better, so I switched.
![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Lane finding:
* I first used `corners unwarped` and only than applied the `pipeline` thresholds because it felt right to threshold in the same space in which I want to find the curvature.
* The funciton `find_lane_pixels_init` #309-#387 takes the binary warped image and applies the sliding window histogram technique taught in the lessons to find all the thresholded pixels from the right and left lanes:
  ![alt text][image5]

* The function `fit_poly` #436-#439 takes the thresholded pixels of the two lanes and fits a polynomial to each one.
* The function `search around poly` #459-#515 fits a polynomial using `fit_poly` to the pixels that passed the thresholds and fit in a mask of +-margin from the previous iteration polynomial. The function returns success if the pixel number for both lanes is above a thresh.
* The function `process_image` goes through the whole process described above, toggles between lane initialization and continuous tracking of lanes and also handles visualization and curvature calculation described next:


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The function `measure_curvature_real` uses the pixel/meters ratios provided in class and assumes it is valid to the warped perspective image, calculate the polyomial in meter units and finds the curvature using the curvature equation.

In lines #567-#570 the distance from center is calculated under the assumption that the camera principle axis is aligned with the center of the vehicle, the average curvature between the two lane polynomials start is subtracted from the mid pixel and multiplied by the pixel / meters ratio.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

With great pleasure the visualization took me too much time:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/challenge.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

No special issues, the project was pretty straight forward copying and pasting the lessons. The main issues for me was how to create the visualization of obstacles and lanes.

The main problem here would be the hardcoded threshold values plus the pipelines runs very slow. At night this algorithm doesn't stand a chance.
