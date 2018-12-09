# Project 02 : Finding Lane Lines on the Road

## 01. Finding Lane Lines on the Road

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


## 02. Reflection

### 01. The draw_lines() function

My pipeline consisted of 5 steps. First, I converted the images to grayscale, then I applied Gaussian smoothing with kernel 5, then I applied a Canny transform to identify edges, then I applied a polygonal mask to the image in order to select the desired area of the image by excluding unwanted areas such as the sky and lateral objects. I then applied the Hough transform and defined the lines of the edges and finally I drew the lines over the image.

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by separating the points over the x-axis into 2: the right x's and the left x's based on whether the x-coordinates or both point were both in the first half of the image or the second half of the image.

I then used the polyfit function to determine the slope "m" and the intersect "b" the right and the left line. I also calculated the end points at the bottom of the image and I calculated the vertex of the triangle where the two lines meet (which is the hypothetical point where the two lanes meet at the horizon).

Below is how I modified the draw_line() function:

```
def draw_lines(img, lines, color=[255, 100, 100], thickness=10):
    """
    NOTE: this is the function you might want to use as a starting point once you want to
    average/extrapolate the line segments you detect to map out the full
    extent of the lane (going from the result shown in raw-lines-example.mp4
    to that shown in P1_example.mp4).  

    Think about things like separating line segments by their
    slope ((y2-y1)/(x2-x1)) to decide which segments are part of the left
    line vs. the right line.  Then, you can average the position of each of
    the lines and extrapolate to the top and bottom of the lane.

    This function draws `lines` with `color` and `thickness`.    
    Lines are drawn on the image inplace (mutates the image).
    If you want to make the lines semi-transparent, think about combining
    this function with the weighted_img() function below
    """

    img_x_center = img.shape[1] / 2
    right_lines_x = []
    right_lines_y = []
    left_lines_x = []
    left_lines_y = []


    for line in lines:
        x1,y1,x2,y2 = line[0]

        # Separate right from left lines
        if np.minimum(x1, x2) > img_x_center:
            right_lines_x.append(x1)
            right_lines_x.append(x2)
            right_lines_y.append(y1)
            right_lines_y.append(y2)

        elif np.maximum(x1, x2) < img_x_center:
            left_lines_x.append(x1)
            left_lines_x.append(x2)
            left_lines_y.append(y1)
            left_lines_y.append(y2)

    if len(right_lines_x) > 0:
        right_m, right_b = np.polyfit(right_lines_x, right_lines_y, 1)  # y = m*x + b
        draw_right = True
    else:
        right_m, right_b = 1, 1
        draw_right = False

    if len(left_lines_x) > 0:
        left_m, left_b = np.polyfit(left_lines_x, left_lines_y, 1)  # y = m*x + b
        draw_left = True
    else:
        left_m, left_b = 1, 1
        draw_left = False


    # Find 2 end points for right and left lines, used for drawing the line
    # y = m*x + b --> x = (y - b)/m
    y1 = img.shape[0]
    x = (right_b - left_b) / (left_m - right_m)
    y2 = left_m * x + left_b

    right_x1 = (y1 - right_b) / right_m
    right_x2 = (y2 - right_b) / right_m

    left_x1 = (y1 - left_b) / left_m
    left_x2 = (y2 - left_b) / left_m


    # Convert calculated end points from float to int
    y1 = int(y1)
    y2 = int(y2)
    right_x1 = int(right_x1)
    right_x2 = int(right_x2)
    left_x1 = int(left_x1)
    left_x2 = int(left_x2)


    # Draw left and right lines
    if draw_left:
        cv2.line(img, (left_x1, y1), (left_x2, y2), color, thickness)

    if draw_right:
        cv2.line(img, (right_x1, y1), (right_x2, y2), color, thickness)
```


### 02. Potential shortcomings with the current pipeline

One potential shortcoming would be what would happen when there are two parallel lines on the road close to each other. This is the case of lines on the side of the road where the first line is the line separates the carriageway from the emergency lane and the second line separates the emergency lane from the curb. Both of these lines are on the left of the image and the interpolation algorithm will try to interpolate over both.

Another shortcoming could be any type of wall separating lanes going in different directions. Walls could also have strong gradients and appear in the Canny transform as a line.


### 03. Possible improvements to the pipeline

A possible improvement would be to take the line which is the "closest" to the center of the image.

Another potential improvement could be to interpolate over different points in the same region of the image so that we can identify 2 parallel lines on the left of the image for example.
