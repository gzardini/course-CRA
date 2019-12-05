# Exercises {#cra-plan-exer status=ready}

Excerpt:
Apply your competences in software development to a planning task on a Duckiebot.
In this exercise you will learn how to implement different control algorithms on a Duckiebot and gain intuition on a range of details that need to be addressed when controlling a real system.

 <div class='requirements' markdown='1'>
 Requires: [Terminal Basics](+duckietown-robotics-development#terminal-basics)
 Requires: [Docker Basics](+duckietown-robotics-development#docker-basics)
 Requires: Planning using Balkcom curves basics (TODO: add reference to the paper)
 Results: Ability to use the Balkcom curves to plan and act on a real robot. </div> <minitoc/>

 ## Overview
 In the following exercise you will be asked to implement a planning and control algorithm to steer your Duckiebot to specific locations.
 Particularly, you will use the concepts you learned so far to detect an AprilTag and the notion of Balkcom curves (TODO: add reference) to steer your Duckiebot to a location in which the AprilTag is centered in the field of view (FOV).

 ## Background
 Consider the differential drive model of the Duckiebot

 $\dot{x}=\frac{r}{2}(v_l+v_r)\cos(\theta)$
 $\dot{y}=\frac{r}{2}(v_l+v_r)\sin(\theta)$
 $\dot{\theta}=\frac{r}{L}(v_r-v_l)$

 TODO: add figure here for the model

Furthermore, consider the distance traveled by the center of motion of the Duckiebot while driving on a path $\tile{\vec{q}}$ from $\vec{q}_I$ to $\vec{q}_G$:

$L(\tilde{\vec{q}},\tilde{\vec{q}}) = \int_{0}^{t_\text{F}} \sqrt{\dot{x}(t)^2+\dot{y}(t)^2} \text{d}t$.

As shown in LaValle (TODO: add reference), the shortest path for this case is equivalent to the path that takes minimum time. However, minimizing the distance covered by the center of motion
of the Duckiebot does not take into account the cost of rotations. In fact, if the wheels have opposite angular velocities, the center of motion of the spinning Duckiebot does not move. But should we capture this?
Well, your spinning Duckiebot will spend energy and time, which you want to minimize.

Balkcom and Mason (TODO: add reference) considered a slightly different problem, bounding the speed of the differential drive and minimizing the total time it takes to reach the goal location.
The new action set of the robot is $U=[-1,1]\times [-1,1]$, where 1 represents the velocity bound. The new criterion, which considers $\theta$ is:

$L(\tilde{\vec{q}},\tilde{\vec{q}}) = \int_{0}^{t_\text{F}} \sqrt{\dot{x}(t)^2+\dot{y}(t)^2}+\vert \dot{\theta}(t)\vert \text{d}t$

In the paper, it was shown that one only needs the four motion primitives reported in Table (TODO: add reference to table) to write the robot optimal paths.

## Exercise 1
Since in the remaining of this exericse you will rely on Balkcom and Mason's work, go over their paper and gain an intuition of the algorithm principles.

## Exercise 2

