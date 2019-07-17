## Writeup

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

[image1]: ./camera_calout_undis/calibration2.jpg "Undistorted"
[image3]: ./test_images_out/straight_lines1_out.jpg "Binary Example"
[image4]: ./test_images_out/straight_lines1_draw_region.jpg "Warp Example"
[image5]: ./test_images_out/straight_lines1_warped_lane_marking.jpg "Lane region marked green"
[image6]: ./test_images_out/straight_lines1_warped_lane_marking_result.jpg "Output"
[image7]: ./test_images_out/straight_lines1_warped.jpg
[video1]: ./output_video/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image1]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. The code is in the 5th block.
My threshold are:
S channel threshold is s_thresh=(170, 255), Sobel X threshold is sx_thresh=(30, 100)
Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`. The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points. The wrapper code is in the 6th block. I chose the hardcode the source and destination points in the following manner:

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 568, 470      | 200, 0        | 
| 260, 680      | 200, 680      |
| 717, 470      | 1000, 0       |
| 1043, 680     | 1000, 680     |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]
![alt text][image7]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I get the lane points using the sliding window method:
- get the bottom half of the picture
- get the base of the left/right lane by getting the peak of histogram in left/right half of the picture
- the sliding window of the left/right lane is a region around the base of left/right lane 
- get all the points within the sliding window of the left/right lane
- get the left and right lane by fitting my lane lines with a 2nd order polynomial by using the polyfit() function. The polygon between left lane and right lane is painted in green in the picture below.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`
I used the formula provided in the lecture. The formula generate stable curvature for larger curves, but changes a lot across framess for straight lines. This is because straight lines have a much smaller `A`, so any changes in `A` make curvature changes a lot. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I used the same function `warper` to unwrap the image. And I defined a function `weighted_img` in the block 10 to apply the unwrapped image back.
Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

The additional work I take to make the pipeline is a class LaneMarker to store the lane information from the last frame. The reason is we want the lane to stay stable across frames when the algorithm does not work well with some frames. For each frame, we check the validity of its detected lane by the lane width in pixels and if the left and right lane are parallel. If the lanes are not valid, I replace the lanes with the result from last frame. This works well in the project video. 

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?


The approach I take here is very standard. I spent quite decent amount of time finding a good transform wrapping the picture. The pipeline will fail when there are other lines close to the lane. In the challenge video, the dark line in the center will be misidentified as the left lane. It would probably be solved by tweaking parameters in lane validation. Another issue with this pipeline is it does not work well with big turn since the sliding window assumes the existence of lane points from bottom to top. One solution: If there are not enough points in the sliding window and the window is at the border, we can drop all the points in the sliding window.

The formula calculating curvature is very sensitive to the change of the 2nd order parameters when the lane is straight. This causes the curvature unstable across frames. This could be resolved by setting a maximum curvature for a straight lane (e.x, 100000m), and substitute the curvature with 100000 * curvature / (100000 + curvature), which is close to curvature when curvature is small, and to 100000 when the curvature is large.
