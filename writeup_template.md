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

[image1]: ./output_images/undistorted_test6.jpg "Undistorted"
[image2]: ./test_images/test6.jpg "Road Transformed"
[image3]: ./ouptut_images/combined_binary_test6.jpg "Binary Example"
[image4]: ./output_images/perspective_transform_test6.jpg "Warp Example"
[image5]: ./output_images/processed_image_test6.jpg "Fit Visual"
[image6]: ./output_images/.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in `Advanced Lane Finding.ipynb`

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. This implemenation has been based off of the camera calibration example from udacity/CarND-Camera-Calibration.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to a test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to test image 6:
![alt text][image2]

The result for a distortion corrected image is above, at figure1.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of HLS and gradient thresholds to generate a binary image (thresholding steps can be seen in the sixth code cell of `Advanced Lane Finding.ipynb`).  Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform()`, which appears in the eighth code cell of `Advanced Lane Finding.ipynb`.  The `perspective_transform()` function takes as inputs an image (`img`), and internally defines source (`src`) and destination (`dst`) points.  I chose to hardcode the source and destination points in the following manner:

```python
src = np.float32([[240,720],[580,450],[710,450],[1160,720]])
dst = np.float32([[300,720],[300,0],[900,0],[900,720]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 580, 450      | 300, 0        | 
| 240, 720      | 320, 720      |
| 1160, 720     | 900, 720      |
| 710, 450      | 900, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I wrote a function, `find_lanes()` to do some other stuff and fit my lane lines with a 2nd order polynomial. The result of determining the lane curvature, and fitting the lane back onto the undistorted image is this:

![alt text][image5]

`find_lanes()` is based on a function I wrote for the Lane Finding project at the beginning of the term. The ouputs are two arrays left and right, as well as the y values associated. These three are 720 values long.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in code cell 15 of `Advanced Lane Finding.ipynb` by evaluating a factor of meters per pixel (two numbers recommended in the lessons at 15.35. This function also contains a way to measure the car centers distance from the center of the lane. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code cell 13 in `Advanced Lane Finding.ipynb` in the function `fit_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

The image has an overlay in the top left displaying the left and right curvatures as well as the distance from the center of the lane.

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_video/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

While playing back the project_video I noticed that at :41 there is a shadow across the road from a tree, and for that section of road lasting ~1 second, the lane lines warbled and jittered. Aside from that the pipeline seems effective.

I attempted to run the pipeline on the challenge video, but the edge detector was confused by the difference between pavement and asphalt, resulting in an extremely jittery and inaccurate representation of the lane. The harder_challenge_video resulted in errors as well. 

One way to fix this would be some function within the edge detector to enforce a certain pixel distance between the two lane lines. This could be coupled with a certainty factor for each lane. Using these two would ensure that lane lines are determined accurately.