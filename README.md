## Advanced Lane Finding Project

****

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
[image2]: ./test_images/test1.jpg "Normal Road"
[undist_image2]: ./output_images/test_undist_2.jpg "Road Transformed"
[image3]: ./output_images/test_binthresh_4.jpg "Binary Example"
[image4]: ./output_images/warped_perspective.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/overlayed_frame.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the Jupyter notebook named "Advanced_Lane_Finding.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![Original vs Undistorted][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![Test image][image2]
Undistorted output:
![Undistorted image][undist_image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (3rd code cell in the Jupyter notebook).  Here's an example of my output for this step. 

![Binary Threshold Gradient][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transformappears in lines 12 through 16 in the 4th code cell of the Jupyter notebook.  The warping takes as inputs an unistorted binary image (`bin_undist`), as well as source (`src`) and destination (`dst`) points.  I chose to hardcode the source and destination points in the following manner:

```python
    src = np.float32([[240,675],[580,460],[700,460],[1040,675]])
    dst = np.float32([[320,720],[320,0],[960,0],[960,720]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 580, 460      | 320, 0        | 
| 240, 675      | 320, 720      |
| 1040, 675     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I used a sliding window approach on histogram of x-pixels of a warped binary image. The histogram peaks represent where most of the vertical lines' x-pixels lie. Having found the base of the line, I kept moving my window/area of interest upwards till end of image. This gave me the bunch pixels that constitue the lane in the image.I fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the middle of the 2nd cell from the last in my Jupyter notebook.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in final 50 linesin the 2nd from last code cell in the Jupyter notebook.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_video/project_video_overlayed.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In the video processing pipline, I started by undistorting the input image.

Here I used sliding histogram approach to find the general area where lane pixels lie. For further frames, a simple fit was performed based on parameters gathered from first frame and line pixels were estimated. I used s-channel color thesholding and Sobel operator to find gradient in x-direction. This helped me create nice binary images where the lanes popped out for further analysis. The current pipeline has minor problems when too many shadows appear. A better approach would be to try additional L-channel color threshold for images in LUV space for whites and B-channel in LAB space for shading and brightness variations.  Also, video processing can be made more robust by averaging over past n-frames data points.
