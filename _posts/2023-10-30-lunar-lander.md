---
defaults:
  - scope:
      path: ""
      type: posts
    values:
      layout: single
      author_profile: false
      read_time: true
      comments: true
      share: true
      related: true
---
### Summary
We explore two approaches to solving the Lunar Lander simulation provided by OpenAI: the first approach employs Discretized Q-Learning (discretized QL), and the second utilizes Deep SARSA with a neural network approximator. We achieve a considerable amount of learning in discretized Q-Learning and reached an average reward of 111.4. Our implementation of Deep SARSA performed better, solving the environment with an average reward of 256.3.

### Introduction

The Lunar Lander environment from OpenAI is an emulation of an Atari game where the player must land a lunar vehicle on a randomly generated lunar floor. The environment handles the inputs for us, providing us with the position and coordinates of the lander, and the x, y and angular velocities in a continuous observation space. Additionally, the environment will give instantaneous rewards depending on the position and velocity of the Lander, and a +100/-100 final reward for a successful/failed landing, respectively. Rewards are designed so an increased reward is earned if the Lander moves closer to the landing pad, and its velocity approaches 0. 

The variables of the state are shown below. 

[x,y,v_x,v_y,θ,v_θ,leg_L,leg_R]

x and y are the location of the Lander, θ is the angle and variables shown as v refer to the velocity. The first six variables are used to determine the overall flight of the Lander. The last two will be used in the landing of the lift to indicate if the left or right leg has made contact.

Going off traditional 