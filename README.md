# CarND-Controls-MPC
Self-Driving Car Engineer Nanodegree Program

---


[//]: # (Image References)
[img1]: ./img/car_model.png
[img2]: ./img/mpc_10pt.png





## Project Summary
In this project, a MPC controller is created to make a simulated self-driving follow desired track. The MPC controller is formulated as a nonlinear optimization problem, which is solved with an open source solver [Ipopt](https://projects.coin-or.org/Ipopt). The implemented MPC controller is able to drive the vehicle to follow desired track with an average speed of 80 mph.


The development environment is set up with the udacity/controls_kit docker image. To activate the docker container, run the following command. 
```
docker run -it -p 4567:4567 -v $USER_PATH:/work udacity/controls_kit:latest
```

### Model description
The model of the vehicle is a kinematic bicycle. The states of the model are chosen as: x, y positions, yaw angle, velocity, cross track error, and yaw angle error. The actuators for the model are steering angle and acceleration. The update equation for the kinematic bicycle model is given as follows.
![car_model][img1]

This model is simple yet gives us a good abstraction of the actual car dynamics. Our MPC controller is designed based this kinematic bicycle model. Since the MPC controller executed periodically with real-time sensing data, it can effectively compensate for tracking error with feedback.

* Tuning MPC controller parameters
The MPC controller has many parameters. The tuning process is mostly by trial and error. However, there are also some intuitions we can follow to speed up the process. The controller steps N=10 and dt=0.1s are selected to achieve the best performance of the car. Larger N*dt extends the look ahead horizon of the controller, so that we can better optimize for the overall performance. However, dt needs to be smaller so that we can better approximate the nonlinear dynamics of the car. At the same, N cannot be too large because it will introduce too many optimization variables that cannot be computed within allowed amount of time. The final selection reaches a balance between performance, computation time, and accuracy.

* Other important tuning parameters are the cost weights for different cost components. The following set of weights gives our controller the best tracking behavior. It allows the car to reach an average speed of 80mph without running off the road.
```
    AD<double> lateral_cost = 3000;
    AD<double> heading_cost = 2000;
    AD<double> v_cost = 1;
    AD<double> control_delta_cost = 1;
    AD<double> control_a_cost = 1;
    AD<double> control_delta_rate_cost = 5;
    AD<double> control_a_rate_cost = 1;
```

* MPC reference generation
The track information is provided as a set of waypoint along the way. A third order polynomial is used to fit waypoints. Then this fitted polynomial is used to extract reference points for the car to follow. The MPC controller solves a nonlinear optimization problem to make the MPC control points to follow the reference points subject to the vehicle dynamics constraint. The reference points and actual MPC control points are visualized as yellow and green lines in the simulator. (See the video below for detail)

* MPC with latency of 100 ms
The simulated car has a delay of 100 ms in actuation. If the delay is not compensated, the  control actions will always be 100ms late than desired. This will cause the car to oscillate around the desired path. This actuation is easily compensated with MPC. We first propagate the current state forward for 100ms, then feed this state as initial state to the MPC controller. In this way, the desired control action after delay is computed correctly. 


### Final video (click on the image to view the youtube video)
The rendering of the simulator takes a large percentage of the CPU. Thus the MPC controller can only run at a limited frequency. But with our efficient implementation, the MPC controller can drive the car safely at an average speed of 80 mph.
[![mpc_video][img2]](https://www.youtube.com/watch?v=SYYWxPRTjLM)


## Dependencies

* cmake >= 3.5
 * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1(mac, linux), 3.81(Windows)
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets
    cd uWebSockets
    git checkout e94b6e1
    ```
    Some function signatures have changed in v0.14.x. See [this PR](https://github.com/udacity/CarND-MPC-Project/pull/3) for more details.

* **Ipopt and CppAD:** Please refer to [this document](https://github.com/udacity/CarND-MPC-Project/blob/master/install_Ipopt_CppAD.md) for installation instructions.
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page). This is already part of the repo so you shouldn't have to worry about it.
* Simulator. You can download these from the [releases tab](https://github.com/udacity/self-driving-car-sim/releases).
* Not a dependency but read the [DATA.md](./DATA.md) for a description of the data sent back from the simulator.


## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./mpc`.

## Tips

1. It's recommended to test the MPC on basic examples to see if your implementation behaves as desired. One possible example
is the vehicle starting offset of a straight line (reference). If the MPC implementation is correct, after some number of timesteps
(not too many) it should find and track the reference line.
2. The `lake_track_waypoints.csv` file has the waypoints of the lake track. You could use this to fit polynomials and points and see of how well your model tracks curve. NOTE: This file might be not completely in sync with the simulator so your solution should NOT depend on it.
3. For visualization this C++ [matplotlib wrapper](https://github.com/lava/matplotlib-cpp) could be helpful.)
4.  Tips for setting up your environment are available [here](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/0949fca6-b379-42af-a919-ee50aa304e6a/lessons/f758c44c-5e40-4e01-93b5-1a82aa4e044f/concepts/23d376c7-0195-4276-bdf0-e02f1f3c665d)
5. **VM Latency:** Some students have reported differences in behavior using VM's ostensibly a result of latency.  Please let us know if issues arise as a result of a VM environment.

## Editor Settings

We've purposefully kept editor configuration files out of this repo in order to
keep it as simple and environment agnostic as possible. However, we recommend
using the following settings:

* indent using spaces
* set tab width to 2 spaces (keeps the matrices in source code aligned)

## Code Style

Please (do your best to) stick to [Google's C++ style guide](https://google.github.io/styleguide/cppguide.html).

## Project Instructions and Rubric

Note: regardless of the changes you make, your project must be buildable using
cmake and make!

More information is only accessible by people who are already enrolled in Term 2
of CarND. If you are enrolled, see [the project page](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/f1820894-8322-4bb3-81aa-b26b3c6dcbaf/lessons/b1ff3be0-c904-438e-aad3-2b5379f0e0c3/concepts/1a2255a0-e23c-44cf-8d41-39b8a3c8264a)
for instructions and the project rubric.

## Hints!

* You don't have to follow this directory structure, but if you do, your work
  will span all of the .cpp files here. Keep an eye out for TODOs.

## Call for IDE Profiles Pull Requests

Help your fellow students!

We decided to create Makefiles with cmake to keep this project as platform
agnostic as possible. Similarly, we omitted IDE profiles in order to we ensure
that students don't feel pressured to use one IDE or another.

However! I'd love to help people get up and running with their IDEs of choice.
If you've created a profile for an IDE that you think other students would
appreciate, we'd love to have you add the requisite profile files and
instructions to ide_profiles/. For example if you wanted to add a VS Code
profile, you'd add:

* /ide_profiles/vscode/.vscode
* /ide_profiles/vscode/README.md

The README should explain what the profile does, how to take advantage of it,
and how to install it.

Frankly, I've never been involved in a project with multiple IDE profiles
before. I believe the best way to handle this would be to keep them out of the
repo root to avoid clutter. My expectation is that most profiles will include
instructions to copy files to a new location to get picked up by the IDE, but
that's just a guess.

One last note here: regardless of the IDE used, every submitted project must
still be compilable with cmake and make./
