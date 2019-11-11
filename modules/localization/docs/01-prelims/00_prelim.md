# Preliminaries {#cra-loc-prelim status=ready}

Preliminaries...


## Helpful tools

### The belief histogram

To see the belief histogram, run:

    laptop $ dts start_gui_tools

and then

    laptop-container $ rqt_image_view

Select the topic : `/DUCKIEBOT_NAME/lane_filter_node/belief_img`.

### Rviz

Rviz (ROS visualization) is a 3D visualizer for displaying sensor data and state information from ROS. More on information can be found here: http://wiki.ros.org/rviz

For this exercise rviz will be helpful to display sensor messages from the Duckiebot. In order to run rviz run:By selecting the appropriate topic we can output desired information. In particular xxxx and xxx will be useful in this exercise.

<figure>
<img style="width:30em" src="images/rosviz_screenshot.png"/>
</figure>

To start rviz run the following command

    laptop $ dts start_gui_tool ![DUCKIEBOT_NAME]

and then

    laptop-container $ rviz
  
**Rosparameter manipulation:**

TODO: Tomasz : say how to run everything with the same gui_tools container !

As a reminder: for rosparameter introspection and manipulation we recommend the following steps:

    laptop $ dts start_gui_tools ![DUCKIEBOT_NAME]

1) Listing the parameters:

    container $ rosparam list

2) Getting the parameters:

    container $ rosparam get parameter_name

3) Setting the parameters:

    container $ rosparam set parameter_name value

