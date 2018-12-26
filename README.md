
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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Step 1: Camera Calibration - Distortion Correction

The code for this step is contained in the second code cell of the IPython notebook located in "./examples/final.ipynb"

OpenCV is most popular library in computer vision.In that functions findChessboardCorners and calibrateCamera are the backbone of the image calibration
Arrays of object points, corresponding to the location (essentially indices) of internal corners of a chessboard, and image points, the pixel locations of the internal chessboard corners determined by findChessboardCorners, are fed to calibrateCamera which returns camera calibration and distortion coefficients. These can then be used by the OpenCV undistort function to undo the effects of distortion on any image produced by the same camera.Generally, these coefficients will not change for a given camera  even lens.

First step is to prepare "object points", which will be the (x, y, z) coordinates of the chessboard corners.
	
Assumption - chessboard is fixed on the (x, y) plane at z=0 such that the object points are the same for each calibration image
`objpoints` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  	

[image1]: ./output_images/Camera Calibration/1.png "1"

[image2]: ./output_images/Camera Calibration/2.png "2"

[image3]: ./output_images/Camera Calibration/3.png "3"

[image4]: ./output_images/Camera Calibration/4.png "4"

[image5]: ./output_images/Camera Calibration/5.png "5"

[image6]: ./output_images/Camera Calibration/6.png "6"

[image7]: ./output_images/Camera Calibration/7.png "7"

[image8]: ./output_images/Camera Calibration/8.png "8"

[image9]: ./output_images/Camera Calibration/9.png "9"

[image10]: ./output_images/Camera Calibration/10.png "10"	

[image11]: ./output_images/Camera Calibration/11.png "11"	

[image12]: ./output_images/Camera Calibration/12.png "12"	

[image13]: ./output_images/Camera Calibration/13.png "13"	

[image14]: ./output_images/Camera Calibration/13.png "14"	
	
Second step, the locations of the chessboard corners were used as input to the OpenCV function calibrateCamera to compute the camera calibration matrix and distortion coefficients.

Finally, the camera calibration matrix and distortion coefficients were used with the OpenCV function undistort to remove distortion from highway driving images. 

[undistrot]: ./output_images/Camera Calibration/undistrot.png "undistrot"	

If we compare the two images, there are differences between the original and undistorted image around edges, indicating that distortion has been removed from the original image.

### Step 2: Perspective Transform
The goal of this step is to transform the undistorted image to a "birds eye view" of the road which focuses only on the lane lines and displays them in such a way that they appear to be relatively parallel to eachother.
There is opencv function getPerspectiveTransform  and warpPerspective  which take a matrix of four source points on the undistorted image and remaps them to four destination points on the warped image. The source and destination points were selected manually by visualizing the locations of the lane lines on a series of test images

In the Jupyter notebook, in the Fifth and sixth code cells from the top. The unwarp() function takes as inputs an image (img), as well as source (src) and destination (dst) points. I chose to hardcode the source and destination points in the following manner:


# define source and destination points for transform
src = np.float32([(575,464),
                  (707,464), 
                  (258,682), 
                  (1049,682)])
dst = np.float32([(450,0),
                  (w-450,0),
                  (450,h),
                  (w-450,h)])
				  
Assumtion : Road will be flat and camera position relatively not change.


Image

[image15]: ./output_images/Bird eye view/1.png "Bird Eye View"	

### Step 3: Apply Binary Thresholds
In this step I attempted to convert the warped image to different color spaces and create binary thresholded images which highlight only the lane lines and ignore everything else. I found that the following color channels and thresholds did a good job of identifying the lane lines in the provided test images:

I selected the L channel of the HLS color space to isolate white lines and the B channel of the LAB colorspace to isolate yellow lines

The L Channel from the HLS color space, with a min threshold of 220 and a max threshold of 255, did an almost perfect job of picking up the white lane lines, but completely ignored the yellow lines.

The B channel from the Lab color space, with a min threshold of 190 and an upper threshold of 255, did a better job than the S channel in identifying the yellow lines, but completely ignored the white lines.

