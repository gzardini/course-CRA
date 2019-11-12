# Preliminaries {#cra-loc-prelim status=ready}

Excerpt: Instructions on how to setup your workflow for the exercise.

## Required steps

### Run the exercise

Run the exercise container:

    laptop $ docker -H ![DUCKIEBOT_NAME].local run --name lane_following_cra2 --net host -v /data:/data duckietown/lane-following-cra2:daffy

This container runs an extended version of the lane following demo from `dt-core`. It includes additional parameters which are important for this exercise.


### Run rviz

`rviz` (ROS visualization) is a 3D visualizer for displaying sensor data and state information from ROS. More on information can be found in the official [ROS wiki](http://wiki.ros.org/rviz)

For this exercise `rviz` will be helpful for displaying sensor messages from the Duckiebot. By selecting the appropriate topic we can output desired information.

<figure>
<img style="width:30em" src="images/rosviz_screenshot.png"/>
</figure>

First, make sure that the your display can be accessed from a container. Run:

    laptop $ xhost +local:root
    
Note: When you are done with the exercise, you should run the reverse command in order to secure your screen access again:

    laptop $ xhost -local:root

To start `rviz` run the following container:

    laptop $ docker run -it --net=host -e VEHICLE_NAME=![DUCKIEBOT_HOSTNAME] --env="DISPLAY" --volume="$HOME/.Xauthority:/root/.Xauthority:rw" duckietown/rviz-cra2:daffy-amd64 /bin/bash

then:

    laptop-container $ export ROS_MASTER_URI="http://![DUCKIEBOT_IP]:11311"

and also:

    laptop-container $ export ROS_IP=![DUCKIEBOT_IP]

finally we can launch the application:

    laptop-container $ rviz

After starting `rviz` we need to add the required topics we want to inspect

* `/![DUCKIEBOT_NAME]/duckiebot_visualizer/segment_list_markers`
* `/![DUCKIEBOT_NAME]/lane_filter_node/belief_img`
* `/![DUCKIEBOT_NAME]/lane_pose_visualizer_node/lane_pose_markers`

After adding these 3 topics, `rviz` should show the output as in the figure above.

### Change rosparams

The following functions will be useful to change the dynamic parameters in the exercises:

    laptop $ dts start_gui_tools ![DUCKIEBOT_NAME]

1) Listing the parameters:

    duckiebot-container $ rosparam list

2) Getting the parameters:

    duckiebot-container $ rosparam get ![PARAMETER_NAME]

3) Setting the parameters:

    duckiebot-container $ rosparam set ![PARAMETER_NAME] ![VALUE]
