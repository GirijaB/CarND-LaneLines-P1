

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

The goal of this project is to find lane lines on the road images.

The steps performed to find lane lines on road images were as follows:-

    1. Convert the image to grayscale.
    
    2. Canny Edge detection
    
    3.Region of Interest Selections.
    
    4.Hough Transform Line Detection.
    

![Alt text](/test_images/solidWhiteCurve.jpg?raw=true)


Convert the image to grayscale.


![Alt text](/examples/grayscale.jpg?raw=true)


Apply Canny Edge detection.

low_threshold = 20
high_threshold = 150
edges = canny(blur_gray, low_threshold, high_threshold)

![Alt text](/examples/canny_threshold.png?raw=true)


Mark region of interest on Canny Edge image by drawing polygons and filling the edges in the region.

imshape = image.shape

vertices = np.array([[
            (0, imshape[0]),
            (imshape[1]/2 - 50,3*imshape[0]/5),
            (imshape[1]/2 + 50,3*imshape[0]/5),
            (imshape[1],imshape[0]),
        ]], dtype=np.int32)
        
![Alt text](/examples/masked.jpg.png?raw=true)



Then apply Hough Transform on the Canny Edges to identify lines from the edges and then processing them to identify the predominant linesand filling that on the edges. 

rho = 1

theta = np.pi/180

threshold = 1

min_line_length = 10

max_line_gap = 5

line_image = hough_lines(masked_image, rho, theta, threshold, min_line_length, max_line_gap)

![Alt text](/examples/hough.jpg.png?raw=true)



Finally we map the detected lines on the imag.

![Alt text](/examples/laneLines_thirdPass.jpg?raw=true)


modified the draw_lines() ad fit lines function to aggregate and extrapolate the current set of lines generated. We have right and left lane and assume that the car drives in the center of the lane. To determine the lane lines we get the points that represent the lines from Hough transform and divide them according to their (approximate) position in the image.

Another approach is to divide the lines by positive and negative slope, except that the lines representing the edges of lanes detected on either side might have negative or positive slopes adding to noise when selection; In that case we can add a second filter by x position.

Once we have the points that belong to each of the lane lines we try to find the lane lines by fitting the points to a line with linear regression.

if #x_points and y_points are the lists of x and y coordinates of the points,
the slope and the intercept of the line are calculated as below:-
slope, intercept = np.polyfit(x_points, y_points, 1)  # 

    x1 = int((y1 - intercept) / slope)
    x2 = int((y2 - intercept) / slope)
    return x1, y1, x2, y2
    
From this line representation we can get the values for the top_x, top_y and bottom_x, bottom_y for the fitted line that represents the lane.

bottom_y is fixed, which is the height of the image which we can get from image.shape. 
The coordinate bottom_x is the x-coordinate of this point where the line would intersect with the bottom of the image,
which we can grab using the equation y = mx + b. 

In python this can be written as:
  def get_x_coordinate_for_y(y, slope, intercept):
      x = (y - intercept)/slope
      return np.uint32(x)


To get the top coordinates we calculate the minimum values for the y-coordinates evaluated on the line equation
polynomial = np.poly1d([slope, intercept])
line = polynomial(x_points)

Hence, minimum value in line would be top_y, and we can get top_x from the function get_x_coordinate_for_y.

Now that we have the top_x, top_y, bottom_x, bottom_y we can draw both the lanes with cv2.line on a blank image, which should give us the lane lines.


###2. Short Comings of this project:-

    1.The parameters for the region of interest are dependent on the the initial angle for the lane lines and their position in the image (decided by the position of the camera w.r.t. the road). Complex shaped roads and sharp turns might not return correct lanes.
    
    2.In real world scenarios the road lane lines can be worn out and faded and also here could be tire mars , leaves and many obstacles like dirt etc that will produce poor results.
    
    3.Another shortcoming is each set of lane lines determined in a image/frame of a video is independent of the previous frame. This can cause flicker due to changes in the frame lines or sudden changes in the line lengths or angle due to noise or other errors.

    
   
###3. Possible Improvements that can be done:-

    1.Possible improvement is to not consider image lines for the region of interest but consider doing some geometrical transformations to map curvature lines. 
    
    2.A possible way to reduce the flicker and smoothen the sudden changes in the lanes in the video is to remember previous N frames and aggregate the position of the lane lines.

