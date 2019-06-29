# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

The pipeline is complete for finding all the lane markers. Some parameter tuning is needed.

Specifically, 

my gaussian blur is 5

my canny threshold is [50,150]

And my region of interest is:

np.array([[(0,imshape[0]),(imshape[1] * (0.5 - midwidth), imshape[0] * (1 - height)), (imshape[1] * (0.5 + midwidth), imshape[0] * (1 - height)), (imshape[1],imshape[0])]], dtype=np.int32)

As shown below:
 
![alt text][logo]

[logo]: https://github.com/OnionMoeCat/udacity_self_driving_course/blob/master/lane_finding/test_images/regionOfInterest.png "region of interest"

And my parameters for Hough Lines are:

rho = 1 # distance resolution in pixels of the Hough grid
theta = np.pi/180 # angular resolution in radians of the Hough grid
threshold = 20     # minimum number of votes (intersections in Hough grid cell)
min_line_length = 15 #minimum number of pixels making up a line
max_line_gap = 5    # maximum gap in pixels between connectable line segments

btw, parameter tuning is boring. sgh.

The interesting part is figuring out the left and right lane from lane markings using extrapolation.

Firstly, I put all lines with a slope > 0.5 to right lane line collection, all lines with a slope < -0.5 to left lane to left lane line collection. Then, I use cv2.fitLine() function to calculate left lane line and right lane line.

To make this pipeline works for challenge video, I use the running average of left lane and right lane (represented by a point and a vector) instead of the lane directly cv2.fitLine(). Running average formula: 0.9 * last average + 0.1 * current. Running average improves the stability of lane marking in challenge video a lot. Also, if the current lane's direction deviates from average a lot (current lane direction * last average lane direction < 0.9), I just use the last average lane.


### 2. Identify potential shortcomings with your current pipeline

I assume the lane is a straight line. This is not the case in the challenge video, so the curve far ahead in the challenge video is not marked properly. 

Region of interests does work well when the road is curvy.

Hough line detection parameters may not work for other videos.


### 3. Suggest possible improvements to your pipeline

Don't assume the lane is straight. This could be done by finding gaps between lines and fill them. Actually, I tried this improvement out, but the lane marking at curve were often ignored by Hough Lines transform, so I don't see any improvement.