I chose to create a combined binary threshold based on the above mentioned binary thresholds, to create one combination thresholded image which does a great job of highlighting almost all of the white and yellow lane lines.

This is implemented In the Jupyter notebook, 9 and 10 code cells from the top. 

Image

[image16]: ./output_images/Combined/1.png "Combined Image"	

### 4. Lane-line pixels and fit their positions with a polynomial?

The functions sliding_window_polyfit and polyfit_using_prev_fit, which identify lane lines and fit a second order polynomial to both right and left lane lines.
This is implemented In the Jupyter notebook, 12,13 14 code cells from the top. 

Computes a histogram of the bottom half of the image. 
Finds the bottom-most x position of the left and right lane lines. (We can also identified from the local maxima of the left and right halves of the histogram, but I changed these to quarters of the histogram just left and right of the midpoint. This helped to reject lines from adjacent lanes).
Identifies ten windows from which to identify lane pixels, each one centered on the midpoint of the pixels from the window below. 
Pixels belonging to each lane line are identified and the Numpy polyfit() method fits a second order polynomial to each set of pixels. The image below demonstrates how this process works:

[image16]: ./output_images/PolyFit/1.png "Poly Fit"

The polyfit_using_prev_fit function performs basically the same task, but it leveraging a previous fit and only searching for lane pixels within a certain range of that fit. 

### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature is based upon this https://www.intmath.com/applications-differentiation/8-radius-curvature.php  and calculated in 16th cell.
Other Reference : http://mathworld.wolfram.com/RadiusofCurvature.html
					https://www.math24.net/curvature-radius/

curve_radius = ((1 + (2*fit[0]*y_0*y_meters_per_pixel + fit[1])**2)**1.5) / np.absolute(2*fit[0])

In this example, fit[0] is the first coefficient (the y-squared coefficient) of the second order polynomial fit, and fit[1] is the second (y) coefficient. y_0 is the y position within the image upon which the curvature calculation is based (the bottom-most y - the position of the car in the image - was chosen). y_meters_per_pixel is the factor used for converting from pixels to meters. This conversion was also used to generate a new fit with coefficients in terms of meters.
The position of the vehicle with respect to the center of the lane is calculated with the following lines of code:

lane_center_position = (r_fit_x_int + l_fit_x_int) /2
center_dist = (car_position - lane_center_position) * x_meters_per_pix
r_fit_x_int and l_fit_x_int are the x-intercepts of the right and left fits, respectively. This requires evaluating the fit at the maximum y value (719, in this case - the bottom of the image) because the minimum y value is actually at the top (otherwise, the constant coefficient of each fit would have sufficed). The car position is the difference between these intercept points and the image midpoint (assuming that the camera is mounted at the center of the vehicle).

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The final step in processing the images was to plot the polynomials on to the warped image, fill the space between the polynomials to highlight the lane that the car is in, use another perspective trasformation to unwarp the image from birds eye back to its original perspective, and print the distance from center and radius of curvature on to the final annotated image.

In the Jupyter notebook, in the 18 cells from the top

[image16]: ./output_images/draw_lane/1.png "Draw Lane"


---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [video1] ./project_video_output.mp4 "Video"

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

There are major two challanges that i faced
		Applying Binary ThreashHold
		calculating the radius of curvature of the lane and the position of the vehicle
I have improved performance in processing image by doing following 		
	My goal in developing a video processing pipeline was to create as smooth of an output as possible. To achieve this, I created a class for each of the left and right lane lines and stored features of each lane for averaging across frames.The video pipeline first checks whether or not the lane was detected in the previous frame. If it was, then it only checks for lane pixels in close proximity to the polynomial calculated in the previous frame. This way, the pipeline does not need to scan the entire image, and the pixels detected have a high confidence of belonging to the lane line because they are based on the location of the lane in the previous frame.

There are some assumption that we made while developing this
	There are no major up and down on road, means road will be flat
	Camera position will not change.
	I selected to average the coefficients of the polynomials for each lane line over a span of 10 frames but this can cause problem/fail.
	
