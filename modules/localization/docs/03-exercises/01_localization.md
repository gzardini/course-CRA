# Exercises {#exercise-localization status=ready}

Excerpt: Understand the components of the imaging pipelime: from image to localization.

The goal of this exercise is to familiarize us with the imaging pipeline to localization, which is the base for the lane following demo.


<div class='requirements' markdown='1'>
  Requires: [Camera calibration](+opmanual_duckiebot#camera-calib)

  Requires: [Docker basics](+duckietown-robotics-development#docker-basics)

  Requires: [ROS basics](+duckietown-robotics-development#sw-advanced)

  Requires: [Knowledge of the software architecture on a Duckiebot](+duckietown-robotics-development#duckietown-code-structure)

  Results: Understand the trade-offs when dealing with image processing

  Results: Insights into the image pipeline of a duckiebot.
</div>

## Introduction

Determining the position is essential for any Duckiebot to survive in the city. In the following section we will go step by step through the various passages of the image pipeline: from image to position.

Fig. XX shows the 3 important of the localization:

<figure>
  <img style="width:30em" src="images/image_pipeline_overview.png"/>
</figure>


### Line detector node

* Line detector node is responsible for detecting line in front of the Duckiebot in three different colours: red, white and yellow.    
   
* Published by line detector node:

Segment list (type: SegmentList.msg) is an array which saves all segments (type: Segment.msg) found in the image. A segment consists of colour (red, yellow, white) and 2D vector (startpoint, endpoint)
    
### Lane filter node

* Lane filter node is responsible for estimating the reference position of the Duckiebot with respect to the lanes.    
  
* Published by lane filter node:
Lane pose (type: duckietown_msgs/lane_pose) is struct with the following parameters which are currently in use:
  
  1. d (float32) the lateral offset, where d = 0 is the middle of the right lane
  2. phi (float32) probably the angle

#### Bayes filter

Bayes filters are a probabilistic tool for estimating the state of dynamic systems. In the case of our Duckiebot, given a stream of observation (in this case the camera image) we compute a measurement likelihood matrix with a histogram filter. This enable us to calculate an initial belief. Then, we project the belief of the previous time step to the current time step. We can do this in the following way:

belief (t+1) = belief(t) * measurement_likelihood(t)

From the belief, we then extract the pose d and angle phi with the highest probability.

#### Histogram filter

each 2d white and yellow segments are projected onto the Duckiebots reference frame. Then the horizontal distance from the white/yellow segment to the desired position (middle point between right white lane and yellow lane) is calculated. The same is done for the angle phi. For all the segments a histogram is calculated which can be then display under as an image stream. 

To see it run

    laptop $ dts start_gui_tools

and then

    laptop-container $ rqt_image_view

select the topic : /studentbot09/lane_filter_node/ml



Snippet of the the generation of votes for the histogram filter

    #Generation of votes for the the histogram filter
    def generateVote(self, segment):
        p1 = np.array([segment.points[0].x, segment.points[0].y])
        p2 = np.array([segment.points[1].x, segment.points[1].y])
        t_hat = (p2 - p1) / np.linalg.norm(p2 - p1)

        n_hat = np.array([-t_hat[1], t_hat[0]])
        d1 = np.inner(n_hat, p1)
        d2 = np.inner(n_hat, p2)
        l1 = np.inner(t_hat, p1)
        l2 = np.inner(t_hat, p2)
        if (l1 < 0):
            l1 = -l1
        if (l2 < 0):
            l2 = -l2

        l_i = (l1 + l2) / 2
        d_i = (d1 + d2) / 2
        phi_i = np.arcsin(t_hat[1])
        if segment.color == segment.WHITE:  # right lane is white
           if(p1[0] > p2[0]):  # right edge of white lane
               d_i = d_i - self.linewidth_white
           else:  # left edge of white lane
               d_i = - d_i
               phi_i = -phi_i
           d_i = d_i - self.lanewidth        
           elif segment.color == segment.YELLOW:  # left lane is yellow
           if (p2[0] > p1[0]):  # left edge of yellow lane
               d_i = d_i - self.linewidth_yellow
               phi_i = -phi_i
           else:  # right edge of white lane
               d_i = -d_i
           d_i = - d_i

           weight = 1
           d_i += self.center_lane_offset

           return d_i, phi_i, l_i, weight

## Helpful tools

* Rviz 
    (ROS visualization) is a 3D visualizer for displaying sensor data and state information from ROS. More on information can be found here: http://wiki.ros.org/rviz
    For this exercise rviz will be helpful to display sensor messages from the Duckiebot. In order to run rviz run:By selecting the appropriate topic we can output desired information. In particular xxxx and xxx will be useful in this exercise.

    <figure>
    <img style="width:30em" src="images/rosviz_screenshot.png"/ >
    </figure>

    To start rviz run the following command

    laptop $ dts start_gui_tool rviz
  
* Rosparameter manipulation

    As a reminder:
    for rosparameter introspection and manipulation we recommend the following steps:

    laptop $ dts start_gui_tools DUCKIEBOT_HOSTNAME

  1) Listing the parameters:
    
    container $ rosparam list

  2) Getting the parameters:
    
    container $ rosparam get parameter_name

  3) Setting the parameters:
    
    container $ rosparam set parameter_name value


## Instructions


### Task 1: Line detector exercise

As previously introduced, the line_detector_node detects white, yellow and red segments. Due to computational constraints of the Duckiebot, a trade-off between the number of segments processed and computational resources allocated has to be made. The goal of this exercise is to analyze this trade-off by determining the relationship between the number of segments processed and the number of pose estimates that are being computed. For this task the parameter

- /BOT_NAME/line_detector_node/segment_max_threshold

can be dynamically adjusted.

    TODO: While varying the maximal number of segments record a rosbag of the lane_pose with its publishing frequency in Hz. Plot the relationship between the number of segments on one axis and the frequenz on the other axis. 



### Task 2: Lane pose exercise

As outlined in the introduction section, the lane_filter_node estimates the Duckiebot desired pose by means of recursive Bayes estimation. As a lever we can change the size of the belief/likelihood matrices. Therefore we are interested in analyzing the effect of various matrix sizes on the precision/standard deviation of the lane pose. For this task the parameter

- /BOT_NAME/lane_filter_node/matrix_delta_d
- /BOT_NAME/lane_filter_node/matrix_delta_phi

can be dynamically adjusted. 
    
    TODO: While varying the size of the belief/likelihood matrix record a rosbag of the lane_pose with its publishing frequency in Hz. Plot the relationship between the size of the belief/likelihood matrix and the publishing frequency in Hz.

### Task 3: English driver

One of our brave Duckiebots wanted to make a visit to Great Britain before Brexit happens. However, it needs to adhere to the local driving rules. Therefore it need to learn to drive on the left side of the road. 

    TODO: The task is to make the Duckiebot drive on the left side of the road. An appropriate parameter and the code snippet provided above is sufficient to finish this task.