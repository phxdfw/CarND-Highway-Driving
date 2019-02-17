# CarND-Path-Planning-Project
Self-Driving Car Engineer Nanodegree Program

### MODELS
#### Brake
When it needs braking, if the distance is very close to the car ahead, i.e. < `0.6 * SAFE_DISTANCE`, I used `-MAX_ACC` as the max possible value to brake the car; if the distance is not very close yet, I designed a linear function (when distance is `0.6 * SAFE_DISTANCE`, brake is `-MAX_ACC`; when distance is `SAFE_DISTANCE`, brake is `0`) to apply the brake value. Therefore the braking is always within the allowed range, and it will also change continously depending on the distance from the car ahead without too much jerk. See src/main.cpp Line 310.

#### Acceleration
When it needs speeding up, if the current velocity `ref_vel` is far from the speed limit, i.e. < `MAX_SPEED - 5.0`, I used `MAX_ACC` as the max allowed acceleration; if `ref_vel` is close to the speed limit, the acceleration gradually drops down to zero using another linear function (when velocity is `MAX_SPEED - 5.0` mph, acceleration is `MAX_ACC`; when the velocity is `MAX_SPEED`, the acceleration is `0`). Therefore the acceleration is also always within the allowed range, and changes continously with the current velocity. So the jerk is also small here. See src/main.cpp Line 322.

#### Change Lane
* There are flags `car_left` and `car_right` that could be marked `true`, when there is a car in the corresponding lane within the range `[car_s - 0.5 * SAFE_DISTANCE_LANE_CHANGE, car_s + SAFE_DISTANCE_LANE_CHANGE]`.
* The change lane logic is triggered when a car is ahead within `SAFE_DISTANCE` and if it is possible to do so. It will always prefer the left lane to the right lane. It takes higher priority than the braking logic, which applies only when it could not change lane.
* If the car is on left or right lane and the middle lane is empty, it will switch back to the middle lane. The reason to do so is that next time when approaching a slow car in front of it, it will have more opportunity to change lane.
* See src/main.cpp Line 297.

### STEPS
1. I followed the course video of "Project Q&A" step by step to get started, including:
  * Make the car runnable on the simulator (in a simple straight line)
  * Make use of `prev_path` data to handle the signal delay
  * Keep the car inside lane #1 by the `spline` curve fitting tool and `getXY` conversion
  * Avoid collision to the car around, by using sensor fusion data and change `ref_vel` when `too_close`
  * Avoid cold start, i.e. making the car speeding up gradually from `0` mph to `MAX_SPEED = 49.5` mph at the beginning within the max allowed acceleration

2. I optimized the brake and acceleration to be a continous variable (when remaining in the lane)
  * Please see the MODEL part above for Brake and Acceleration sections

3. I added some logic to let the car to change lane
  * I grouped the sensor fusion data to 3 lanes (and neglecting data if outside the 3 lanes in the car direction), based on their `d` values
  * Please see the MODEL part above for Change Lane section

4. I ran the simulator again and again, to debug and update the parameters I set
  * At first the speed always go beyond `MAX_SPEED`, and then I found it is by a bug in the linear function in my acceleration model, i.e. the acceleration needs to be well defined when outside the linear range.
  * Then I found the car is very reluctant to change its lane. After updating the setting on `SAFE_DISTANCE_LANE_CHANGE` and its range, then the problem gets solved.
  * Then I also found sometimes the car could not be slowed down in time and hits the car ahead. After adjusting the linear function from apply max brake at `0.3 * SAFE_DISTANCE` to `0.6 * SAFE_DISTANCE`, it drives much better.

### Problems Left
Sometimes in the simulation when a slow car from another lane suddenly switch to my lane, with a very short distance ahead of my car, it will be hit by my car. This does not happen frequently, actually it is not easy to reproduce. But I still think it is worthy to mention. I feel this is because of my car's mechanical limit on braking so it could not slow down too fast, and I did not find a good way to solve that issue. Is it worthy to drive much slower than speed limit when no car is blocking me, just to be defensive enough if the slow car from nearby lane might suddenly jump into my lane? I don't think human drivers will do that all the time.