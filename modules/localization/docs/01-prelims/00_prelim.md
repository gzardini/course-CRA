# Preliminaries {#cra-loc-prelim status=ready}

Excerpt: Instructions on how to setup your workflow for the exercise 

## Required steps

### Run the exercise 

Run the exercise container:

    laptop $ docker -H ![DUCKIEBOT_NAME].local run --name lane_following_cra2 --net host -v /data:/data duckietown/lane-following-cra2:daffy

This container runs the lane following demo from dt-core. Additionaly it includes additional parameters which are important for this exercise.


### Run Rviz

Rviz (ROS visualization) is a 3D visualizer for displaying sensor data and state information from ROS. More on information can be found here: http://wiki.ros.org/rviz

For this exercise rviz will be helpful to display sensor messages from the Duckiebot. By selecting the appropriate topic we can output desired information.

<figure>
<img style="width:30em" src="images/rosviz_screenshot.png"/>
</figure>

To start rviz run the following command

    laptop $ dts start_gui_tool ![DUCKIEBOT_NAME]

and then

    laptop-container $ rviz &

The ampersand is necessary since dts_start_gui_tools only supports one terminal and we need to launch rosparam from dts_start_gui_tools too. After starting Rviz we need to add the required topics we want to inspect

1. `/DUCKIEBOT_NAME/duckiebot_visualizer/segment_list_markers`
2. `/DUCKIEBOT_NAME/lane_filter_node/belief_img`
3. `/DUCKIEBOT_NAME/lane_pose_visualizer_node/lane_pose_markers`

After adding these 3 topics, rviz should show the output as in the figure above.


### Change rosparams 

Go back to the terminal where you launched rviz (the one inside the gui_tools container). The following functions will be useful to change the dynamic parameters in the exercises: 

1) Listing the parameters:

    container $ rosparam list

2) Getting the parameters:

    container $ rosparam get ![PARAMTER_NAME]

3) Setting the parameters:

    container $ rosparam set ![PARAMETER_NAME] ![VALUE]


