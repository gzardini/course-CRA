# Learning materials {#cra-loc-lm status=ready}

Excerpt: Understand the components of the localization pipeline: from image to lane pose estimation.

The goal of this exercise is to familiarize us with the pipeline that extract lane localization from the image stream. This is the base of the lane following demo.

## Overview of the pipeline

Determining its own position in the lane is essential for any Duckiebot to survive in the city. In the following section we will go step by step through the various steps of the image pipeline: from image to position.

Fig. 4.1 shows the 2 important parts of the localization, the **line detector** and the **lane filter**, and where they stand in the whole image to control pipeline. The control aspect will be the focus of the next set of exercises. We will focus here only on the two above-mentioned parts.

<figure>
  <img style="width:30em" src="images/image_pipeline_overview.png"/>
</figure>

## Line detector node

### Role of the node

The line detector node is responsible for detecting lines in the field of view of the Duckiebot, of three different colours: red, white and yellow.

### Ros interfacing of the node

**The line detector node subscribes to:**

* The corrected image stream

**The line detector node publishes:**

* Segment list (type: SegmentList.msg) is an array which saves all segments (type: Segment.msg) found in the image. A segment consists of colour (red, yellow, white) and 2D vector (startpoint, endpoint).

## Lane filter node

### Role of the node

The lane filter node is responsible for estimating the position of the Duckiebot with respect to the center of the driving lane.  

### Ros interfacing of the node

**The lane filter node subscribes to:**

* The segment list from the line detector node

**The lane filter node publishes:**

Lane pose (type: duckietown_msgs/lane_pose) is struct with the following parameters which are currently in use:
  
  1. d (float32) the lateral offset, where d = 0 is the middle of the right lane
  2. phi (float32) the angle from the center of the lane to the orientation of the duckiebot

Note: When the duckiebot is perfectly aligned in the center of its lane, facing forward, this estimation should be (d = 0.0, phi = 0.0)

### Bayes filter

Bayes filters are a probabilistic tool for estimating the state of dynamic systems. In the case of our Duckiebot, given a stream of observation (in this case the camera image) we compute a measurement likelihood matrix with a histogram filter. This enable us to calculate an initial belief. Then, we project the belief of the previous time step to the current time step. We can do this in the following way:

belief (t+1) = belief(t) * measurement_likelihood(t)

From the belief, we then extract the pose d and angle phi with the highest probability.

### Histogram filter

Each 2d white and yellow segments are projected onto the Duckiebots reference frame. Then the horizontal distance from the white/yellow segment to the desired position (middle point between right white lane and yellow lane) is calculated. The same is done for the angle phi. For all the segments a histogram is calculated which can be then display under as an image stream.

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