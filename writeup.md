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

[video1]: ./project_video.mp4 "Video"


[image6]: ./output_images/undistort_output.jpg "Output"
[image7]: ./output_images/undistorted5.png "Output"
[image8]: ./output_images/Color_space_grad_combin_test6.jpg "Output"
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

For to do this we have two options after undoing the distortion, color space and gradient transform, the perspective transform: 
##### 1-
 We start from scratch getting the histogram of the frame which means we draw a diagram that implies where on the x axis, you can find the biggest number of nonzero (white) pixles, this basically means that at this x coordinate we probably have the start point of our lanes, so we get it, and start the window search method which is we draw a virtual window around this start point, and compute the number of nonzero pixels within it, if it exceeds a certain margin, we recenter it on the mean coordinate of these points, this way the window will keep moving up the frame and we will keep storing nonzero pixels of the lane, so eventually we will have lists of the lane pixels.
This process is done by the function `find_lane_pixels()` in`P2- For Delivery.ipynb` in the 6th cell lines 1 through 100.
So this function returns the detected pixels to get it in another function `advanced_fit_poly()` in the 8th code cell.
(you'll find 3 versions of this function `fit_poly()` used in the early image testing-you can ignore it, then `advanced_fit_poly()` which we will discuss now, and then `without_window()` which we will discuss when talking about the 2nd lane detection option )

So what `advanced_fit_poly()` does is that it gets the lane pixels colors them in Red for the Left Lane and in Blue for the Right Lane,finds a polynomial that fits them, Then it starts the calculations of the lanes' curatures, vehicle's position relative to the center, and the distance between the two lanes (For sanity checks that the detection is sane and acceptable so far), then the function calls the unwarpping function, unwarps the frame and weighs it onto the original frame, then prints out the texts that imply the calculations outputs.

##### 2-
The second option we have is, to start with option 1, and store some lane detection results (say for 20 frames or more or less), get the average/mean of these detections, add a margin, this will give us an average polynomial for each lane, and a margin around it (let's call this combination the green area) so in the green area, for each frame, we will most probably find our lanes, so the second option is, instead of searching randomly everywhere in each frame, we search in this part only, so instead of using the sliding window search method, we get the nonzero pixles which are in the green area, and as long as the resulted detections pass some sanity checks, this method should work just fine.
so what `without_window()` function does (which you will find in code cell no.9) is that it takes the averaged polynomial values, creates a green area, gets the nonzero pixels in it, fits a polynomial to them, and then calculates the curvature and the other calculations as in the `advanced_fit_poly()` function.

the sanity checks are checks like - Are both of the lanes curvature reasnobaly similar? do they make sense? are they quite parallel? - is the distance between the lanes reasonable? and so on.. as long as the detections pass these checks, we will add this result to our sane_detected_lanes[] list and make it an input to the green area calculations. (Whether it's detected from option 1 or option 2 since they passed the checks they should be added to the list)



![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in `advanced_fit_poly()` line 50 through 80 function and `without_window()` line 60 through 100.
I followed the student's method mentioned in the lesson, which is that we convert the polynomials' coefficients' to the real world dimentions first then we substitute in the curvature calculation equations which were mentioned in the lessons as well, 
parameters 
`my = 30/700 # meters per pixel in y dimension
 mx = 3.7/600 # meters per pixel in x dimension`
regarding the vehicle position, for the `advanced_fit_poly()` since we computed a histogram, i assumed that the starting points that were used as base points to the sliding window algorithm, they were the center of each lane line which i used to calculate the center of the lane, then assumed that the center of the frame is the vehicle's center as well, so it was easy to calculate the distance between the mid point between the two lane lines, and the vehicle's center. and also calculate the distance between the two lane lines.
while in the `without_window()` because we didn't compute a histogram, i assumed that the x coordinate corresponding to the biggest y coordinate in the polynomials, i assumed it's the center of each lane line, and then did the same as above.
 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in `P2- For Delivery.ipynb` in the 5th code cell lines 1 through 30 in the function `unwarp()`. After computing the polynomials and getting the lanes correctly and all, this function draws a polygon to fill the area between the lanes, unwarps it using the function cv2.warpPerspective(), then weights it on the original image as shown:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_video.mp4)
Here's a [link to my video result](https://classroom.udacity.com/nanodegrees/nd013-ent/parts/f114da49-70ce-4ebe-b88c-e0ab576aed25/modules/37588c52-f496-4568-898e-651cbd0e8fc1/lessons/22dced51-df3a-4679-b2bf-a5ca5cd91699/concepts/0a96d23f-6c22-4053-a7f6-83e12ce5a6ec#)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Issues faced in my project might be: 
- the bugs of the image dimensions in my code, sometimes weighing two images wouldn't work and there would be errors that mean that the two images' dimensions are not equal, it took me a while to understand what this means and to fix it.
- the rest were how to turn the pipeline in my head of the Look Ahead filter and the smoothing into actual code.
My pipeline may fail: 
 - probably if the car was moving up or down a hill, the perspective transform wouldn't work as expected.
 - if the camera is not implemented right in the center of the vehicle the calculations may fail.
 - if there's a great curve my sliding window algorithm wouldn't probably work.
To make it more robust:
 - instead of hardcoding the perspective transform parameters, make it dynamic which means through further image processing computations like detecting the parallelogram that the lanes form and get its corners then transform it.
 - instead of assuming that the camera is in the center of the vehicle we can make a starting code to futher calibrate it, like if it's not in the center calculate how much it's drifted and use this value in further computations.
 - instead of my sliding window algorithm, use the algorithm with the convolution method, it should be much more robust in extreme cases.

