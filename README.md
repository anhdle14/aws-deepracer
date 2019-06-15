# AWS DeepRacer
---

## Introduction 

> A trip down to Self-driving car, this time AWS DeepRacer.

- [AWS Docs](https://docs.aws.amazon.com/deepracer/latest/developerguide/what-is-deepracer.html)

## Review

So far my experiences the AWS DeepRacer Simulator is quite buggy, and basic. 
And most of the leauge top 10 are basically max speed with predefined waypoints that you can get from logs. The fatest i can get is around 15s~16s (on AWS re:Invent track).

AWS DeepRacer has 30USD for free tier so it is around 10 hours of free training. And the maximum to train a model is 480 minutes so you do not have a lot of play time and test time. Most of my models are less than 200 minutes of training, and i can not retrained the model either. So most of the time my model can only run on 1 track, when you evaluate on another track, they have very bad results.

## Trials
---

### Disclaimers:

- I have only tried to run on AWS Re:Invent 2018 and Kumo Torakku tracks.

### Things to considered

```python
{
  "all_wheels_on_track": Boolean,    # flag to indicate if the vehicle is on the track
  "x": float,                        # vehicle's x-coordinate in meters
  "y": float,                        # vehicle's y-coordinate in meters
  "distance_from_center": float,     # distance in meters from the track center 
  "is_left_of_center": Boolean,      # Flag to indicate if the vehicle is on the left side to the track center or not. 
  "heading": float,                  # vehicle's yaw in degrees
  "progress": float,                 # percentage of track completed
  "steps": int,                      # number steps completed
  "speed": float,                    # vehicle's speed in meters per second (m/s)
  "streering_angle": float,          # vehicle's steering angle in degrees
  "track_width": float,              # width of the track
  "waypoints": [[float, float], … ], # list of [x,y] as milestones along the track center
  "closest_waypoints": [int, int]    # indices of the two nearest waypoints.
}
```

### `SelfMotivator` based on [scottpletcher](https://github.com/scottpletcher/deepracer)

```python
def on_track_reward(all_wheels_on_track, progress, steps, speed):
    if all_wheels_on_track and steps > 0:
        reward += ((progress / steps) * 100) + (speed ** 1.5) # I changed this down for Kumo Torakku because with 2.0 modifier the car never able to complete the track.
    else:
        reward = 1e-2
    return float(reward)
```

### `FocusOnWaypoint`

This should combine with others

```python 
import math

def direction_reward(reward, waypoints, closest_waypoints, heading):
    next_point = waypoints[closest_waypoints[1]]
    prev_point = waypoints[closest_waypoints[0]]

    # Calculate the direction in radius, arctan2(dy, dx), the result is (-pi, pi) in radians
    direction = math.degrees(math.atan2(next_point[1] - prev_point[1], next_point[0] - prev_point[0]))

    # Cacluate difference between track direction and car heading angle
    direction_diff = abs(direction - heading)

    # Penalize if the difference is too large
    if direction_diff > 30:
        reward *= 0.5

    return reward
```

### `SteeringReward`

![](https://i.redd.it/cu3g1o1uqhy21.jpg)

```python
def steering_reward(reward, steering, is_left_of_center, speed):
    # For hard turn:
    if steering < -25 and is_left_of_center == False and speed < 3 or steering > 25 and is_left_of_center == True and speed < 3:
        reward *= 1.2
    else:
        reward *= 0.8

    # For soft turn
    if abs(steering) < 15 and speed < 4:
        reward *= 1.1
    else:
        reward *= 0.9

    # For straight
    if abs(steering_angle) < 0.1 and speed > 4:
        reward *= 1.4

    return reward
```

### Tips

- Do not over complicated the functions.
- You can combine multiple reward functions.
- Make it run 100% first than make it faster.
- Check the Log Visualize tools from AWS repo.




