##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[undistort1]: ./output_images/undistort1.png "Undistorted"
[undistort2]: ./output_images/undistort2.png "Undistorted"

[bin3]: ./output_images/bin3.png "Binary Image"
[bin4]: ./output_images/bin4.png "Binary Image"
[bin5_color_and_others]: ./output_images/bin5_color_and_others.png "Binary Image"
[bin1_yellow]: ./output_images/bin1_yellow.png "Binary Image - Yellow lane retained"



[bin_transformed1]: ./output_images/bin_transformed1.png "Transformed Binary Image"



[warped3]: ./output_images/warped3.png "Warped Image"
[polyfit3]: ./output_images/polyfit3.png "Polyfit Image"
[filled_lane]: ./output_images/filled_lane.png "Filled lane Image"





[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first and second code cells of the IPython notebook Advanced-Lane-Finding.ipynb. I followed the steps given below:

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objPoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgPoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objPoints` and `imgPoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to some images using the `cv2.undistort()` function and obtained this result: 

![alt text][undistort1]


###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.

I apply the distortion correction to the test images using `cv2.undistort()` and applying the camera calibration matrix and distortion coefficients obtained in the previous step. The code is in the 3rd code cell of the IPython notebook Advanced-Lane-Finding.ipynb. The results are shown below:

![alt text][undistort2]

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

In the 5th code cell the IPython notebook Advanced-Lane-Finding.ipynb, I used color gradient thresholds to produce a binary image. Here's an example of my output for this step:

![alt text][bin3]

The yellow lane is not detectd by other methods while the color gradient threshold does. Thus, in order to capture both white and yellow lane lines, I then combined Sobel gradient, magnitude and direction, and color thresholds in the 9th code cell. An example of the result is below:

![alt text][bin5_color_and_others]

![alt text][bin1_yellow]



####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform()`, which appears in the 5th code cell of the IPython notebook Advanced-Lane-Finding.ipynb.  The `perspective_transform()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])

```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][warped3]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

My code for identifying lane-line pixels and polynomial fitting is located in the 11th code cell of the IPython notebook Advanced-Lane-Finding.ipynb.

I used histogram of lower half of the binary image to locate the 2 starting points of the lane lines (left and right). Then by using sliding window technique, I located lane-line pixels by identifying nonzero pixels within sliding windows. If the number of pixels in the current window is large enough (above a minimum threshold), recentering of the next window is done by averaging the pixels in the current window.

Then I performed a polynomial fitting on the found lane-line pixels and obtained a result like this visualization:

![alt text][polyfit3]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the 12th code cells of the IPython notebook Advanced-Lane-Finding.ipynb.

I first run an second order polynomial fitting to obtain `left_fit` and `right_fit` coefficients of the left and right lane lines. Then I used the following equations to calculate the curvature of the lane lines:

`y_eval = np.max(ploty)
left_curverad = ((1 + (2*left_fit[0]*y_eval + left_fit[1])**2)**1.5) / np.absolute(2*left_fit[0])
right_curverad = ((1 + (2*right_fit[0]*y_eval + right_fit[1])**2)**1.5) / np.absolute(2*right_fit[0])
`

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the 13th and 14th code cell of the IPython notebook Advanced-Lane-Finding.ipynb. Here is an example of my result on a test image:

![alt text][filled_lane]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

At the beginning, my code didn't work well on yellow lane lines. Then I realized that I hadn't used gradients on other color spaces. I then added L-channel gradients and got bette results.

However, for frames with shadows, binary images are very noisy and this makes it harder to detect lane lines by using only histogram of nonzero pixels.

