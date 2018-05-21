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

[image1]: ./output_images/calibration1_undist.jpg "Undistorted"
[image2]: ./output_images/undistort_output.jpg "Road Transformed"
[image3]: ./output_images/binary_combo_example.jpg "Binary Example"
[image4]: ./output_images/warped_straight_lines.jpg "Warp Example"
[image5]: ./output_images/test1_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/test4_output.jpg "Output"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.

The code of this project is in IPython notebook located in "./code_XiaYang.ipynb".

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in code_cell_2.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

### Pipeline (test images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image.  Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()` at code_cell_8.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I set the source and destination points in the following manner:

```python
src = np.float32([[252,698],[573,465],[712,465],[1064,698]])
dst = np.float32([[412,720],[412,0],[888,0],[888,720]])
```

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 252, 698       | 412, 720        |
| 573, 465      | 412, 0      |
| 712, 465     | 888, 0      |
| 1064, 698      | 888, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

All Warped binary images were saved as output_images/warped_*.jpg.

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I used convolutions in conjunction with sliding window search to find the hot pixels belongs to the lane lines, and fit my lane lines with a 2nd order polynomial to those identified pixels kind of like this:

![alt text][image5]

Code to identify lane line pixels using convolution in conjunction with sliding window search is contained in find_lane_pixels(self, warped) method in LineTracker class (code_cell_10).
I then used np.polyfit() function to fit a 2nd order polynomial function to fit their positions.
In those procedures,  some statistics were calculated:
•  Horizontal difference of lowermost and uppermost point of polynomial fit from that of previous fit.
•  Vertical distance between lowermost and uppermost lane line pixel detected
•  Deviation of width of detected lane at nearest, halfway and farthermost  positions from standard width of a lane.
•  Number of points in the fit.
These checks are included in :
•  filter_fitx(self, fitx, margin) method in Line class
•  lane_sanity_checks(self) method in LineTracker class
These two class were contained in  code_cell_10.
These statistics can help determine whether the found line is good or not.
All images of line fits were saved as output_images/*_fit_lines.jpg.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did these in code_cell_15. I averaged the x values from both polynomial fit lines and then converted the pixel positions to x, y positions in real world using xm_per_pix and  ym_per_pix values. And then fit a polynomial function in this domain to get the radius of curvature of the road. I calculated the position of the vehicle with respect to center by taking the average of offset of both lines respect to center of the frame.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I generated top down view of the result plotted on warped road image in cell_code_10. I implemented this step in function map_lane in code_cell_14  to map the result back to undistorted image of the road..
Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output. Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to project video result](./output_videos/project_video_output.mp4)
Here's a [link to challenge video result](./output_videos/challenge_video_output.mp4)
Here's a [link to harder challenge video result](./output_videos/harder_challenge_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I noticed that sudden changes of lighting make it hard to consistently detect lane line pixels. It's necessary to examine more advanced filtering and brightness equalization techniques to make the pipeline more robust under sudden changes of lighting.
