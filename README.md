# **Advanced Lane Finding**

## Project Writeup

### Nikko Sadural

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

[image1]: ./camera_cal/calibration2.jpg
[image2]: ./_calibration2_undistorted.jpg
[image3]: ./test_images/straight_lines2.jpg
[image4]: ./output_images/_undist_3.jpg
[image5]: ./output_images/_combined_binary255_3.jpg
[image6]: ./output_images/_warp_binary255_3.jpg
[image7]: ./output_images/_polyfit_7.jpg
[image8]: ./output_images/_final_image_7.jpg
[video1]: ./output_images/project_video_output.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

To calculate the camera matrix and distortion coefficients, I started off by initializing the `objp` array of object points that correspond to (x,y,z) coordinates in the real world (assuming an x-y plane where z=0). Then I created the arrays `objpoints` and `imgpoints` to store 3D real world points and 2D image plane points, respectively, from all the calibration images. With these arrays as inputs into the `cv2.calibrateCamera()` function, the camera matrix `mtx` and distortion coefficients `dist` were computed to apply to all images captured by the camera.

##### Distorted (original) image:
![alt text][image1]

##### Undistorted (corrected) image:
![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Distortion correction was calculated from the camera calibration step. The correction has been applied to each of the original test images to generate an undistorted copy of each image. An example of a distorted and undistorted image can be seen below.

##### Distorted (original) test image:
![alt text][image3]

##### Undistorted (corrected) test image:
![alt text][image4]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient threshold to generate a binary image. I created one pipeline for the test images and another pipeline for the project video. For the test image pipeline, my thresholding steps are found in lines 59-135. For the project video pipeline, my steps are in lines 13-58 using functions `changeColorToSaturationAndBinary()`, `sobelxAndBinary()`, `sobelyAndBinary()`, and `combineSaturationSobelxSobely()`. Here is an example of my output for this step.

##### Thresholded binary image example:
![alt text][image5]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I performed a perspective transform on the thresholded binary image from the previous step after applying region masks to remove any noise from between the lane lines and the front of the car. For the images, the masking and perspective transform are found in lines 137-175. For the video, they are found in lines 60-92 in function `imageMaskWarp()`. I hardcoded the `src` and `dst` points needed for the function `cv2.getPerspectiveTransform()` in lines 75-85 for the video pipeline. The masked and warped image was then generated using the perspective transform `M` as an input for the function `cv2.warpPerspective`. To verify the mask and transform were applied correctly, the lane lines below are shown to be parallel.

##### Masked and warped binary image example:
![alt text][image6]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

After masking and warping the thresholded binary image, I applied a sliding windows algorithm to detect the location of the parallel lane lines. The lane locations at the start of the image process are found by finding the arguments maximizing the histogram of summation of points along the x-axis of the image. The sliding window hyperparameters are hardcoded. For each of the following windows after the initial window is located, the location of the sliding window is then placed at the average x-position from the previous window. For the image pipeline, this can be seen in lines 177-244. For the video pipeline, this is found in lines 94-155 of the `fitQuadratic()` function.

After the lane lines were located, the stored values for left and right lane pixel positions in `leftx`, `lefty`, `rightx`, and `righty` are used to fit a second-degree polynomial using the `np.polyfit()` function. The quadratic coefficients are generated for each lane in `left_fit` and `right_fit`. Then x and y data were generated to plot the polynomial curves as a function of the y-coordinate `ploty`. For the image pipeline, this is found in lines 246-257. For the video pipeline, this is in lines 157-168 in the `fitQuadratic()` function.

An extra precaution was taken in case one of the lane line polynomials exceeded the width of the original image and caused an error. In that case, the lane line polynomial that exceeded the width of the image is replaced with the opposite lane line polynomial curve. For the images, this is in lines 259-267. For the video, this is in lines 170-178 of the `fitQuadratic()` function.

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

##### 2nd order polynomial curve fit example:
![alt text][image7]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature of each lane line was calculated in lines 277-291 for images and lines 182-198 for the video in the `detectLane()` function. The radius of curvature for each line was calculated using the given equation in the lesson using the quadratic coefficients stored in `left_fit` and `right_fit` arrays. It is important to note that a conversion was done from pixels to meters in order to obtain real world values from the image. The conversion factors used were `my = 30/masked_warped_image.shape[0]` to represent 30 meters per pixel, and `mx = 3.7/900` to represent 3.7 meters per 900 relevant pixels along the x-axis (between the lane lines at the bottom of the image).

The vehicle position with respect to the lane center was calculated in lines 293-300 for images and 200-207 for the video in the `detectLane()` function. It was assumed that the camera is placed at the center of the longitudinal axis of the vehicle, such that the distance from the center of the lane lines to half of the image width is the offset from the center of the lane, `center_offset`. If the offset is positive, then `offset_dir = 'right'`. If negative, then `offset_dir = 'left'`.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines 302-337 for images and lines 209-237 for videos in my function `detectLane()`. See an example image result below.

##### Lane detection image example with radius of curvatures and center offset:
![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result][video1]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The initial problem I faced was being able to detect the lane lines only using binary thresholds for the S channel of the HLS image and the Sobel x gradient. After tuning some of the threshold parameters, I decided to finally combine the Sobel y gradient into the final binary thresholded image. Another problem I faced was finding a good set of sliding window hyperparameters to be able to get better parallel, vertical curve fits for the straight-lane-line test images. One situation in which my pipeline could fail is if both left and right lane lines are curve-fit to exceed the width of the image. Since the pipeline only assumes that one of the lane lines could cause an error, if there were a sharp enough turn to cause both lines to exceed the image, then the result would be catastrophic. A similar error could also occur for lane change maneuvers or if a car were to enter the lane. One final improvement that could be made is to improve the radius of curvature calculation by starting with a more robust method of curve fitting lane lines such that they are parallel.
