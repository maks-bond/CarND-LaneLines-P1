# **Finding Lane Lines on the Road** 

[//]: # (Image References)

[image1]: ./examples/1.jpg "1"
[image2]: ./examples/2.jpg "2"
[image3]: ./examples/3.jpg "3"
[image4]: ./examples/4.jpg "4"
[image5]: ./examples/5.jpg "5"
[image6]: ./examples/6.jpg "6"
[image7]: ./examples/7.jpg "7"
[image8]: ./examples/8.jpg "8"
[image9]: ./examples/9.jpg "9"
[image10]: ./examples/10.jpg "10"

### Reflection

### 1. Pipeline description

The pipeline to draw lane lines on an image consists of the following steps:
1) Select region of interest where lane lines are likely to be in an image    
![image1]
2) Convert an image to grayscale.
This step involves conversion of an image to HSV format.
Then we split HSV image into three layers and work with last 'Value' layer.
![image2]
We normalize it to span [0, 255] range and apply mask to select areas with high brightness.
Such an algorithm of image conversion to grayscale was created to solve 'challenge' video which has areas with high brigthness and such areas caused problems when using normal RGB to Gray conversion. See below for more details on problems with 'challenge' video.  
3) Blur grayscale image obtained from step 2 with 5X5 gaussian kernel.
![image3]
4) Apply canny edge detector with lower threshold of 150 and high threshold of 300. Such arguments have been tuned based on experimentation on some images.  
![image4]

5) Apply houhg transform to get lines  
![image5]

6) Filter out lines to preserve only those which have slope appropriate to left and right line. The slope for left line is around 0.67 and the slope for right line is around -0.67. Such slope values have been tuned during experimentation.  


7) If there are some lines for left and right lane. Find top and bottom points for such lines. Top point is selected as intersection of left and right lines. Bottom point is just in the bottom of the image.  
![image6]

### 2. Improvements for diffenet levels of brightness
Originally the pipeline was implemented in the same way as above, however step #2 was implemented as simple RGB to Gray conversion.  
First and second test videos have produced good result of lane lines detecion. However challenge video showed some problems.  
In particular images with high brightness failed to detect lane lines. Challenge video has such images in the middle of the video.  

Such problem was solved in the following way:  
process_image function was extended to save an image whcih failed lane lines detection.
Then such images have been used to analyze the reason for the failure. 
It was found that RGB to grayscale conversion causes problem for such images as grayscale value of the line is similar to grayscale value of the road. That caused edge detection to fail.  
  
Different ways of color image to gray conversion have been tried: 
* Selection of different color channels (red, green or blue)
* Conversion of an image to LAB color space and using each layer.
* Conversion of an image to YCrCb color space and using each layer.
* Conversion of an image to HSV color space and using each layer.

As a result of such experimentation last layer of HSV color space showed the best results. It was visible when looking at gray scale image of that layer that lane lines have different color value comparing to its background.
Also to make edge detection even more robust, range mask was applied to select values in the [215, 255] range.
It seems like yellow or white lane lines have higher brightness in all the images whcih explains the success of such approach.  
Here are some image examples:  
1) Image which failed line detection with normal RGB to GRAY conversion: 
![image7]  
2) Pure Grayscale version:
![image8]

As we can see the yellow lane has very similar pixel values to its background, which fails edge detection.
  
3) 'Value' channel of HSV color space:  
![image9]
As we can see such grayscale image is much better as left yellow line has  higher contrast with the road which wlould cause to have higher gradient during canny edge detection.  
  
4) Masked HSV 'Value' channel:  
![image10]

It was observed that lanes have high brightness, as a result after application of a mask, the lanes are seen to be distinct from the road which makes line detection even more robust.  

Such improvements have helped to make better line detection on challenge video.


### 3. Shortcomings and potential improvements

Shortcomings:  
1) When we select region of interest we assume that lines are in particular area of the image. That requires the camera to be always positioned in the front center of the car
2) We tested the algorithm on the videos and images with relatively similar brightness and similar color of the lines. We might have problems at night or during rainy weather when brightness is not as high.
3) Algorithm works on the road with perfectly drawn lane lines. However that is not the case in all the roads. Sometimes lane lines are old and not very visible. Sometimes there are some other random lines drawn on the road maybe because of some road construction.
4) We select lines after hough transform using some specific line slopes. That requires the camera to be similarly calibrated and similarly positioned as in the test images. Different camera calibrartion or different camera position might cause problems.
5) In the high traffic conditions lane lines might be not visible at all because of other cars in front. That will cause problems detecting lines.
6) In the video it is visible that lane lines detection is shaky. It would be better if lines would always be stable.
7) Our algorithm draws straight lines and doesn't detect the curve of the road

Possible improvements:
1) To stabilize lines in the video we could average results from previous video frames.
2) We could fit the polynomial of higher order to detect the curve of the road. Also potentially we could run hough transform which detects curvy lines.
3) In case no lines are detected we can just fallback to resutls from previous video frames to increase the probablity of line detection.
4) Would be great to build machine learning model which would allow to tune the parameters for hough transform, canny edge detection, slopes of the left and right lines. For that we would require some training set of images with correctly detected lines.