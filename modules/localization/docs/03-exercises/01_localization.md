# Exercises - lane pose estimation {#exercise-localization status=ready}

Excerpt: Play with the parameters of the localization pipeline.

The goal of this exercise is to play with existing parameters to understand the different trade-offs mentioned in [](#cra-loc-lm).

<div class='requirements' markdown='1'>
  Requires: [Camera calibration](+opmanual_duckiebot#camera-calib)

  Requires: [Docker basics](+duckietown-robotics-development#docker-basics)

  Requires: [ROS basics](+duckietown-robotics-development#sw-advanced)

  Requires: [Knowledge of the software architecture on a Duckiebot](+duckietown-robotics-development#duckietown-code-structure)

  Results: Understand the trade-offs when dealing with image processing parameters

  Results: Insights into the image pipeline of a duckiebot.
</div>

## Task 1: Line detector exercise

As previously introduced, the `line_detector_node` detects white, yellow and red segments. The more segments we get, the more accurate we expect the lane filter to be, but also the more resources we need for computation of the pose estimate (memory as well as cpu). This is a trade-off of accuracy versus computation efficiency. The goal of this exercise is to analyze this trade-off by determining the relationship between the number of segments processed and the quality and frequency of pose estimates that are being computed.

For this task the parameter `/DUCKIEBOT_NAME/line_detector_node/segment_max_threshold` can be dynamically adjusted.

#### Choosing the maximum number of segments {#exercise:lineDetector}

While running the exercise provided lane following, play with `/DUCKIEBOT_NAME/line_detector_node/segment_max_threshold`, and record different rosbags (One for each value of `segment_max_threshold`).

Write a custom python script to analyse the frequency of the topic `/DUCKIEBOT_NAME/lane_filter_node/lane_pose` for each bag. Plot the relationship between `segment_max_threshold` on one axis and the mean and standard deviation of the lane_pose frequency on the other axis (provide at least 4 points on the plot). Include a point with a very high `segment_max_threshold`, to virtually allow all segments to be computed.

But frequency isn't the only relevant metric. Using one segment per color will give fast computation but very noisy and unstable estimation. Using the `rviz` tools that you launched before, estimate the stability of the estimation and find out the minimal number for `segment_max_threshold` that keeps a stable estimation.

<end/>

## Task 2: Lane pose exercise

As outlined in the introduction section, the lane_filter_node estimates the Duckiebot desired pose by means of recursive Bayes estimation. As a lever we can change the size of the belief/likelihood matrices. Therefore we are interested in analyzing the effect of various matrix sizes on the precision/standard deviation of the lane pose. For this task the following parameter will be dynamically adjusted:

* `/DUCKIEBOT_NAME/lane_filter_node/matrix_mesh_size`

#### Choosing the best matrix size {#exercise:laneFilter}

While running the exercise provided lane following, play with `matrix_mesh_size`, and record different rosbags (One for each value of `matrix_mesh_size`).

Write a custom python script to analyse the frequency of the topic `/DUCKIEBOT_NAME/lane_filter_node/lane_pose` for each bag (should be the same as last exercise). Plot the relationship between `matrix_mesh_size` on one axis and the the mean and standard deviation of the frequency of the `lane_pose` topic on the other axis (provide at least 4 points on the plot).

Warning: sometimes, when changing the parameters dynamically errors might occur, since the matrix size might be changing during computation of the segments.

<end/>

## Task 3: English driver

One of our brave Duckiebots wanted to make a visit to Great Britain before Brexit happens. However, it needs to adhere to the local driving rules. Therefore it need to learn to drive on the left side of the road.

#### Driving the english style {#exercise:englishDriver}

The task is to make the Duckiebot drive on the left side of the road. An appropriate parameter in the [provided code snippet](#histogramfilter) is sufficient to finish this task.

<end/>