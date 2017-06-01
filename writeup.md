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

**Figure 1.** *Gridline Reference.*

Once points were chosen for the perspective transform, the transform was tested on a calibration image.

![alt text][image2]

**Figure 2.** *Calibration Image.*

![alt text][image3]

**Figure 3.** *Perspective Transform.*

Once the perspective transform was functional, it was time to mask the image and determine what was navigable terrain and where any rock samples were. The function provided only used a lower bound for the color threshold, but I found it was necessary to also include an upper bound for rock sample detection. The upper and lower bounds of RGB values chosen for terrain mapping were: (160, 160, 160) and (255, 255, 255). The upper and lower bounds of RGB values chosen for sample detection were: (90, 90, 0) and (160, 160, 75). Below are examples of both masks applied to an image.

![alt text][image3]

**Figure 4.** *After Perspective Transform, Before Color Mask Applied.*

![alt text][image6]

**Figure 5.** *Perspective Transform with Navigable Terrain Mask Applied.*

![alt text][image9]

**Figure 6.** *Perspective Transform with Rock Sample Mask Applied.*

#### 2. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 
The first step in creating the process_image() function was to define the source and destination points for the perspective transform. I used the same values that were used in the Perspective Transform section of the test notebook. Next, the perspective transform function was applied to the input image. 

Once the overhead view was created it was time to determine which parts of the image were navigable terrain, rock samples, or obstacles. Because I was interested in 3 separate things I decided to use 3 different color thresholds to create 3 different masked images. The navigable terrain and rock sample masks used the threshold values described in the Perspective Transform section; the obstacle mask used a lower bound of (0, 0, 0) and an upper bound of (100, 100, 100).

After the color masks were applied it was time to use those images to create a world map. For each of the color masked images a rotation was first applied to the pixels, using the rover's yaw value to rotate the pixels to a reference point where the rover's yaw would be 0. Next the pixels were placed on the world map using the rover's position vector and an appropriate scaling factor.

![alt text][image8]

**Figure 7.** *Top left: Rover Camera View.
Top Right: Perspective Transform Applied to Rover Camera View.
Bottom Left: Color Mask Applied to Perspective Transform.
Bottom Right: Masked Terrain Pixels After Coordinate Transform. Arrow Shows Mean Angle of Navigable Terrain.*

Finally the image processing function is run on a set of images saved from a test run of the rover. 
**Video output can be found under:
RoboND-Rover-Project/output/test_mapping_own_data.mp4**


### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.

The skeleton of the perception_step() function was created using the process_image() function from the test notebook. The difference is that in perception_step() the image, position, and orientation data need to be coming from the Rover object where as the process_image() function takes that data from a Databucket object populated from the csv file a test run of the rover generates. The biggest difference between the Rover and the Databucket objects is that the Rover object store the x and y position data in a 2-tuple, while the Databucket object has separate arrays for x position and y position.

Once the perception_step() function was created the rover could navigate autonomously in a way that would very nearly pass the requirements of the assignment. There were several challenges facing the rover that were addressed in the decision step.

  1. Initially the rover would run into walls or obstacles and sometimes get stuck. By increasing the distance at which the rover initiates stopping in front of a wall or obstacle from 50 to 100 pixels I was able to improve this somewhat.
  
  2. The change in (1) improved the rover's performance, but it would still get stuck in certain circumstances. When there is a rock ahead of the rover but a clear path on either side of the rock, the vector lengths of the navigable terrain would be long enough that the rover's stop_forward distance would not be triggered. If the angles of the navigable terrain were roughly equal on either side of the obstacle, the rover would happily drive straight into the obstacle. To fix this, I changed the way the rover detected forward clearance. Instead of using the entire range of navigable terrain, I created a function that only took into account the length of the navigable terrain vectors within 15 degrees to the left or to the right of the rover. Using a narrow field of view to detect obstacles made the rover less likely to not detect an obstacle in its path.
  
  3. Fidelity would start out high and slowly creep towards 60%. I found that by increasing the lower bound of the color threshold for navigable terrain to (175, 175, 175) the fidelity was improved without noticeably affecting navigation.
  

#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

**Rover Settings:**
**Resolution: 1920 x 1080**
**Graphics Quality: Fantastic**
**Frames per second: 29**

Based on the way the drive_rover.py and decision.py files were already set up, once the perception.py file was completed the rover was very near to passing the project requirements. With the tweaks to the decision_step() and perception_step() functions described above the rover performance was improved to the point where it reliably fulfilled the project requirements.

There were several improvements I wanted to make but was not able to due to time constraints. Below is the roadmap I created to fulfill the challenge requirements:

  1. Prevent rover from getting stuck in circles or ignoring a narrow passageway in favor of a larger but already mapped passageway. I planned to tackle this by having the rover drop "breadcrumbs," and altering the steering angle to have a preference for directions without breadcrumbs. A breadcrumb trail could also be useful for returning to the starting position.
  
  2. Have the rover pick up samples. Telling the rover to pick up samples when it is near them would be fairly trivial. The interesting part of the problem is having the rover navigate to the samples and pick them up. In cases where there is only one sample in view I think it would be fairly easy to override the normal navigation, point the rover to the sample, and drive to it. If more than one sample was in view however, logic would have to be added to make sure the rover handles sample pickup one at a time. An especially difficult edge case might be if the rover sees two samples, drives to the first sample, but there is an obstacle directly between the first sample and the second sample. Now the rover couldn't just use the sample locations for navigation, it would need to drive around the obstacle in between.
  
  3. Ensure 100% of the terrain is mapped. As is the rover will map out large swaths of the terrain but it will miss small alcoves in the passages. My plan to tackle this was to look at the edges of the map and when the navigable terrain was not completely surrounded by obstacle terrain. This would let the rover find any gaps in its map, at least on the edges. I'm not sure how to handle any unmapped terrain between obstacles in the center of the map.
  
  4. Return to the starting position. I added code to have the rover record its starting position. My vision of the 'go-home' algorithm was to have the rover follow its breadcrumbs out of whatever passage it was in until there was a straight line of navigable terrain between it and the starting location.


