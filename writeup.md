# **Reflection: Finding Lane Lines on the Road**


## Marco Pennacchiotti


### Pipeline


The pipeline for finding lane lines includes the following steps:


1. `color_select()`: I applied color selection to retain only white and yellow pixels, i.e. the colors of the lane lines. The function `inRange()` of `cv2` has been used to select color ranges for both white and yellow.
2. `gaussian_blur()`: I applied Gaussian blur on the grayscaled image to suppress noisy pixels. I tried different parameter values for the kernel and I did not find much difference in the end results when keeping the kernel relatively small.
3. `canny()`: I applied Canny edge detection to detect edges indicating lane markings. By appliying a manual grid search, I noticed that a large range of values would work well across the project for both the low and the high threshold. However, when I ran the pipeline on the challenge.mp4 video I realized that careful parametrization was needed, as the road in the video contained many tire and asphalt markings that could have been erroneously detected with low thresholds. Also, bringing the thresholds too high would not allow to detect the left lane on the lighter portion of the road.    
4. `region_of_interest()`: I selected the trapezoidal portion of the image on which to perform lane detection.
5. `hough_image()`: In the last step I applied the Hough transform to find lane segments; then, I inferred the lane markings by applying linear regression to a subset of segments that were likely to be part of the left and right lanes.

The step that required more effort was step 5.

The parameter selection for the Hough transform was not trivial. Unlike Cunny edge detection, changing a parameter even slightly could have resulted in either including spurious segments or detecting no lane segments at all. The problem was even more evident in the challenge.mp4, for the same reasons exposed in step 4. It took a few iterations to get the parameters right.

Linear regression was initially applied to the output of Hough transform, separately for the left and right lane, by selecting segments with slopes respectively smaller and bigger than 0. It was immediately clear that linear regression was unable to fit the lane lines because of the presence of spurious segments. Even one only small spurious segment in the image would result in a wrong regression fit.

I therefore focused my attention of identifying and removing the spurious segments. My final solution was to use thresholding, by simply discarding segments whose slopes was too high or to small to be part of either the left or the right lane. I also tried more sophisticated methods for detecting such segments, all inspired by outlier detection techniques. One method calculated the mean and standard deviation of the slopes of the set of segments of a lane, and then discarded from the set those segments that were x standard deviations from the mean. Another method was to apply the outlier detection methods described in http://scikit-learn.org/stable/modules/outlier_detection.html to the spatial coordinates of all points of the segments.
Both methods did not perform better that the simple thresholding approach.

As for linear regression, I initially applied it to the two end points of all selected segments for a lane. This would result in good but not optimal result.
I found that the main reason was that longer segments should have been weighted higher since they contributed more points to the regression. I therefore applied linear regression weighting each end point by
the length of its line. This got better results. In the final system I however decide to use a more intuitive and natural approach. Instead of weighting end points, I explicitly used *all* points of a segment.
The results of the two approaches were  almost identical, and I therefore decided to go ahead with the latter solution.


### Shortcomings and Improvements

The solution I proposed relies on linear regression with degree 1, i.e. lanes can only be modeled by straight lines. This is a limitation because roads with curves, as in the challenge.mp4 video, cannot be modeled accurately. I have tried to solve this problem by allowing regression with a degree 2 polynomial fit. However this solution resulted in more problems than improvements, as parts of the videos with straight lanes would be subjected to alterations. An improvement that I did not explore is to actually model curves  starting with a set of predefined equations that describe prototypical curves.

Another shortcoming is that the predicted lane lines are not completely stable across frames in the video, as they have an evident jitter. This is probably due to the car wobbling slightly on the lateral axis due to unevenness of the road. One improvement for this shortcoming could be to smooth a frame's lane line by looking at the immediately previous frame, e.g. with a window of one or two seconds.

Finally, the solution is limited by the fact that only lanes of certain colors can be considered and that the overall environmental conditions in the videos are optimal. More sophisticated solutions are needed to make the solution more robust.  
