---
defaults:
  - scope:
      path: ""
      type: posts
    values:
      layout: single
      author_profile: true
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

\[ [x, y, v_x, v_y, \theta, v_{\theta}, \text{leg}_L, \text{leg}_R] \]

x and y are the location of the Lander, θ is the angle and variables shown as v refer to the velocity. The first six variables are used to determine the overall flight of the Lander. The last two will be used in the landing of the lift to indicate if the left or right leg has made contact.

### Discretizing the State Space
#### Methods
Initially, we use an off-policy Q-Learning algorithm along with a discretization scheme to attempt the problem, which means the Q update is done based on the maximum value action in the current Q table, rather than solely the on-policy action. The update rule we use is shown below:
`Q(s,a) = Q(s,a) + \alpha [r + \gamma \max_{a'} Q(s',a') - Q(s,a)]`
We also employ an ε-greedy exploration strategy in our implementation. For every state that the agent encounters, we will choose with probability ε to do a uniform random action from the action space. This ε is decayed as the agent continues to train on more episodes, so as to prioritize exploitation. 

#### Discretization
We start by discretizing the 8-feature continuous state space into an 8-dimensional grid. We do face the curse of dimensionality by discretizing, as the size of the grid will grow exponentially. To counteract this, we define the area associated with the goal of landing on the platform as a high value area. We use our domain knowedge that landing in the flat region between the flags requires the lunar lander to be oriented with legs down, and positioned near the middle of the environment. This requires all of the state position and velocity variables to be close to 0. 

Knowing this, we will define a lower and upper bound for each state variable to create a discretization grid, with the area outside the high-value area being only defined by one state. Thus, the entire observation space will be divided into ni states, and the area within the high-value area will be discretized into (n_i – 2) states, which is defined by the discretization scheme. Within the respective high-value area, we will create a “grid” of (n_i – 2) evenly divided discrete ranges. The discretization scheme is the list n, that contains the number of states for each feature. The high value regions will be fixed and set manually.

<div class="imgcap">
<img src="/assets/images/lunar_lander/Discretization.jpg">
<div class="thecap">Example of discretizing the x and y features.</div>
</div>