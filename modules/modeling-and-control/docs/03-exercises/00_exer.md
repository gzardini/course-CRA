# Exercise: Control {#cra-mac-exer status=ready}

Excerpt: Apply your competences in software development to a controls task on a Duckiebot.

In this exercise you will learn how to implement different control algorithms on a Duckiebot and gain intuition on a range of details that need to be addressed when controlling a real system.

<div class='requirements' markdown='1'>

  Requires: [Terminal Basics](+duckietown-robotics-development#terminal-basics)

  Requires: [Docker Basics](+duckietown-robotics-development#docker-basics)

  Requires: Control Theory Basics

  Results: Ability to implement a controller on a real robot.

</div>

<minitoc/>

## Overview
In the following exercise you will be asked to implement two different kinds of control algorithms to control
the Duckiebot. In a first step, you will write a PI controller and gain some intuition on
different factors that are important for the controller design such as the discretization method, the sampling time, and the latency of the estimate. You will also learn what an anti-windup scheme is and how it can be useful on a real robot. <br />
In a second step, you will implement a Linear Quadratic Regulator, or LQR for short. You then augment it by an integral part, making it a LQRI. This is a more high-level approach to the control problem. You will see how it is less intuitive but at the same time it brings certain advantages as you will see in the exercise.

## PI control {#cra-mac-pi-control}

### Modeling
As you have learned, using [](#fig:duckiebot_topview) one can derive a continuous-time nonlinear model for the Duckiebot. Considering the state $\vec{x}(t)=\begin{bmatrix} d& \varphi \end{bmatrix}^\intercal$, one can write
$\dot{\vec{x}} = \begin{bmatrix} v\cdot \sin(\varphi) \\ \omega \end{bmatrix}$. Where $v$ is the linear velocity and $\omega$ the yaw rate of the Duckiebot.

<div figure-id="fig:duckiebot_topview">
  <figcaption> "Top view of the Duckiebot on a road with its two states." </figcaption>
  <img style="width: 75%; height: auto;" src="duckiebot_topview.pdf"/>
</div>

After linearization around the operation point $\vec{x}_\text{e}=\begin{bmatrix} 0&0 \end{bmatrix}^\intercal$ (if you do not remember linearization, have a look at chapter 5.4 in [](#bib:Astr)), one has
$\dot{x}(t)= \begin{bmatrix} 0&v \\ 0&0 \end{bmatrix}x(t) + \begin{bmatrix} 0 \\ 1 \end{bmatrix}u(t)$ where the input $u(t)$ is the desired yaw rate of the Duckiebot.
Furthermore, you are provided the output model

$y(t)=\begin{bmatrix} 6&1 \end{bmatrix}\vec{x}(t).$

Using the linearized version of the model, you can compute the transfer function of the system:

$P(s)=\frac{s+6v}{s^2}$.

If you do not remember how, have a look at chapter 8 in [](#bib:Astr).

Consider now the error to be $e(t)=r(t)-y(t)$. Using a PI-controller (if you do not remember what a PI-controller is, have a look at chapter 10 in [](#bib:Astr)), one can write

$u(t) = k_\text{P}\left(e(t) + \frac{1}{T_I}\int_{0}^{t}e(\tau) d\tau\right) = k_\text{P} e(t) + k_\text{I}\int_{0}^{t}e(\tau) d\tau$

with k_\text{P} being the proportional gain and k_\text{I} being the integral gain. <br />
In frequency domain, this corresponds to

$U(s)=C(s)E(s)$,    

with

$C(s)= k_\text{P}+\frac{k_\text{I}}{s}$


#### Find the gains {#exercise:find-gains}
Using the above defined model of the Duckiebot and the structure for a PI controller, find the parameters for the proportional and integral gain of your PI controller such that the closed-loop system is stable. You can follow the steps below to do this: <br />

- For the Duckiebot you are assuming a constant linear velocity $v = 0.22 \text{m/s}$. Given this velocity and using a tool of your choice (for example the [Duckiebot bodeplot tool](https://julien.li/submit/bodeplot/)), find a proportional gain $k_\text{P}$ such that $L(s) = P(s)C(s)$ has a crossover frequency of
approximately $4.2 \text{rad/s}$.

- Next, find an integral gain $k_\text{I}$ such that $L(s)$ has a gain margin of approximately $-25.6dB \approx 0.053$.
(this refers to a gain of the controller which is about 19 times higher than the critical minimal gain that is needed for stability).

The aforementioned numbers are needed in order to guarantee stability. You are free to play around with them and see for yourself how this impacts the behaviour of your Duckiebot.

<end/>

### Discretization
Now that you have found a continuous time controller, you need to discretize it in order to implement it on your Duckiebot. There are several ways of doing this.
In the following exercise, you are asked to implement the designed PI controller in reality, using different discretization techniques.

#### Discretization of a PI controller {#exercise:discretize-pi}
There is a template for this and the following exercises of this chapter. It is a Docker image of the `dt-core` with an additional folder called `CRA3`. This folder contains two controller templates, `controller-1.py` and `controller-2.py`. The first one will be used for the exercises with the PI controller and the other one for the implementation of the LQR(I) which you will do from exercise [](#exercise:LQR) onwards.
Start by pulling the image and running the container with the following command:

    laptop $ docker -H ![DUCKIEBOT_NAME].local run --name dt-core-CRA3 -v /data:/data --privileged --network=host -it duckietown/dt-core:CRA3-template /bin/bash

Note: in case you have to stop the container at any point (in case you take a break or the Duckiebot decides to crash and therefore makes you take a break), start the container again (for example by using the portainer interface (`http://hostname.local:9000/#/containers`)) and jump into it using the following command:

    laptop $ docker -H ![DUCKIEBOT_NAME].local attach dt-core-CRA3

The text editor _vim_ is already installed inside the container such that you can change and adjust files within the container without having to rebuild the image every time you want to change something. If you are not familiar with vim, you can either read through this [short beginners guide to vim](https://www.linux.com/tutorials/vim-101-beginners-guide-vim/) or install another text editor of your choice. <br />
Now use vim or your preferred text editor
to open the file `controller-1.py` which can be found in the folder `CRA3`.

The file contains a template for your PI controller including input and output variables of the controller and several variables which will be used within this exercise. As inputs you
will get the lane pose estimate of the Duckiebot and you will have to compute the output which is in the form of the yaw rate $\omega$. <br />

Familiarize yourself with the template and fill in the previously found values for the proportional and integral gain $k_\text{P}$ and $k_\text{I}$.  
Now you are ready to implement a PI controller using different discretization methods: <br />
<br />

##### Assume constant sampling time
In a first attempt, you can use an approximation for your sampling time. The Duckiebot typically updates its lane pose estimate, i.e. where the Duckiebot thinks it is placed within the lane, at around 12 Hz. If you assume this sampling rate to be constant, you can discretize the PI controller you designed. Implement your PI controller under the assumption of a constant sampling in the file `controller-1.py`. When discretizing the system, choose Euler forward as the discretization technique (if you do not remember how, have a look at chapter 2.3 in [](#bib:Zardini)).
You can run the controller you just designed by executing the following command:

    laptop-container $ roslaunch duckietown_demos lane_following_exercise.launch veh:=![DUCKIEBOT_NAME] exercise_name:=1

Observe the behaviour of the Duckiebot. Does it perform well? What do you observe? Think about why this is the case.

*Optional*: repeat the above task using the Tustin discretization method. Do you observe any difference? <br />
<br />

##### Assume a dynamic sampling time
Now you will use the actual time that has passed in between two lane pose estimates of the Duckiebot to discretize the system. The time between two lane pose estimates is already available to you in the template and is called `dt_last`. Adjust the discretization method (either Euler forward or Tustin) of your controller to account for the actual sampling time.
After you adjusted your file, use the same command as above to test your controller.
Observe the behaviour again, what differences do you notice? Why is that? <br />
<br />

##### Assume a large sampling time
In the last exercise you implemented a discrete time controller and saw how slight variations in the sampling time can have an impact on the performance of the Duckiebot. You now want to further explore how the sampling time impacts the performance of the controller by increasing it and observing the outcome. For the following, consider Euler forward as the discretization technique.
The model of a Duckiebot only works on a specific range of consequent states $[d_{i,i+1},\varphi_{i,i+1}]$. If these values grow too abruptly, the camera loses sight of the lines and the estimation
of the output $y$ is not anymore possible. By increasing $k_\text{s}$ in `controller-1.py`, check how much you can reduce the sampling rate before the system destabilizes. Notice that since your controller is discrete, you can only increase the sampling time $T$ in discrete steps $k_\text{s}$ where $T_\text{new} = k_\text{s}\cdot T$.
This functionality is already implemented in the lane controller node for you. To reduce the sampling rate, the Duckiebot only handles every $k_\text{s}$-th measurement ($\text{#measurements} \bmod k_\text{s} \equiv 0$), and drops all the other measurements.
Adjust the parameter $k_\text{s}$ such that the Duckiebot becomes unstable. What is the approximate sampling time when the Duckiebot becomes unstable?  
Again run your code with:

    laptop-container $ roslaunch duckietown_demos lane_following_exercise.launch veh:=![DUCKIEBOT_NAME] exercise_name:=1

After you have found a value for $k_\text{s}$ that destabilizes your Duckiebot, try to improve the robustness of your controller against the smaller sampling rate and make it
stable again. There are different ways to do this. Explain how you did it and why.

<end/>

### Latency of the estimate
Until now, the delay which is present in the Duckiebot (the plant) has not been explicitly addressed. From the moment an image is recorded until the lane pose estimate is available, it takes roughly 85ms. This implies that you will never be able to act upon the exact state that your Duckiebot is observed to be in. In the following exercise you will examine how the Duckiebot behaves if this delay between image acquisition and pose estimation changes.

#### Increasing the delay {#exercise:delay}
<br />

##### Stability - Theoretical
As you have already seen in the previous tasks, the time delay of 85ms does not destabilize your system. By using your calculations from [](#cra-mac-pi-control), you are indeed able to identify a maximal time delay such that your system is still stable in theory. This can be done by having a look at the transfer function of a time-delayed system:
$P_\text{d}(s) = e^{-sT_\text{d}} P(s)$ with $T_\text{d}$ being the time delay.
An increase of $T_\text{d}$ leads to a shift of the phase in negative direction. Therefore, $T_\text{d}$ must not be larger than the phase margin of $L(s)$ (which was roughly $70^{\circ}$ in our case) to prevent destabilizing the system. Calculate the maximal $T_\text{d}$ such that the system is still stable.
<br />

##### Stability - Practical
Before you can reach the theoretical limit you found in the previous task, the Duckiebot will most likely leave the road and the pose estimation will fail since the lines are not in the field of view of the camera anymore. In  `controller-1.py`, increase the time delay gain $k_\text{d}$ of the system until the Duckiebot cannot stay in the lane anymore. Notice that the time delay is implemented in discrete steps of $k_\text{d} * T$ where T is the sampling time.
Again run your code with:

    laptop-container $ roslaunch duckietown_demos lane_following_exercise.launch veh:=![DUCKIEBOT_NAME] exercise_name:=1

How big is the difference between the theoretical and the practical limit? <br />
*Optional*: Check if using another discretization technique substantially changes these numbers.

<end/>  

### Increase performance of your PI controller
The integral part in the controller comes with a drawback in a real system: Due to the fact that the motors on a Duckiebot can only run up to a specific speed, you are not able to perform unbounded high inputs demanded by the controller.
If the Duckiebot cannot execute the commands which the controller demands, the difference between the demanded input and the executed input will remain and therefore be added on top of the demanded input which is already too high to be executed.
This leads us to a situation in which the integral term can become very large. If you now reach your desired equilibrium point, the integrator will still have a large value, causing the Duckiebot to overshoot. <br />
But behold, there is a solution to this problem! It is called anti-windup filter and will be examined in the next exercise.   

#### Effect of an Anti-Windup Filter {#exercise:anti-windup}
In [](#fig:anti_reset_windup_ex), you can see a diagram of an anti-windup logic for a PI-controller. $k_\text{t}$ determines how fast the integral is reset and is usually chosen in the order of $k_\text{I}$.

<div figure-id="fig:anti_reset_windup_ex">
  <figcaption> A PI-controller with an anti-windup logic implemented, Feedback Systems from Aström and Murray, page 308.</figcaption>
  <img style="width: 75%; height: auto;" src="PI-with-anti-windup.png"/>
</div>

Typically, the actuator saturation (i.e. when it reaches its physical limit) can be measured. In our case, however, as there is no feedback on the wheels commands that are being executed, we will make an assumption. You will simulate a saturation of the motors at a value of $u_\text{sat} = 2\text{rad/s}$. <br />
Below you can find a simple helper function that you can use to add an anti-windup to your existing PI controller. It takes an unbounded input and limits it to the mentioned saturation input value $u_\text{sat}$. Use it to extend your existing PI controller with an anti-windup scheme. <br />
Furthermore you are given the parameter $k_\text{t}$ in the file `controller-1.py`. It shall be used as a gain on the difference between the input $u$ and the saturation input value $u_\text{sat}$ which is fed back to the integrator part of the controller as it is shown in [](#fig:anti_reset_windup_ex).
As a first step, test the performance of the Duckiebot with the anti-windup term turned off (i.e. $k_\text{t} = 0$). You will see that the performance is poor after curves. If you increase the integral gain $k_\text{I}$, you are even able to destabilize the system!
In order to avoid destabilization and improve the performance of the system, set $k_\text{t}$ to roughly the same value as $k_\text{I}$. Note the difference!
You can run your code as before with:

    laptop-container $ roslaunch duckietown_demos lane_following_exercise.launch veh:=![DUCKIEBOT_NAME] exercise_name:=1

*Optional:* With different values of $k_\text{P}$ and $k_\text{I}$, one could improve the behaviour even more.

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

By now you should have a nicely working controller to keep your Duckiebot in the lane, which is robust against a range of perturbations which arise from the real world.
But is the solution that you found optimal and does it give us a guarantee on its stability? The answer to both of these questions is no. Also, you saw that the fact that your model is not exactly matching the reality can lead to a worse performance. <br />
Therefore, it would be useful to have a control algorithm which does not depend heavily on the given model and gives us guarantees on its stability.
In the last two exercise parts, you will look at a different controller which will help us solve the above mentioned problems; namely a Linear-Quadratic-Regulator (LQR).

## Linear Quadratic Regulator (*Optional*)
A Linear Quadratic Regulator (LQR) is a a state feedback control approach which works by minimizing a cost function. This approach is especially suitable if we want to have some high-level tuning parameters where the cost can be traded off against the performance of the controller. Here, we typically refer to "cost" as the needed input $u(t)$ and "performance" as the reference tracking and robustness characteristics of the controller.
In addition, LQR control works well even when no precise model is available as it is often the case in practical applications. This makes it a suitable controller for real world applications.


#### Discretize the model {#exercise:discretize-model}
As in the part above, you will start with the model of the Duckiebot. This time though you are going to discretize the system before creating a controller for it which will make updating the weights easier once you test your controllers on the real system. The continuous time model of a Duckiebot is:
$\dot{\vec{x}}=A\vec{x}+Bu=\begin{bmatrix}0 & v\\ 0 & 0\end{bmatrix}\vec{x}+\begin{bmatrix}0\\1\end{bmatrix}u$
$\vec{y}=C\vec{x}=\begin{bmatrix}1 & 0\\ 0 & 1 \end{bmatrix}\vec{x}$
With state vector $\vec{x}=\begin{bmatrix}d, & \varphi\end{bmatrix}^T$ and input $u=\omega$. Notice, that the matrix $C$ is an identity matrix, which means that the states are directly mapped to the outputs.
Discretize the system in terms of velocity $v$ and the sampling time $T$ using exact discretisation (if you do not remember how, have a look at chapter 1.4 in [](#bib:SigSys)) and test your discretization using the provided [Matlab-files](https://github.com/duckietown/docs-exercises/tree/master19/book/exercises/200_control_systems/additional_material)). What do you observe?
Add the found matrices in the template `controller-2.py`.

<end/>

#### Implement a LQR {#exercise:LQR}
To achieve a better lane following behaviour, a LQR can be implemented. The structure of a state feedback controller such as the LQR looks as in [](#fig:lqr-feedback):

<div figure-id="fig:lqr-feedback">
 <figcaption> Block diagram of a state feedback control.</figcaption>
 <img style="width: 70%; height: auto;" src="lqr_feedback.png"/>
</div>

Because of limited computation resources, a steady-state (or _infinite horizon_) version of the LQR will be implemented. Because you are considering the discrete time model of the Duckiebot, the Discrete-time Algebraic Riccati Equation (DARE) has to be solved:

$\Phi = A^T\Phi A - (A^T \Phi B)(R+B^T \Phi B)^{-1}(B^T \Phi A)+Q$

To solve this equation use the Python control library (see [Python control library documentation](https://python-control.readthedocs.io/en/0.8.2/)). <br />
<br />

##### A  word on weighting
In general, it is a good idea to choose the weighting matrices to be diagonal, as this gives you the freedom of weighting every state individually. Also you should normalize your $R$ and $Q$ matrices. Choose the corresponding weights and tune them until you achieve a satisfying behaviour on the track.
To find suitable parameters for the weighting matrices, keep in mind that we are finding our control input by minimizing a cost function of the form

\begin{equation}
u_{LQR}(t) = \underset{u(t)}{\text{argmin}} \phantom{0} J_{LQR}(u(t)) = \underset{u(t)}{\text{argmin}} \phantom{0} \int_{0}^{\infty} u^TRu + x^TQx + 2x^TNu ~ dt
\end{equation}

So intuitively, one can note that a low weight on a certain state means that it has less of an impact when trying to minimize the overall cost function. A high weight means that we want to minimize this state more in order to minimize the overall function. <br />
For example, if we give a low weight on the input $u(t)$, i.e. the weighting matrix $R$ contains smaller values than the weighting matrix $Q$, the controller will care less about the input used and therefore converge to the desired reference faster. <br />
Last but not least, choosing N=0 is typical as it provides guarantees on performance and robustness.

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
TODO: Missing the part where we explain that we are not dealing with a LQR, but with a LQ"G". cioè

<end/>
