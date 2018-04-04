## Project 4 Writeup

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

[image1]: ./output_images/undistort_output.png "Undistorted"
[image2]: ./output_images/undistorted_image.png "Road Transformed"
[image3]: ./output_images/stacked_thresholds.png "Binary Example"
[image4]: ./output_images/perspective_transform.png "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/output.png "Output"


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the "Correct Distortion" section of the IPython notebook located at "./P4.ipynb" in function cal_undistort().  

I start in the section above titled "Find Corners" by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images.  Using the matrix and distortion coefficients, I call `cv2.undistort()` to correct the image:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in sections titled: "Apply Sobel," "Magnitude and Gradient," "Direction of the Gradient," "HLS and Color Thresholds," and "Color and Gradient").  Here's an example of my output for this step:

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `compute_binary_warped()`, which appears in section titled "Binary Warped Image" of my IPython notebook.  The `compute_binary_warped()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points (which if set as global variables).  I chose the pick the source and destination points in the following manner:

```python
height = img.shape[0]
width = img.shape[1]

x_top_left = width * 0.46
x_top_right = width * 0.54
y_top = height * 0.63

x_bottom_left = width * 0.155
x_bottom_right = width * 0.845
y_bottom = height

corners_src = np.array([[[x_top_left, y_top]],\
                        [[x_top_right, y_top]],\
                        [[x_bottom_left, y_bottom]],\
                        [[x_bottom_right, y_bottom]]])

top = 0
bottom = 720
left = (int)(width/2 - height/2)
right = (int)(width/2 + height/2)

corners_dst = np.array([[[left, top]],\
                        [[right, top]],\
                        [[left, bottom]],\
                        [[right, bottom]]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 589, 454      | 280, 0        | 
| 691, 454      | 1000, 0       |
| 198, 720      | 280, 720      |
| 1082, 720     | 1000, 720     |

The advantage of this scaled approach versus hardcoding is that it works for different camera resolutions.  I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In the "Find the Lines" section of my IPython notebook, I used the functions `first_window_search()` and `window_search()` to find the lane line pixels using the sliding window search method.  I then fit my lane lines with a 2nd order polynomial like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

In the "Measure Curvature" section of my IPython notebook, I used the function `measure_curvature()` to compute the radius of curvature.  I did this by first plugging the maximum y value of my lane line (the value at the bottom of the image) back into the 2nd order polynomial equation.  I then converted from pixels back to meters.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the "Unwarp Lines" section using functions `unwarp_lines()` and `find_offset()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The approach I used to lane finding was to use HLS Saturation thresholding combined with Sobel x thresholding.  This method worked quite well for the project video except at 41 seconds when the car goes over the bridge that is shadowed by the tree.  During this time my lanes get a little wobbly.  If I were going to pursue this project further, I could improve this by combining another threshold mask that worked well in these lighting conditions.
