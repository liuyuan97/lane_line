
# Advanced Lane Finding Project

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

The code is gave in the jupyter notebook file of lane_lines.ipynb.  In order to help people to review, that juputer notebook file is also saved as a html file of lane_lines.html.
 
[//]: # (Image References)

[image1]: undis_0.png "Undistorted"
[image2]: color_fit_lines.jpg "Fit Visual"

[image3]: straight_lines1_proc.png "straight1"
[image4]: straight_lines2_proc.png "straight2"
[image5]: test1_proc.png "test1"
[image6]: test2_proc.png "test2"
[image7]: test3_proc.png "test3"
[image8]: test4_proc.png "test4"
[image9]: test5_proc.png "test5"
[image10]: test6_proc.png "test6"

[video1]: project_video_proc.mp4 "Video1"
[video2]: challenge_video_proc.mp4 "Video2"

### Camera Calibration

The code for this step is contained in the first code cell of the IPython notebook.  The "object points" are the (x, y, z) coordinates of the chessboard corners in the world. It is assumed that the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.   Since the actual chess board size is unknown, the chess board square size is taken as 1.   Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time successfully detecting all chessboard corners in a test image.  

`imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection using 'cv2.findChessboardCorners'.  However, the correct number of chess board square must be provided.

Then the output `objpoints` and `imgpoints` are used to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### Step 1: Obtain a distortion-corrected image

Using the cv2.undistort() function and distortion matrix obtained in the camera calibration to remove the camera distortion.

#### Step 2: Detect lane lines

The detection is performed over the HLS color space.  However, only L and S channel data are used.  For each channel, the angle, magnitude, and x-direction of the gradient are used.  Also, the channel data magnitude are also used to filter out some low value background data which could have the similar gradient information as the lane lines.  All of these threshold could be adjusted based on the field data.  For more information, please refer the gradient detection portion of the code.


#### Step 3: Apply a perspective transform to obtain a up-down image/bird view 
The image directly from the camera suffers from a projection distortion, which tends to make the lane line converge in the far distance.  Thus, a perspective transform has to be applied to convert the image to be the bird view which is equivalent to viewing the land directly from the top of sky.  The cv2.warpPerspective function is used to do such transform.  In order to use that function, the perspective transform matrix has to be provided.  This matrix is computed by cv2.getPerspectiveTransform function.  The working method of cv2.getPerspectiveTransform is to use image pixel location points from the source and destination image to compute.  At least four points have to be provided.  One of example is given as following:


| Source        | Destination   | 
|:-------------:|:-------------:| 
| 526, 498      | 200, 350      | 
| 760, 498      | 1000, 350     |
| 271, 672      | 200, 650      |
| 1034, 672     | 1000, 650     |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.


#### Step 4: Search lane lines, and curve fitting 

After the perspective transform, a binary image with the clear lane lines is ready for find the lane lines. First, a histogram method is used to find the rough lane line position in the image.  Then, two sliding window, placed around the left, and right line centers, to find and follow the lines up to the top of the frame.  Refer the line search code for more information.

The found lane lines are fitted to the second-order polynomial as shown in the following figure.

![alt text][image2]

#### Step 5: Compute the radius of curvature of the lane and the position of the vehicle with respect to center

Based on the formula of the radius of curvature, the fitting second order polynomial parameters can be used straightforward.  To compute the position of the vehicle with respect to center, the following two assumption are used:

* the lane width is 3.7 meter wide
* the image center is also the center of the vehicle

#### Results

The above static image pipeline is applied to several test images.  The results are shown as following.

![alt text][image3]     
---
![alt text][image4]         
---
![alt text][image5]   
---
![alt text][image6]    
---
![alt text][image7]    
---
![alt text][image8]    
---
![alt text][image9]    
---
![alt text][image10]
---

### Pipeline (video)

#### Line track algorithm
The video process pipeline follows the similar procedure as the static image.  However, the difference is that the consecutive video frame does not change significant frame by frame.  After doing the line search for the first frame, there is no need to do the line search for the following frame too.  The curve fitting algorithm could be applied directly to the range of the previous detection area, which is more computation efficient.

Also, the line detection from each frame could be averaged to obtain a more stable, and robust estimation.  Here, the simple exponential average is used.  However, in the average process, the track results have to be checked.  In case the track process fails, the lane line search algorithm has to be applied again to recover from the failure.

In some extreme case where the gradient detection almost fails due to very dark, shadowed lane area, the track algorithm does not update the line detection results, and only utilize the previous estimations.  That is to mimic the human being behavior.

#### The normal results
Here's a [link to the normal video result](project_video_proc.mp4).  That is relatively easy case.  The detection results are very good.

#### The challenge results
Here's a [link to the challenged video result](challenge_video_proc.mp4).  This video is quite difficult.  It has several area with gradient, and some shadow areas where there is very dark.  In general, the detection results are good.  However, the detection performance on the shadowed are is barely acceptable.  

---

### Discussion

This project implements the whole lane line detection algorithm.  The proposed algorithm pipeline works well in general.  However, there are still several areas needed to improve as 

* In the HLS color space, only L and S channels are utilized.  It needs to propose a scheme which also incoporate H channel information.
* The proposed gradient algorithm does not work well on the shadowed area.  The detection results from some image detection give out very few line points while the detection results from others produce many wrong points.
* It needs to propose an approach to remove the outliers.
* The proposed exponential average treats all polynomial coefficients equally.  It is need to understand these coefficients' contributions more, and propose a better filtering method.
