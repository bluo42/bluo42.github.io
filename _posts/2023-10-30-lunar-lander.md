# Lunar lander

### Summary
This paper explores two approaches to solving the Lunar Lander simulation provided by OpenAI: the first approach employs Discretized Q-Learning (discretized QL), and the second utilizes Deep SARSA with a neural network approximator. We find that while discretized Q-Learning achieves a respectable average reward of 111.4, Deep SARSA outperforms it significantly, achieving an average reward of 256.3.

### Introduction

The Lunar Lander environment from OpenAI is an emulation of an Atari game where the player must land a lunar vehicle on a randomly generated lunar floor. The environment handles the inputs for us, providing us with the position and coordinates of the lander, and the x, y and angular velocities in a continuous observation space. Additionally, the environment will give instantaneous rewards depending on the position and velocity of the Lander, and a +100/-100 final reward for a successful/failed landing, respectively. Rewards are designed so an increased reward is earned if the Lander moves closer to the landing pad, and its velocity approaches 0. 

Unlike discrete problems in simpler environments, the Lunar Lander observation space is continuous, presenting a problem for us in using traditional methods such as Q-Learning and SARSA. We initially try using state space grid discretization for Q-Learning. When this does not meet the requirements for solving the environment, we choose to employ a neural network to map the observation space to the actions in a non-linear fashion. 

The variables of the state are shown below. 

[x,y,v_x,v_y,θ,v_θ,〖leg〗_L,〖leg〗_R]
