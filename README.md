# Project 4 - Advanced Lane Finding

The steps of this project are as following:
- Step 1: Distortion correction using a calculated camera calibration matrix and distortion coefficients
- Step 2: Perspective transform to warp the image to a birds eye view perspective of the lane lines
- Step 3: Color thresholds to create a binary image which isolates the pixels of lane lines
- Step 4: Identify lane lines and fit polynomials to boundaries
- Step 5: Compute radis of curvature and vehicle position from center
- Step 6: Warp the detected lanes back to the original image
- Step 7: Display lane lines and estimation of curvature and vehicle position

### Step 1 - Distortion Correction
Use `findChessboardCorners` and `drawChessboardCorners` to identify the locations of corners on pictures of a chessboard of various angles.

![png](./output_images/corners.png)

I wrote cal_undistort() function to distort the image, which uses OpenCV function `calibrateCamera` to compute the camera calibration matrix and distortion coefficients, given locations of the chessboard corners, then uses OpenCV function `undistort` to remove distortion from images, given the calibration matrix and distortion coefficients.

![png](./output_images/undistorted.png)

### Step 2: Perspective Transform
The goal of this step is to transform the undistorted image to a "birds eye view" of the road. I wrote the transform() function in which the OpenCV functions `getPerspectiveTransform` and `warpPerspective` are used to take four source points on the undistorted image and map them to four destination points on the warped image. The source and destination points were selected manually by visualizing the locations of the lane lines on various test images.

![png](./output_images/warped.png)

### Step 3: Binary Threshold
The function threshold() is defined to convert the warped image to different color spaces and create binary thresholded images which try to focus on detecting the lane lines. Specifically, I use the following combined color channels and thresholds to identify the lane lines:

The B channel from the Lab color space, with threshold of (150,215), which identifies yellow lines well but ignores white ones.
The L Channel from the LUV color space with threshold of (210,255), which identifies white lines well but ignores yellow ones.

![png](./output_images/binary.png)

### Steps 4: Fit a polynomial to lane lines

- Identify peaks in the histogram to determine location of lane lines.
- Identify all non zero pixels close to histogram peaks using `numpy.nonzero()`.
- Fit a polynomial to lanes using `numpy.polyfit()`.

![png](./output_images/fit.png)

### Steps 5: calculate radius of curvature:
I wrote a function curvature(left_fit,right_fit) to calculate the raidus of curvature, which basically followed the course codes as follows:

```
ym_per_pix = 30./720 # meters per pixel in y dimension
xm_per_pix = 3.7/700 # meteres per pixel in x dimension
left_fit_cr = np.polyfit(lefty*ym_per_pix, leftx*xm_per_pix, 2)
right_fit_cr = np.polyfit(righty*ym_per_pix, rightx*xm_per_pix, 2)
left_curverad = ((1 + (2*left_fit_cr[0]*np.max(lefty) + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
right_curverad = ((1 + (2*right_fit_cr[0]*np.max(lefty) + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])
```

### Steps 6: calculate vehicle position from the center of road
I wrote a function distance(left_fit,right_fit) to calculate the postion.
- Calculat get the midpoint of the lane `mid_lane = (left_fitx[0]+right_fitx[0])/2`
- Calculat the distance from center: `distance=abs((mid_image - mid_lane)*xm_per_pix) `
- Also multiplying the number of pixels by `xm_per_pix=3.7/700`.

### Step 7: Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

Plot the polynomials on to the warped image, fill the space in-between to highlight the lane. Use perspective trasform again to warp the image from birds eye back to original perspective. Lastly print the distance and radius of curvature to the annotated image. The actual codes are inside the process_frame() function.

![png](./output_images/fill.png)

## Video Processing
The video processing pipeline is basically a repetation of step 7 from frame to frame, contained in process_frame() function. To make the output smooth, I averaged the coefficients of the polynomials for each lane line among 10 frames. Specifically, Line class is defined and instances for both the left and right lane lines are created to store necessary attributes. The pipeline also searches in a nearby region for previously found lane lines and searches the entire image (blind sliding windows) if no previous lines were found.

The original video can be accessed here: 

The project video: https://drive.google.com/open?id=0B8g4mCBBmkoaajFKdTVaalZxNms

The challenge video: https://drive.google.com/open?id=0B8g4mCBBmkoaeVRfeVlpNTJtS3M

### Reflections
This is by far the most complicated project in the program, it deals with a lot of programming details and trying to modularize my codes seems to be painful, which makes it a little difficult to debug. Further work could be done for decoupling the codes in the future.
 
This project pipeline has a relatively robust performance on both the normal and challenge videos. It is also capable of dealing with shadows in the second video. However, it does not perform very well on the even harder video, the filled lane jittered severely as the brightness is quite high on most of the images, which implies that the choosing of color space and other theresholding parameters should be more robust in order to deal with various brightness/weather conditions.


