# Preliminaries {#cra-loc-prelim status=ready}

Excerpt: Instructions on how to setup your workflow for the exercise 

## Required steps

### Run the exercise 

Run the exercise container:

    laptop $ docker -H ![DUCKIEBOT_NAME].local run --name lane_following_cra2 --net host -v /data:/data duckietown/lane-following-cra2:daffy

This container runs the lane following demo from dt-core. Additionaly it includes additional parameters which are important for this exercise.


### Run rviz

Rviz (ROS visualization) is a 3D visualizer for displaying sensor data and state information from ROS. More on information can be found here: http://wiki.ros.org/rviz

For this exercise rviz will be helpful to display sensor messages from the Duckiebot. By selecting the appropriate topic we can output desired information.

<figure>
<img style="width:30em" src="images/rosviz_screenshot.png"/>
</figure>

To start rviz run the following container

    laptop $ docker run -it --net=host --env="DISPLAY" --volume="$HOME/.Xauthority:/root/.Xauthority:rw" duckietown/rviz-cra2:daffy-amd64 /bin/bash

and then

    laptop-container $ export ROS_MASTER_URI="http://![DUCKIEBOT_IP]:11311"

and also

    laptop-container $ export ROS_IP=![DUCKIEBOT_IP]

finally we can do:

    laptop-container $ rviz

After starting Rviz we need to add the required topics we want to inspect

1. `/DUCKIEBOT_NAME/duckiebot_visualizer/segment_list_markers`
2. `/DUCKIEBOT_NAME/lane_filter_node/belief_img`
3. `/DUCKIEBOT_NAME/lane_pose_visualizer_node/lane_pose_markers`

After adding these 3 topics, rviz should show the output as in the figure above.

### Change rosparams

Go back to the terminal where you launched rviz (the one inside the gui_tools container). The following functions will be useful to change the dynamic parameters in the exercises:

1) Listing the parameters:

    container $ rosparam list

2) Getting the parameters:

    container $ rosparam get ![PARAMETER_NAME]

3) Setting the parameters:

    container $ rosparam set ![PARAMETER_NAME] ![VALUE]


