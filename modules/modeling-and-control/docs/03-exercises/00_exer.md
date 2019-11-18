# Exercise: Control {#cra-mac-exer status=ready}

Excerpt: Apply your competences in software development to a controls task on a Duckiebot.

In this exercise you will learn how to implement different control algorithms on a Duckiebot and gain intuition on a range of details that need to be addressed when controlling a real system.

<div class='requirements' markdown='1'>
  Requires: [Control Theory Basics](#cra-mac-lm)

  Requires: [Terminal Basics](+duckietown-robotics-development#terminal-basics)

  Requires: [Docker Basics](+duckietown-robotics-development#docker-basics)

  Results: Ability to implement a controller on a real robot.


</div>

<minitoc/>

## Overview
In the following exercise you will be asked to implement two different kinds of control algorithms to control
the Duckiebot. In a first step, you will write a PI controller and gain some intuition on
different parameters that are important for the controller design such as discretization method,
sampling time and delay. You will also learn what an anti-windup scheme is and how it can be useful in
a real robot.\

In a second step, you will implement a Linear Quadratic Regulator, or LQR for short. You then augment it by an integral part, making it a LQRI. This approach represents a more high-level approach to the
control problem. You will see how it is less intuitive but at the same time it brings certain advantages
with it as you have learned in [](#cra-mac-lm).

## PI control {#cra-mac-pi-control}
### Modeling
As you have learned in class, using Fig. [](#fig:duckiebot_topview) you can derive a state-space model of a Duckiebot. Specifically, considering the state $\vec{x} = \begin{bmatrix} d& \varphi \end{bmatrix}^\intercal$, one can write the nonlinear model:

$\dot{\vec{x}} = \begin{bmatrix} v \cdot \sin(\varphi) \\ \omega \end{bmatrix}$.

The input of the system is $u = \omega$ and the output is $y$.

<figure>
    <img figure-id="fig:duckiebot_topview" figure-caption="Lane Pose of the Duckiebot." style="width: 75%; height: auto;" src="duckiebot_topview.pdf"/>
</figure>


After linearization around the operation point $\vec{x} = \begin{bmatrix}0 & 0\end{bmatrix}^\intercal$ (if you do not remember model linearization, have a look at (TODO:put reference), the model looks as follows:

$\dot{\vec{x}} = A\vec{x} + Bu = \begin{bmatrix} 0 & v \\ 0 & 0 \end{bmatrix}\vec{x} + \begin{bmatrix} 0 \\ 1 \end{bmatrix}u$,

$\vec{y} = C\vec{x} =\begin{bmatrix}6 & 1\end{bmatrix}\vec{x}$.

The transfer function of the system is $P(s) = \frac{s + 6v}{s^2}$.

Let $e = r - y$. A PI-controller for this system looks like

$u(t) = k(e(t) + \frac{1}{T_I}\int_{0}^{t}e(\tau) d\tau) := k_P e(t) + k_I\int_{0}^{t}e(\tau) d\tau$

in time domain and

$C(s) = k(1 + \frac{1}{T_Is}) := k_P + k_I\frac{1}{s}$
in frequency domain, where
$U(s) = C(s)E(s) = C(s)(R(s) - Y(s))$.

#### Find the gains {#exercise:find-gains}
Using the above model of the Duckiebot and the structure for the PI controller, find the parameters for the proportional and integral gain of your PI-controller such that it is stable. You can follow the steps below to do this:\
- For the Duckiebot you are assuming a constant linear velocity $v = 0.22 $ $ m/s$. Given this velocity and by using a tool of your
choice (for example the [Duckiebot bodeplot tool](https://julien.li/submit/bodeplot/)), find a proportional gain $k_P$ such that $L(s) = P(s)C(s)$ has a crossover frequency of
approximately $4.2$ $ rad/s$.
- Next, find an integral gain $k_I$ such that $L(s)$ has a gain margin of approximately $-25.6dB \approx 0.053$
(this refers to a gain of the controller which is about 19 times higher than the critical minimal gain that is needed for stability).

<end/>

### Discretization
Now that you have found your continuous time controller, you need to discretize it in order to implement it on your Duckiebot. There is several ways of doing this as you have discovered in
[](#cra-mac-lm). In the following exercise, you are asked to implement your PI controller using different discretization methods.

#### Discretization of a PI controller {#exercise:discretize-pi}

There is a template for this exercise. It is a docker image of the `dt-core` with an additional folder called `CRA3`. This folder contains two controller templates, `controller-1.py` and `controller-2.py`. The first one will be used for the exercises with the PI controller and the other one for the implementation of the LQR(I) which you will do about later on.

Start by pulling the image and running the container with the following command:

    laptop $ docker -H ![DUCKIEBOT_NAME].local run --name dt-core-CRA3 -v /data:/data --privileged --network=host -it hosnerm/dt-core:CRA3-template /bin/bash

Note: in case you have to stop the container at any point (in case you take a break or the Duckiebot decides to crash and therefore make you take a break), start the container again (for example by using the portainer interface (`http://hostname.local:9000/#/containers`)) and jump into it using the following command:

    laptop $ docker -H ![DUCKIEBOT_NAME].local attach dt-core-CRA3

The text editor vim is already installed inside the container such that you can change and adjust files within the container quickly. If you are not familiar with vim, you can either read through this [short beginners guide to vim](https://www.linux.com/tutorials/vim-101-beginners-guide-vim/) or install another text editor of your choice.\
Now use vim or your preferred text editor
to open the file `controller-1.py` which can be found in the folder `CRA3`.\
The script contains a template for your PI controller including input and output variables of the controller and several variables which will be used within this exercise. As inputs you
will get the lane pose estimate of the Duckiebot and you will have to compute the output which is in the form of the yaw rate omega.\   
Familiarize yourself with the template and fill in the previously found values for the proportional and integral gain $k_P$ and $k_I$.  


Now you are ready to implement a PI controler using different discretization methods:


##### Assume constant sampling time
In a first attempt, you can use an approximation for your sampling time. The Duckiebot typically updates its lane pose estimate, i.e. where the Duckiebot thinks it is placed within the lane, at around 12 Hz. If you assume that the 12Hz are a constant value,
you can discretize your PI controller using a constant sampling time. Implement your PI controller under the assumption of a constant sampling in the file `controller-1.py`.

You can run the controller you just wrote by executing the following command:

    laptop-container $ roslaunch duckietown_demos lane_following_exercise.launch veh:=![DUCKIEBOT_NAME] exercise_name:=1

Observe the behaviour of the Duckiebot. Does it perform youll? Think about why this is the case.


##### Euler Forward
Now you will use the actual time that has passed in between two lane pose estimates of the Duckiebot and discretize using the Euler forward method. The time between two
lane pose estimates is already available to you in the template and is called `dt_last`. Use your knowledge from (#cra-mac-lm) to adjust the discretization method of your controller.\
After you adjusted your file, use the same command as above to test your controller.
Observe the behaviour again, what differences do you notice? Why is that?


%%%%%SHALL you EVEN DO TUSTIN --> VERY SMALL DIFFERENCE????%%%%%
##### Tustin
Finally you want to use Tustin's method for discretization. Implement it using your theoretical knowledge from [](#cra-mac-lm) and compare its performance to the controller with the Euler forward discretization.

<end/>


### Sampling Time
In the last exercise you implemented a discrete time controller and saw how slight variations in the sampling time can have an impact on the performance of the Duckiebot. you now want to further explore how the sampling time impacts the performance of the controller by decreasing the sampling time and observing the outcome.   

#### Impact of a smaller sampling rate {#exercise:sampling}

The model of a Duckiebot only works on a specific range of states $[d,\varphi]$. If those values grow too large, the camera loses sight of the lines and the estimation
of the output $y$ is not possible any longer, causing the system to destabilize. By increasing $k_s$ in `controller-1.py`, check how much you can reduce the sampling rate
until the system destabilizes. Notice that since your controller is discrete, you can only increase the sampling time $T$ in discrete steps $k_s$ where $T_{new} = k_sT$.
This functionality is already implemented in the lane controller node for you. To reduce the sampling rate, the Duckiebot only handles every $k_s$-th measurement ($\#measurement \bmod k_s \equiv 0$), and drops all the other measurements.

Adjust the parameter $k_s$ such that the Duckiebot becomes unstable. What is the approximate sampling time when the Duckiebot becomes unstable?  

Again run your code with:

    laptop-container $ roslaunch duckietown_demos lane_following_exercise.launch veh:=![DUCKIEBOT_NAME] exercise_name:=1

After you have found a value for $k_s$ that destabilizes your Duckiebot, try to improve the robustness of your controller against the smaller sampling rate and make it
stable again. There is different ways to do this. Explain how you did it and why.

<end/>  


### Delay

For now, the delay which is present in the Duckiebot (the plant) has not been explicitly addressed. From the moment an image is recorded until the control action is applied, it takes roughly 85ms. This implies that you will never be able to act upon the exact state that your Duckiebot is in. In the following exercise you will examine how the Duckiebot behaves if this delay between image acquisition and control command changes.

#### Increasing the delay {#exercise:delay}


##### Stability - Theoretical

As you've already seen, the time delay of 85ms does not destabilize your system. By using your calculations from [](#cra-mac-pi-control), you are indeed able to identify a maximal time delay such that your system is still stable in theory. This can be done by having a look at the transfer function of a time-delayed system:

$P_d(s) = e^{-sT_d} P(s)$

An increase of $T_d$ leads to a shift of the phase in negative direction. Therefore, $T_d$ must not be larger than the phase margin of $L(s)$ (which was roughly $70^{\circ}$) to not destabilize the system. Calculate the maximal $T_d$ such that the system is still stable.


##### Stability - Practical

Before you can reach this theoretical limit, the Duckiebot will most likely leave the road and the pose estimation will fail since the lines are not in the field of view of the camera anymore. In  `controller-1.py`, increase the time delay multiplicator $k_d$ of the system until the Duckiebot cannot stay in the lane anymore. Notice that the time delay is implemented in discrete steps of $k_d * T$ where T is the sampling time.

Again run your code with:

    laptop-container $ roslaunch duckietown_demos lane_following_exercise.launch veh:=![DUCKIEBOT_NAME] exercise_name:=1


How big is the difference between the theoretical and the practical limit?

<end/>  

### Increase performance of your PI controller

An integral part in the controller comes with a drawback in a real system: Due to the fact that the motors on a Duckiebot can only run up to a specific speed, you are not able to perform unbounded high inputs demanded by the controller. \
If the Duckiebot cannot execute the demanded commands of the controller, the difference between the demanded input and the executed input will remain and therefore be added on top of the  demanded input which is already too high to be executed.
This leads us to a situation in which the integral term can become very large. If you now reach your desired equilibrium point, the integrator will still have a large value, causing the Duckiebot to overshoot. \
But behold, there is a solution to this problem! It is called anti-windup and will be examined in your next exercise.   

#### Effect of an Anti-Windup {#exercise:anti-windup}
In Fig. [](#fig:anti_reset_windup_ex), you can see a diagram of an anti-windup logic for a PI-controller. $k_t$ determines how fast the integral is reset and is normally chosen in the order of $k_I$.


<figure>
    <img figure-id="fig:anti_reset_windup_ex" figure-caption="A PI-controller with an anti-windup logic implemented, Feedback Systems from Aström and Murray, page 308." style="width: 75%; height: auto;" src="PI-with-anti-windup.png"/>
</figure>


Typically the actuator saturation (i.e. when it reaches its physical limit) could be measured. But as there is no feedback on the wheels commands that are being executed, you will make an assumption for this exercise. You will simulate a saturation of the motors at a value of $u_{sat} = 2rad/s$. \
Below you find a simple helper function that you can use to add an anti-windup to your existing PI controller. It takes an unbounded input and limits it to the mentioned saturation input value $u_{sat}$.Use it to extend your existing PI controller with an anti-windup scheme. \
Furthermore you are given the parameter $k_t$ in the file `controller-1.py`. It shall be used as a gain on the difference between the input $u$ and the saturation input value $u_{sat}$ which is fed back to the integrator part of the controller as it is shown in Fig. [](#fig:anti_reset_windup_ex).

As a first step, test the performance of the Duckiebot with the anti-windup term turned off: $k_t = 0$. You will see that the performance is poor after curves. If you increase the integral gain $k_I$, you are even able to destabilize the system! Edit the file with

In order to avoid destabilization and improve the performance of the system, set $k_t$ to roughly the same value as $k_I$. Note the difference!

You can run your code as before with:

    laptop-container $ roslaunch duckietown_demos lane_following_exercise.launch veh:=![DUCKIEBOT_NAME] exercise_name:=1

*Optional:* With different values of $k_P$ and $k_I$, one could improve the behaviour even more.

##### Template for saturation function:

```python
    def sat(self, u):
            if u > self.u_sat:
                return self.u_sat
            if u < -self.u_sat:
                return -self.u_sat
            return u
```

As you may have found out, for very aggressive controllers with an integral part and systems which saturate for relatively low inputs, the use of an anti-windup logic is necessary. In the case of a Duckiebot however, an anti-windup logic is only needed if you want to introduce a limitation to the angular velocity $\omega$ - for example to simulate a real car (minimal turning radius).
<end/>  


By now you should have a nicely working controller to keep your Duckiebot in the lane which is robust against a range of perturbations which arise from the real world.
But is the solution that you found optimal and does it give us a guarantee on its stability? The answer to both of these questions is no. Also you saw that the fact that your model is not exactly matching the reality can lead to a worse performance. \
Therefore it would be useful to have a control algorithm which does not depend heavily on the given model and gives us guarantees on its stability.

In the last two exercise parts, you will look at a different controller which will help us solve the above mentioned problems; namely a Linear-Quadratic-Regulator (LQR).


## Linear Quadratic Regulator
A linear quadratic regulator (LQR) is a a state feedback control approach which works by minimizing a cost function. This approach is especially suitable if we want to have some high-level tuning parameters where the cost can be traded off against the performance
of the controller. Here we typically refer to "cost" as the needed input and "performance" as the reference tracking and robustness characteristics of the controller. In addition, no exact model of
the object or process to be controlled is needed. LQR control works well even when no precise model is available as it is often the case in practical applications. This makes it a suitable controller
for real world applications.

#### Discretize the model {#exercise:discretize-model}

As in the part above, you will start with the model of the Duckiebot. This time though you are going to discretize the system before creating a controller for it which will make updating the weights easier once you test your controllers on the real system. The continuous time model of a Duckiebot is:

$\dot{\vec{x}}=A\vec{x}+Bu=\begin{bmatrix}0 & v\\ 0 & 0\end{bmatrix}\vec{x}+\begin{bmatrix}0\\1\end{bmatrix}u$

$\vec{y}=C\vec{x}=\begin{bmatrix}1 & 0\\ 0 & 1 \end{bmatrix}\vec{x}$

With state vector $\vec{x}=\begin{bmatrix}d, & \varphi\end{bmatrix}^T$ and input $u=\omega$. Notice, that the matrix C is an identity matrix, which means that the states are directly mapped to the outputs.


Discretize the system in terms of velocity $v$ and the sampling time $T_s$ using exact discretisation from [](#cra-mac-lm) and test your discretization using the provided [Matlab-files](https://github.com/duckietown/docs-exercises/tree/master19/book/exercises/200_control_systems/additional_material)). What do you observe?
Add the found matrices in the template `controller-2.py`.

<end/>

#### Implement a LQR {#exercise:LQR}

To achieve a better lane following behaviour, a LQR can be implemented. The structure of a state feedback controller such as the LQR looks as in Fig. [](#fig:lqr-feedback):

<figure>
    <img figure-id="fig:lqr-feedback" figure-caption="Block diagram of a state feedback control." style="width: 70%; height: auto;" src="lqr_feedback.png"/>
</figure>

Because of limited computation resources, a stead-state (or _infinite horizon_) version of the LQR will be implemented. Because you are considering the discrete time model of the Duckiebot, the Discrete-time Algebraic Riccati Equation (DARE) has to be solved:

$\Phi = A^T\Phi A - (A^T \Phi B)(R+B^T \Phi B)^{-1}(B^T \Phi A)+Q$

To solve this equation use the Python control library (see [python control library documentation](https://python-control.readthedocs.io/en/0.8.2/)).

Note: make sure to normalize your $R$ and $Q$ Matrices. Choose the corresponding weights and tune them until you achieve a satisfying behaviour on the track.

Once you are ready, run your LQR with:

    laptop-container $ roslaunch duckietown_demos lane_following_exercise.launch veh:=![DUCKIEBOT_NAME] exercise_name:=2


Explain what happens when you assign the entries in your weighting matrices different values. Can you describe it intuitively?

<end/>

#### Implement a LQRI {#exercise:LQRI}

The above controller should yield satisfactory results already. But you can even do better! As the LQR does not have any integrator action, a steady state error will persist. To eliminate this
error, you will expand your continuous time state space system by an additional state which describes the integral of the distance $d$.
The expanded system then looks as follows:

$A=\begin{bmatrix}0 & v & 0 \\ 0 & 0 & 0 \\ -1 & 0 & 0 \end{bmatrix}$
$B=\begin{bmatrix} 0 \\ 1 \\ 0 \end{bmatrix}$
$C=\begin{bmatrix} 1 & 0 & 0 \\ 0 & 1 & 0 \\ 0 & 0 & 1 \end{bmatrix}$
$D=\begin{bmatrix} 0 \\ 0 \\ 0 \end{bmatrix}$

**Bonus question** (*optional*): Why don't you also account for the integral state of the angle?

Now discretize the above system as before and extend the state space matrices and the weighting matrices in your existing code in `controller-2.py`.

Run it again with

    laptop-container $ roslaunch duckietown_demos lane_following_exercise.launch veh:=![DUCKIEBOT_NAME] exercise_name:=2

How does your controller perform now?

<end/>
