# Exercises - lane pose estimation {#exercise-localization status=ready}

Excerpt: Play with the parameters of the localization pipeline.

The goal of this exercises is to play with existing parameters to understand the different trade-offs mentioned in [](#cra-loc-lm).

<div class='requirements' markdown='1'>
  Requires: [Camera calibration](+opmanual_duckiebot#camera-calib)

  Requires: [Docker basics](+duckietown-robotics-development#docker-basics)

  Requires: [ROS basics](+duckietown-robotics-development#sw-advanced)

  Requires: [Knowledge of the software architecture on a Duckiebot](+duckietown-robotics-development#duckietown-code-structure)

  Results: Understand the trade-offs when dealing with image processing parameters

  Results: Insights into the image pipeline of a Duckiebot.
</div>

## Task 1: Line detector exercise

As previously introduced, the `line_detector_node` detects white, yellow and red segments. The more segments we get, the more accurate we expect the lane filter to be, but also the more resources we need for computation of the pose estimate (memory as well as CPU usage). This is a trade-off between accuracy and computational efficiency. The goal of this exercise is to analyze this trade-off by determining the relationship between the number of segments processed and the quality and frequency of pose estimates that are being computed.

For this task the parameter `/![DUCKIEBOT_NAME]/line_detector_node/segment_max_threshold` can be dynamically adjusted.

#### Choosing the best number of segments (frequency) {#exercise:lineDetector}

Put the Duckiebot in the city and let it drive one whole loop with the exercise-provided lane following. For every whole loop use a different parameters of `/![DUCKIEBOT_NAME]/line_detector_node/segment_max_threshold` and record a ROS bag of `lane_pose` for each value of `segment_max_threshold`. You should know how to do that from [](+duckietown-robotics-development#ros-logs).

Write a custom Python script to analyze the publishing_frequency of the topic `/![DUCKIEBOT_NAME]/lane_filter_node/lane_pose` for each bag. Plot the relationship between `segment_max_threshold` on one axis and the mean and standard deviation of the lane_pose publishing frequency on the other axis. Provide at least 4 points on the plot. Include a point with a very high `segment_max_threshold` to virtually allow all segments to be computed.

<end/>

Frequency isn't the only relevant metric. Using one segment per color will give fast computation but very noisy and unstable estimation. Using the `rviz` tool that you launched before, you can analyze the stability of the lane_pose.

#### Choosing the best number of segments (stability) {#exercise:lineDetector2}

Create a graph, ploting on the y-axis $(d, \phi)$ against time on the x-axis for each of the loops from the previously recorded ROS bags.

<end/>

## Task 2: Lane pose exercise

As outlined in the introduction section, `lane_filter_node` estimates the Duckiebot's desired pose by means of recursive Bayes estimation. The sizes of the belief/likelihood matrices are adjustable parameters. We are interested in analyzing the effect of various matrix sizes on the precision/standard deviation of the lane pose estimation.

For this task the parameter `/![DUCKIEBOT_NAME]/lane_filter_node/matrix_mesh_size` can be dynamically adjusted.

#### Choosing the best matrix size {#exercise:laneFilter}

While running the exercise-provided lane following, play with `matrix_mesh_size`, and record different ROS bags for the topic lane_pose (one for each value of `matrix_mesh_size`).

Write a custom Python script to analyze the frequency of the topic `/![DUCKIEBOT_NAME]/lane_filter_node/lane_pose` for each bag (should be the same as last exercise). Plot the relationship between `matrix_mesh_size` on one axis and the the mean and standard deviation of the frequency of the `lane_pose` topic on the other axis. Provide at least 4 points on the plot.

Warning: sometimes, when dynamically changing the parameters, errors might occur since the matrix size might be changing during computation of the segments. In the occurrence of such a problem, you can restart the node or set the previous value of the mesh and then retry.

<end/>

## Task 3: English driver

One of our brave Duckiebots wanted to make a visit to a fellow Duckiebot at the [London Science Museum](https://www.sciencemuseum.org.uk/about-us/press-office/science-museum-explores-future-driven-autonomous-vehicles) in Great Britain (yup, must be *really* brave to go right before Brexit :X). However, it needs to adhere to the local driving rules. Therefore you will have to help it learn to drive on the left side of the road.

#### Driving the English style {#exercise:englishDriver}

The task is to make the Duckiebot drive on the left side of the road. The parameter `/![DUCKIEBOT_NAME]/lane_filter_node/lane_offset` and the provided snippet  [provided code snippet](#histogramfilter) is sufficient to complete this task. Coding is not necessary for this exercise.

<end/>
