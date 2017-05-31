## Project: Search and Sample Return
### Writeup Template: You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---


**The goals / steps of this project are the following:**  

**Training / Calibration**  

* Download the simulator and take data in "Training Mode"
* Test out the functions in the Jupyter Notebook provided
* Add functions to detect obstacles and samples of interest (golden rocks)
* Fill in the `process_image()` function with the appropriate image processing steps (perspective transform, color threshold etc.) to get from raw images to a map.  The `output_image` you create in this step should demonstrate that your mapping pipeline works.
* Use `moviepy` to process the images in your saved dataset with the `process_image()` function.  Include the video you produce as part of your submission.

**Autonomous Navigation / Mapping**

* Fill in the `perception_step()` function within the `perception.py` script with the appropriate image processing functions to create a map and update `Rover()` data (similar to what you did with `process_image()` in the notebook). 
* Fill in the `decision_step()` function within the `decision.py` script with conditional statements that take into consideration the outputs of the `perception_step()` in deciding how to issue throttle, brake and steering commands. 
* Iterate on your perception and decision function until your rover does a reasonable (need to define metric) job of navigating and mapping.  

[//]: # (Image References)

[image1]: ./calibration_images/example_grid1.jpg
[image2]: ./calibration_images/example_rock1.jpg
[image3]: ./calibration_images/perspective_transform.png
[image4]: ./calibration_images/camera_view.png
[image5]: ./calibration_images/overhead_view.png
[image6]: ./calibration_images/rgb_thresh.png
[image7]: ./calibration_images/terrain_view.png
[image8]: ./calibration_images/rover_coords.png
[image9]: ./calibration_images/sample_thresh.png

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.
First an image with the gridlines was chosen to define the perspective transform. Source points were chosen on the corners of the gridline in the image, and mapped to a 10 pixel by 10 pixel square of output points.

![alt text][image1]
Figure 1. Gridline Reference.

Once points were chosen for the perspective transform, the transform was tested on a calibration image.

![alt text][image2]
Figure 2. Calibration Image.

![alt text][image3]
Figure 3. Perspective Transform.

Once the perspective transform was functional, it was time to mask the image and determine what was navigable terrain and where any rock samples were. The function provided only used a lower bound for the color threshold, but I found it was necessary to also include an upper bound for rock sample detection. The upper and lower bounds of RGB values chosen for terrain mapping were: (160, 160, 160) and (255, 255, 255). The upper and lower bounds of RGB values chosen for sample detection were: (90, 90, 0) and (160, 160, 75). Below are examples of both masks applied to an image.

![alt text][image3]
Figure 4. After Perspective Transform, Before Color Mask Applied.

![alt text][image6]
Figure 5. Perspective Transform with Navigable Terrain Mask Applied.

![alt text][image9]
Figure 6. Perspective Transform with Rock Sample Mask Applied.

#### 2. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 
The first step in creating the process_image() function was to define the source and destination points for the perspective transform. I used the same values that were used in the Perspective Transform section of the test notebook. Next, the perspective transform function was applied to the input image. 

![alt text][image4]
Figure 7. An Input Image from the Rover Camera View.

![alt text][image5]
Figure 8. The Rover Camera View after the Perspective Transform is Applied.

Once the overhead view was created it was time to determine which parts of the image were navigable terrain, rock samples, or obstacles. Because I was interested in 3 separate things I decided to use 3 different color thresholds to create 3 different masked images. The navigable terrain and rock sample masks used the threshold values described in the Perspective Transform section; the obstacle mask used a lower bound of (0, 0, 0) and an upper bound of (100, 100, 100).

![alt text][image5]
Figure 9. Perspective Transform with Color Mask Applied. Mask for Navigable Terrain only shown.

Once the color masks were applied it was time to use those images to create a world map. For each of the color masked images a rotation was first applied to the pixels, using the rover's yaw value to rotate the pixels to a reference point where the rover's yaw would be 0. Next the pixels were placed on the world map using the rover's position vector and an appropriate scaling factor.

![alt text][image8]
Figure 10. Navigable Terrain Pixels Transformed from Rover Perspective to World Perspective. Arrow indicates Average angle of Navigable Terrain.

### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.


#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  



![alt text][image3]


