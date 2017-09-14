# **Finding Lane Lines on the Road** 

## Writeup

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consists of the following steps:
1) `color_mask` function
    - converts original image to HSV for better yellow color detection
    - converts original image to HLS for better white color detection
    - returns original image with both yellow and white masks applied
2) `grayscale`, `gaussian_blur`, `canny` - in this order
3) `region_of_interest` - (roi)
    - only keep output of 2) that is in roi
    - roi is determined by taking a polygon with two tuneable parameters,
    VOFFSET and HOFFSET, which determines the vertical and horizontal offsets
    from center for the upper corners of the polygon
4) `hough_lines` - returns list of detected lines in roi
5) `draw_lines`, which does the following:
    - buckets lines by slope
    - for lines in each bucket, finds average parameters for a single line
    - finds the longest negative-sloping line in roi and the longest
    positive-sloping line in roi
    - for edge-case where the two lines from above step intersect in roi, find
    the intersecting point and adjust roi to not include it
    - finally, draw the two lines and return them

The entire pipeline described above is implemented in `LaneFinderPipeline.process_image`.

Another cool feature about the pipeline is video smoothening, which is taking the past N_FRAMES and averaging their left and right lanes in order to produce a less shaky video output. This is done by keeping state in the `LaneFinderPipeline` class.


### 2. Identify potential shortcomings with your current pipeline

The current pipeline only detects straight lines. This works pretty well on
long stretches of highway, but contain many real-world edge cases. The pipeline
would perform poorly or fail altogether on curves or sharp turns.

Also, the current pipeline depends pretty heavily on the roi staying in
approximately the same place on each image frame.
This stability gives us the prominent negative and positive sloping lanes and
the road being centered in the image frame. This stability is not guaranteed in
the real world, as camera angles can change and hills can increase/decrease roi
dramatically.

Finally, the current pipeline is optimized to detect white and yellow, which we
assume are the lane colors. This is a pretty restricting feature, as some roads
may not have any prominent lane markers that are white or yellow and may be
marked instead by grass or dirt, for example. Similarly, the current pipeline may
falsely detect white or yellow barriers as lane markers, which is undesired as
well.


### 3. Suggest possible improvements to your pipeline

In the shortcomings section, I mentioned:
- non-straight lines
- dynamic roi selection (instead of fixed for each image)
- going beyond yellow/white markers

In the scope of this project, given more time I probably could have further
improved my roi selection algorithm. As seen in the `challenge_video` results,
the fixed roi selection does not yield accurate results when there is a curve
in the road. The lanes detected and drawn often go beyond the actual lane
markers on the street. Currently, my roi selection is mostly fixed, with
a possible adjustment given to detected lane intersections. This adjustment
already has shown great improvements in the `challenge_video` results, but
a step further might be detecting more precisely where the straight lane ends
and curve starts in the image. 

Another task that could yield incremental improvements would be to more
systematically tune the various parameters to the pipeline. For my current set
of parameters, I mostly did a quick trial-and-error, and seeing how good I can
get the results before giving up and accepting them. A more serious tuning of
parameters could come a long way.

