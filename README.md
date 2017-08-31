# CarND-Path-Planning-Project
Self-Driving Car Engineer Nanodegree Program
   
### Overview

There are a number of ways to solve the path planning project. My first assumption was to build a complicated state machine like in the previous class quizzes.  After implementing the simple center line following from the tutorial video, I realized that a full blown state machine would be overkill.  I am amazed at how well it operates with this simple model.

There are very few states that are necessary to solve the basic version of this problem.

There are two main parts to my soltuion code:

1. Spline Route Planner (from Tutorial)
2. Lane Following and Lane Changing Logic

### Spline Route Planner

The tutorial did a great job explaining how to build a simple route using the spline.h library.  The code creates 3 new waypoints based on a combination of the map path, and the lane we are targetting, plus one previous point from either the last prediction, or current position of the car.  It then fits a spline to these waypoints.  Next it breaks the path into equal segments (unless the car is accelerating or decelerating) and uses the splines to kick out a path that contains 50 points. (It may not need to create all 50 points if there are already some points left in the prediction).

The prediction currently adds points to the end of the array. So there is latency involved in changing the plan. If the number of points was much more than 50 this would be a huge problem. If the car was on a city street and a ball bounced in front, it also might not work well.  On the highway this seems reasonable, and reduces jerks from replanning the route.

### Lane Following and Lane Changing Logic

At the end of the tutorial there was so logic added to detect if there is a car in front of our ego vehicle - within a set distance.

I moved that code into a helper function to simplify the logic. I created a new function called isLaneAvailable.  In this constrained highway problem (and in the interest of getting this project done quickly since I am so late) I decided on a simple logic loop instead of a more complicated state machine, and/or code that assigns weights to the different choices.  

The logic is documented in this comment and implemented below:

            /*
            # Control logic
            # 
            #  1 - check to see if we are too close, 
            #      and set the flag so we can slow down
            #  2 - if we are too close, then first try to 
            #      switch into the lane left of us - 
            #      we use a helper to see if the lane exists, 
            #      and has a gap that is big enough
            #  3 - if the left lane doesn't work, try the 
            #      next one right
            */
         
            too_close = isCarInFront(cars,prev_size,car_s);
            if (too_close)
            {
                 // try to go left first
                 if (isLaneAvailable(cars, prev_size,car_s, lane-1))
                 {
                    lane=lane-1; 
                    std::cout << "go left" << std::endl;
                 }
                 else if (isLaneAvailable(cars, prev_size,car_s, lane+1))
                 {
                    lane=lane+1;
                    std::cout << "go right" << std::endl;
                 }
            }


I struggled with the isLaneAvailable function at first, I didn't have the boolean logic correct on finding an empty slot. Once I drew a diagram of the lanes and put in some sample values it was very straightforward.

### Possible improvements

We can set an additional rule to attempt to move the car back to the center lane after it passes. (ie: an else after the "if (too_close)" stage). The thought would be that the center lane has two options to get around a blocking car, left or right. While if you are in the left or right lane you only have one option.  In practice in the simulator the cars all seem to bunch up, I decided the current solution is good enough.


