## Project 4 - Advanced Lane Finding 

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

[image1]: ./output_images/undistorted.png "Undistorted"
[image2]: ./output_images/undistorted_lanes.png "Road Transformed"
[image3]: ./output_images/binary_example.png "Binary Example"
[image4]: ./output_images/warped.png "Warp Example"
[image5]: ./output_images/color_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/example_output.png "Output"
[video1]: ./project_video_out.mp4 "Video"

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the third code cell of the IPython notebook located in "./P4_code.ipynb".

I began by creating object points 'objpoints', which will be the (x, y, z) coordinates of the chessboard corners. It is assumed that the chessboard is on the x,y plane (z=0) 
and the object points are consistent between images. 'objp' is a reshaped array of coordinates that will be appended each time chessboard corners are found in an image.
'imgpoints' are the corners (x,y pixels) in the image plane that occur on successful chessboard corner detection and are recalculated for each image of the chessboard images using
OpenCV's 'findChessBoardCorners' function.

The 'objpoints' and 'imgpoints' are then input into OpenCV's 'calibrateCamera' function to calculate the camera matrix and distortion coefficients. 
OpenCV's 'undistort' function is used to undistort images using the calculated distortion coefficients and camera matrix. 
An example of undistortion applied to a chessboard image can be seen below.

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.
The following image is an example of undistortion applied to a test image 'straight_lines2.jpg'. The same steps were followed as above where the previously calculated camera matrix and distortion coefficients
were passed to OpenCV's 'undistort' function.
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
Red channel, U channel, and threshold gradients were applied to create a binary thresholded image. A red channel colour threshold was performed in RGB space as well as a U channel threshold in YUV colour space.
The red colour channel was thresholded to be between 220 and 255 for the upper and lower bounds respectively since yellow and white are both within this range. The U channel was thresholded
to be between 220 and 255 since this was able to capture the yellow lane line under various lighting conditions.
Likewise, the sobelx gradient bounds of 20 and 100 were found to be effective for capturing all lane lines and so wered used. Image values that were within either the sobelx or red or U channel threshold values 
were set to a binary value of 1 with all other values set to 0. The channel and gradient thresholding were both performed using the 
'color_gradient_transform'. A region of interest mask was applied using the 'region_of_interest' function to limit the information where warped lane lines were likely to be found.
Both of these functions are contained within the fourth cell of './P4_code.ipynb'. 
An example of a binary thresholded image using the above can be seen in the following figure.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

A perspective transform was used to transform the thresholded image into bird's eye view so that lane curvature could be more accurately analyzed. 
Source 'src' and destination 'dst' points were chosen to create a warping matrix M using OpenCV's 'getPerspectiveTransform' function. The warping
function 'warp_image' was used to perform warping on images given a matrix M and can be found in the fourth cell of './P4.ipynb'. 

Source and destination points were modified iteratively to produce a bird's eye view where the lane lines appeared straight for examples images 
'straight_lines1.jpg' and 'straight_lines2.jpg'. The following source and destination points were used to define the warping transform. 

```
src = np.float32([
        [685, 450],
        [1100,720],
        [595,450],
        [200,720],
        ])

dst = np.float32([
        [960,0], 
        [960,720], 
        [320, 0], 
        [320, 720],
        ])

```
An example of the warping transform applied to 'straight_lines2.jpg' can be seen below where the warped lane lines appear to be straight and parallel to eachother
following the transform.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Lane line pixels were determined either by searching for lane pixels within a certain proximity to the previous lanel lines if possible and by performing a new search if not.
In the case where a new search needed to be performed, a histogram of thresholded pixel counts along the x axis was taken with peaks representing the likely locations of the lane lines.
In order to determine the y-axis variation in the lane lines, y-axis values were incremented in n windows with their respective x positions searched within a specified proximity of pixels.
Once the representative pixel values were found, they were then fit with 2nd degree polynomials using a methodology similar to the image below. The code for finding new lane lines, or lane lines
based on previous lane positions can be found in the 'fit_lane_lines_new' and 'fit_existing_lane_lines' functions respectively in cell four of './P4.ipynb'.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Radius of curvature of the lane lines was calculated by first fitting a polynomial to the lane positions in meters. In order to perform the conversion from pixel space to meters,
the pixel values were multiplied by conversion factors based on knowledge of their real-world values. Once the polynomial was fitted, the radius of curvature was able to be calculated
from the equation using the equation given in the lecture material. An average of the left and right lane lines was used in the labels appended in the processed images and video.

The position of the vehicle with respect to center was calculated by first taking the midpoint between both the left and right lane lines on the warped image. This midpoint represents the
center of the lane in pixel space. The offset from center was calculated via subtraction of the center length of the image (the 'perceived' center in pixel space). This difference was then multiplied
by a conversion factor to determine the distance in meters.

Radius of curvature and position of vehicle with respect to center were calculated respectively using the functions 'calculate_radius' 
and 'calculate_distance_from_center' in cell 4 of './P4.ipynb'.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The pipeline of performing all required image transformations can be found in the 'add_lane_lines' function cell 5 of './P4.ipynb'. An example of the transform
applied to 'straight_lines2.jpg' can be seen below.

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_out.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The approach I took with this project was to produce a working model of the pipeline for a single test image. The process of choosing source and destination points, thresholding values, etc.
was a trial and error approach until there was an acceptable result for the straight line images 'straight_lines1.jpg' and 'straight_lines2.jpg'. Once the process worked for a single test image,
the approach was modularized and made into a pipeline so that the process could be easily applied to other images or video. This also allowed improvements to be made to the functions 
of the pipeline without breaking other stages. A line class was used at the suggestion of the course material to keep track of lines from different frames when processing videos.

Problems I had with my implementation of the project included determining proper source and destination points for the bird's eye view transform, determining effective ways to validate the
lane lines that were discovered, and methods of retaining only the useful information from the image so as not to retrieve erroneous lane lines. Determining an effective means of
smoothing the past n frames of lane lines was also a challenge. 

The current implementation of the pipeline is likely to fail in certain cases due to the challenges mentioned above. In certain cases, lighting changes, shadows, and other sources of gradients such as
shapes on the road coloured similarly to lane lines can cause jittering of lane lines in the project videos.
As well, the current implementation for validating that lane lines are parallel, that radius of curvature is reasonable, and that the lane lines are an acceptable distance 
from eachother is rather crude and oftentimes results in erroneous lines passing the validation tests. Changes in elevation also cause the line smoothing to produce innacurate results for
best estimates of line parameters.

In order to make the pipeline more robust, the images could be thresholded in a more optimized manner and using other colour spaces so that they are less sensitive 
to changes in lighting or other shapes present in or near lane lines.Changes in elevation could also be detected to prompt refitting of lane lines in this case. 
Augmenting the computer vision approach to lane detection with deep learning could also be used to more intelligently detect lane lines in images, though would be much more computationally expensive.

