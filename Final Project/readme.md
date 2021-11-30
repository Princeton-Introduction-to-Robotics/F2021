# MAE345/549: Final Project

For the final project in this class, you will write a program that guides the
Crazyflie from one end of the netted area to the other. To navigate, each Crazyflie is outfitted with a PoV camera capable of live streaming a view from the Crazyflie to your computer. This project is _open ended_, and you are free to use any approach to complete the task at hand.

## Test Scripts

The instructors have provided two example Python scripts based on previous Lab notebooks. If you are new to Python, these scripts are more like conventional programs you may have written in the past than the Jupyter notebooks we used throughout the semester in that all the code in them is executed at once. Both can be run by entering `python <scriptname>` in your terminal or by copying the text into a Jupyter notebook.

The script `test_camera.py` uses OpenCV to open a video device on your computer and plays frames from it in a computer; this is from Lab8. Depending on your computer, the video encoder may be video device 0 or 1 (your webcam might be assigned to the other number). By default, the script reads frames from video device 0, so you may need to change it to 1 manually.

The script `test_cf.py` is a minimal implementation of the full control loop of the Crazyflie based on Lab9. It does the following:

1. Connects to the Crazyflie.

2. Reads frames from the video encoder for five seconds. This removes any black frames that may be received while the radio searches for the camera's frequency.

3. Ascends from the ground.

4. Reads a frame and processes with OpenCV. The processing is broken into the following steps:

  a. Converts the frame from BGR (OpenCV's default) to HSV, which is a more convenient color space for filtering by color.

  b. Applies a mask that creates an image where a pixel is white if the corresponding pixel in the frame has an HSV value in a specified range and black otherwise. The color range was tuned by the instructors to match the red of the obstacles, but different lighting conditions and cameras may require further tuning.

  c. Applies OpenCV's contour detection algorithm to the masked image. This finds the edges between regions of color in an image.

  d. Checks the area (in pixels squared) of the largest contour. If it's greater than a threshold, the Crazyflie is instructed to move right and start Step 4 over. Otherwise, we move on to step 5.

5. The Crazyflie is instructed to land.

Thus, this script causes the drone to move right until it no longer sees a red object of significant size. To run this script, you will need to change the URI in the file to match your drone / group number. Additionally, this file contains a function `position_estimate` which you may fund useful. The function shows an example of retreiving the crazyflie's position estimate.

## Lab Setup

The lab setup is similar to the RRT lab, with two additions:
- In addition to the PVC pipe obstacles, there will also be obstacles in the form of hula hoops (these are also colored red).
- There will be a target object (a book) placed on a table at the end of the obstacle course. A portion of your grade will be based on landing near the target object (see below for details on grading).

You are free to move the obstacles around for testing purposes, but do not remove them from the lab spaces. 

## Demo Day and Grading

We will hold a Demo Day for evaluating final projects. This will be held on Dean's Date (Tuesday, December 14th). At the beginning of December, we will send out a sign-up sheet for the Demo Day. Each team will sign up for a time-slot (20 minutes) and will have three attempts at the obstacle course. Each team will also explain the technical approach that they took to the course staff and will have to submit code for the project. We will use the three netted zones (two in ACEE and one in G105) for the Demo Day; you will be able to choose which zone to use for the demo (to ensure that the lighting conditions are similar to what you assumed when programming the drone). 

Your score on each of the three trials will be based on the following criteria:

1. (80 pts) Distance along the x (i.e., forward) direction your robot traversed before colliding. In particular, the score for a trial will be the fraction of the course your robot successfully traversed before colliding, e.g., (80 pts) * 70/100 if your robot covered 70 percent of the course before colliding ("colliding" is defined as the point at which your robot first touches/hits an obstacle or the ground, or leaves the allowable flying zone marked with tape).
2. (20 pts) Landing on or near the target. The target object will be a book of your choice (that you will bring to the Demo Day). The only restriction on the book is that its length and width should be less than one foot (12 inches); any standard book should satisfy these criteria. The book will be placed by the instructors on the table at the end of the course. The book may be placed "upright" (if you like) to make it easier for the drone to see the book when it is flying. If your drone lands on the table within 15cm of the book (as measured from the closest point on the drone to the closest point on the book), you will receive 20 points. If your drone lands within 30cm of the book, you will receive 10 points. No points will be awarded for this portion if your drone lands more than 30 cm away from the book. 

Each of the three trials will be scored based on the two criteria above. Note that the two criteria are not completely independent; reaching the target relies on successful navigation through the obstacle course. However, it is possible that your drone collides with an obstacle/ground (or leaves the allowable flying area marked with tape) and somehow manages to keep going and land successfully near the target. In this case, you will receive full points for landing (assuming this was successful), but will receive points for navigation based on where your robot first experienced a collision or left the flying zone, e.g., (80 pts) * 70/100 if your robot covered 70 percent of the course before first touching/hitting an obstacle/ground or leaving the allowable flying zone.

**Your total score will be the average of the scores from the three trials.** The rationale for averaging the scores from the trials is to evaluate the reliability of your system. 

## Some Suggested Approaches

### Identifying the target

For identifying the target (book), you can use the pretrained neural networks you used in Assignment 9. 

### Obstacle avoidance and navigation

One option is to construct an estimate of obstacle locations in order to build a map, and then use a sampling based motion planner to navigate toward the goal region. You will need to periodically recompute your motion plan as you gather more information about obstacle locations.

Another option is to do what is known as reactive planning. In reactive planning, you compute a control action to take based on the current context of the system. Notably, reactive planners often do not need a complete map to function. While there are many ways you could implement a reactive planner, optical flow is particularly useful tool for this approach. A simple planner might steer the drone toward regions of it's visual field where time to collision is high. You can also use the fact that obstacles are colored red in order to identify them in the image. 

## Advice From The Instructors

- Don't reinvent the wheel. Unlike previous assignments, you are not restricted in the libraries and techniques you may use to approach this challenge. We especially recommend that you use OpenCV to simplify things as much as possible. For example, if you want to use optical flow to compute the time to collision of your drone with an obstacle, OpenCV has a very good implementation of the optical flow algorithm. Similarly, the script `test_cf.py` makes good use of OpenCV's contour detection algorithm. You are welcome to look up documentation / information on the use of OpenCV, Numpy, etc.

- You can test your vision loop separately from your control code. If you think you are having trouble identifying the obstacles, you can disable your code to fly the drone and display what is happening at different points in your vision pipeline using OpenCV's imshow function. This way you can confirm your drone is seeing what you think it is seeing. You may also be able to do this while the drone is flying, but the instructors had mixed results due to some lag it introduced in our setup.

- You may want to consistently use either the space in G105 or ACEE for testing. This will ensure that lighting conditions are consistent (note that you will be able to choose whether you do the Demo in G105 or ACEE). 

- Start simple. Begin with just moving around one obstacle. Once that is working reliably, add a second and a third.

- Start early. As you may have learned during the hardware labs, getting robots to work in the real world is tricky. Get started early to give yourself the best chance of success on the project! 

We look forward to seeing the approaches you come up with!
