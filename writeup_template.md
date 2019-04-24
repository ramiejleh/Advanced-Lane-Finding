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

[image1]: ./examples/undistort_output.png "Undistorted Chess board"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/undistort_road_output.png "Undistorted Road"
[image4]: ./examples/binary_combo_example.jpg "Binary Example"
[image5]: ./examples/warped.jpg "Warp Example"
[image6]: ./examples/color_fit_lines_1.jpg "Fit Visual"
[image7]: ./examples/color_fit_lines_1.jpg "Fit Visual"
[image8]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

---

### Camera Calibration

#### 1. Briefly stating how I computed the camera matrix and distortion coefficients.

The code for this step is contained in the first code cell of the IPython notebook located in "./Advanced_Lane_Finding_Project.ipynb").  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![Undistorted Chessboard][image1]

### Pipeline (single images)

#### 1. Correcting the distortion of an image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![Road Image][image2]

I defined a correct_distortion function on line 36 of the 4th code cell that utilizes the `cv2.undistort()` function to undistort the image using the Distortion Coefficient and Camera Matrix I obtained from the camera calibration cell.

The result i got is this:
![Undistorted Road Image][image3]

#### 2. Using color transforms and gradients to create a thresholded binary image. 

I used a combination of color and gradient thresholds to generate a binary image (thresholding function `hls_sobel_combination()` is defined at lines 1 through 33 in the 4th code cell).  Here's an example of my output for this step.

![Combined Binary Thresholded Image][image4]

I converted the image to HLS then used a thresholded S channel, Which provides the clearer lanes across different conditions, and combined it with a thresholded Sobel X gradient to obtain a combined thresholded binary image with clear enough lanes to detect later in the project.

#### 3. Performeing a perspective transform.

The code for my perspective transform includes a function called `transform_perspective()`, which appears in lines 39 in the 4th code cell.  The `transform_perspective()` function takes as inputs an image (`img`), as well as tranform matrix (`M`), which i obtained using `getPerspectiveTransform()`on the `src` and `dst`, and image size (`img_size`) points.  I chose to hardcode the source and destination points in the following manner:

```python
src = np.float32([[600, 450], [690, 450], [1100, 680], [280, 680]])
dst = np.float32(
    [[offset_x, offset_y], 
    [img_size[0]-offset_x, offset_y], 
    [img_size[0]-offset_x, img_size[1]-offset_y], 
    [offset_x, img_size[1]-offset_y]]
    )
```

This resulted in the following destination points:

| Destination   | 
|:-------------:|
| 300, 0        |
| 980, 0        |
| 980, 720      |
| 300, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![warped image][image5]

I also prepared a `untransform_perspective()` using an inverse transform matrix (`Minv`) which i made using the `getPerspectiveTransform()` on the `src` and `dst` swicthed. 

#### 4. Identifying lane-line pixels and fit their positions with a polynomial.

This is done in the 5th code cell where i defined a `fit_polynomial()` functions that takes a `binary_warped` image as a parameter and in turn calls the `find_lane_pixels()` function which utilizes a histogram to identify where the lanes start then using a sliding window algorithm to detect the lanes rather accuratelyand returning them in order to fit my lane lines with a 2nd order polynomial kinda like this:

![Lane lines detcted image][image6]

![Lane lines detcted image][image7]

#### 5. Calculating the radius of curvature of the lane and the position of the vehicle with respect to center.

I defined a function in the 7th code cell `measure_curvature_real()` to measure the curvature.
First i define conversions in x and y from pixels space to meters
Then i search around the discovered lane using the `search_around_poly()` function, which is defined in the 6th code cell.
Then i define y-value where we want radius of curvature to be the maximum y-value, corresponding to the bottom of the image.
Lastly i calculate the R_curve (radius of curvature).

#### 6. An example final image output that previews the full pipeline.

I implemented this step in lines 23 through 55 in the 7th code cell. Here is an example of my result on a test image:

![Result Image][image8]

---

### Pipeline (video)

#### 1. The final output!

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.

My approach depends heavily on having steady lane lines with no long interruptions on both sides which is not optimal. 
It works with the output video because the video has reliable lines.
In the case of an interruption the line detection would fail. A way to solve this is predicting where the line would go using previous frames so at any moment where we lose the lane we just draw a coninuation of the last reliable detection we have. Also having a neural network predicting lanes would be of a help.

And since i am using a color and gradient threshold it means that in some lighting conditions where we lose the hight contrast between the lane line and the road, our detection fails again. Finding a more reliable thresholding technique might help here.
But in the case of the output video, the conditions are optimal and clear so the applied thresholding is quite sufficient.