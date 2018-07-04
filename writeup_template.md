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

[undistort_output]: ./examples/undistort_output.png "Undistorted"
[undistorted_example]: ./examples/undistorted_example.png "Road Undistorted Example"
[gradient_thresholded]: ./examples/gradient_thresholded.png "Binary Example"
[perspective_transform_example]: ./examples/perspective_transform_example.png "perspective transform example"
[sliding_window_example]: ./examples/sliding_window_example.png "sliding_window_example"
[ploting_back_lane_example]: ./examples/ploting_back_lane_example.png "ploting_back_lane_example"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

The code for this step is contained in the third code cell of the IPython notebook located in "./Advanced-Lane-Lines.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][undistort_output]

### Pipeline (single images)

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][undistorted_example]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at cell № 10 `./Advanced-Lane-Lines.ipynb`).  Here's an example of my output for this step.  

![alt text][gradient_thresholded]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform()`, which appears in cell 12  in the file `./Advanced-Lane-Lines.ipynb`   The `perspective_transform()` function takes as inputs an image (`original_img`).  I chose not to hardcode the source and destination, but to use use relative positioning using percentages: 

```python
    img_size = (img.shape[1], img.shape[0])
    bottom_width    = .75
    top_width       = .1
    height_trim     = .62
    bottom_trim     = .925
    
    
    img_size = (img.shape[1],img.shape[0])
    src = np.float32([[img.shape[1]*(.5-top_width/2),img.shape[0]*height_trim],
                      [img.shape[1]*(.5+top_width/2),img.shape[0]*height_trim],
                      [img.shape[1]*(.5+bottom_width/2),img.shape[0]*bottom_trim],
                      [img.shape[1]*(.5-bottom_width/2),img.shape[0]*bottom_trim]])
    offset = img_size[0]*.25
    dst = np.float32([[offset, 0],
                      [img_size[0]-offset, 0],
                      [img_size[0]-offset, img_size[1]],
                      [offset ,img_size[1]]])
```

This resulted in the following source and destination points:
Source Points:
[[ ]
 [  ]
 [ ]
 [         ]]
Destination Points:
[[]
 [ ]
 [ ]
 [ ]]

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 576 ,         446.3999939     |320,    0| 
| 704,          446.3999939     |960,    0|
| 1120,         666            | 960,  720|
| 160,          666            | 320,  720|

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][perspective_transform_example]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

To identify lane-line pixels I used sliding window method. This method was implemented in function `sliding_windows_with_fit_polynomial()`. This method use hiostogram to indicate the x-position of the base of the lane lines, and use it as  a starting point for lane lines search. From that point a sliding window was placed around the line centers, to find and follow the lines up to the top of the frame.

![alt text][sliding_window_example]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Calculation curvature and offset of the car from center of the lane  was done  in functuin `radius_of_curvatured` at cell   № 190 in the file `./Advanced-Lane-Lines.ipynb`.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this in the function `draw_lane()` af cell № 192.  Here is an example of my result on a test image:

![alt text][ploting_back_lane_example]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My implementation use sliding window search every new frame, it is the simplest implementation. In future I want to implement seach of lane line on new frame from position where lane was on previous frame. Also should smoothing lane between frames should be appliend. On the hardest road condition this algorithm will not work. Also gradient threshold should be better tuned for pictures of the road with shadows.
