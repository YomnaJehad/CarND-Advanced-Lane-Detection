## Project Writeup 

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
* Determine some margin around the detected lane lines to search around in future frames instead of searching in the whole frame.

[//]: # (Image References)

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"


[image6]: ./output_images/undistort_output.jpg "Output"
[image7]: ./output_images/undistorted5.png "Output"
[image8]: ./output_images/Color_space_grad_combin_test6.png "Output"
[image9]: ./output_images/pres_trans.png "Output"


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]
![alt text][image6]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this:
![alt text][image7]
After computing the camera calibration and the distortion coefficients, we send the original image to the function cv2.undistort() along with those parameters, so it returns the undistorted image.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

After a lot of testing on the test images, and comparing which combination of color and gradient thresholds gies better result of a binary image (thresholding steps in `P2- For Delivery.ipynb` in the third cell in functions get_abs_mag_dir_binary(), get_s_binary(), getCombinedBinary()).  
Here's an example of my output for this step.

![alt text][image8]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `persp_trans()`, which appears in `P2- For Delivery.ipynb` in the 4th cell lines 1 through 25.  The `persp_trans()` function takes as inputs an image (`img`).  I chose the hardcode the source and destination points after a lot of testing and comparing in the following manner:

```python
src=np.float32(
        [[195,img.shape[0]],   #Low_Left
         [1140,img.shape[0]],   #Low_Right
         [550,470],            #Up_Left
         [720,470]])           #Up_Right
    
    offset=300
    
    dst=np.float32(
        [[offset,img.shape[0]],
         [img.shape[1]-offset,img.shape[0]],
         [offset,offset],
         [img.shape[1]-offset,offset]])
```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image9]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in `P2- For Delivery.ipynb` in the 5th cell lines 1 through 30 in the function `unwarp()`. After computing the polynomials and getting the lanes correctly and all, this function draws a polygon to fill the area between the lanes, unwarps it using the function cv2.warpPerspective(), then weights it on the original image as shown:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
