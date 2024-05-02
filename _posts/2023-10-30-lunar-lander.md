---
title: "Grid Q-Learning vs Deep SARSA for Lunar Lander"
classes: wide
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


<figure class="half">
    <a href="/assets/images/lunar_lander/animation_disc.gif"><img src="/assets/images/lunar_lander/animation_disc.gif"></a>
    <a href="/assets/images/lunar_lander/animation_sarsa.gif"><img src="/assets/images/lunar_lander/animation_sarsa.gif"></a>
    <figcaption>Left shows our grid discretization learner. Right shows Deep SARSA learner. </figcaption>
</figure>

### Introduction

The Lunar Lander environment from OpenAI is an emulation of an Atari game where the player must land a lunar vehicle on a randomly generated lunar floor. The environment handles the inputs for us, providing us with the position and coordinates of the lander, and the x, y and angular velocities in a continuous observation space. The environment will give instantaneous rewards depending on the position and velocity of the Lander, and a +100/-100 final reward for a successful/failed landing, respectively. Rewards are designed so an increased reward is earned if the Lander moves closer to the landing pad, and its velocity approaches 0. 

The variables of the state are shown below. 

$$
[x, y, v_x, v_y, θ, v_θ, leg_L, leg_R]
$$

x and y are the location of the Lander, θ is the angle and variables shown as v refer to the velocity. The first six variables are used to determine the overall flight of the Lander. The last two will be used in the landing of the lift to indicate if the left or right leg has made contact.

### Discretized Q-Learning

#### Methods
Initially, we use an off-policy Q-Learning algorithm along with a discretization scheme to attempt the problem, which means the Q update is done based on the maximum value action in the current Q table, rather than solely the on-policy action. The naive approach to solving this problem seems to be a grid discretization. The update rule we use is shown below:

$$
Q(s,a) = Q(s,a) + \alpha \left[r + \gamma \max_{a'} Q(s',a') - Q(s,a)\right]
$$

We also employ an ε-greedy exploration strategy in our implementation. For every state that the agent encounters, we will choose with probability ε to do a uniform random action from the action space. This ε is decayed as the agent continues to train on more episodes, so as to prioritize exploitation. 

#### Naive Discretization
We start by discretizing the 8-feature continuous state space into an 8-dimensional grid. We do face the curse of dimensionality by discretizing, as the size of the grid will grow exponentially with the number of features. To counteract this, we define the area associated with the goal of landing on the platform as a high value area. We use our knowledge that landing in the flat region between the flags requires the lunar lander to be oriented with legs down, and positioned near the middle of the environment. This requires all of the state position and velocity variables to be close to 0. 

Knowing this, we will define a lower and upper bound for each state variable to create a discretization grid, with the area outside the high-value area being only defined by one state. Thus, the entire observation space will be divided into ni states, and the area within the high-value area will be discretized into (n_i – 2) states, which is defined by the discretization scheme. Within the respective high-value area, we will create a “grid” of (n_i – 2) evenly divided discrete ranges. The discretization scheme is the list n, that contains the number of states for each feature. The high value regions will be fixed and set manually.

<figure>
  <img src="/assets/images/lunar_lander/Discretization.jpg" alt="Discretization Example">
  <figcaption>Example of discretizing the x and y features.</figcaption>
</figure>

We tune the discretization scheme, epsilon (initial and decay), and alpha over 1,000 episodes.

#### Results
We can see that by using discretized state space Q-learning we were able to get considerable amount of learning done, although not reaching an above our goal of a 200 average. The mean of the 1,000 episodes in the testing phase was 111.4 with a standard deviation of 140.1. 

The training data does show convergence in the training data around 10,000 episodes in, and no further learning appears to be happening. This can be due to the relatively low number of episodes and high number of states, therefore not allowing us to learn on each state effectively. Testing more granular discretization schemes while running more episodes may show better results. However, it may be difficult to pinpoint exactly which discretization bins would work best for this data without domain knowledge, which is against the spirit of reinforcement learning.

<figure>
  <img src="/assets/images/lunar_lander/discretized_training.png" alt="Discretized Q-learning Results">
  <figcaption>Discretized Q-learning Results.</figcaption>
</figure>

For the evaluation set, we set epsilon to 0 and turned off learning. Those results are shown below, with an average score of 111, which indicate that most of the episodes were actually successful landings. However, the failed landings caused large negative scores which brought down the scores significantly. 

<figure>
  <img src="/assets/images/lunar_lander/discretized_test.jpg" alt="Discretized Q-learning Test">
  <figcaption>Discretized Q-learning Test.</figcaption>
</figure>

### Deep SARSA

#### Methods
We attempt to use an on-policy Deep SARSA algorithm to train a learner. Deep SARSA employs a neural network that takes as the input space of 8 features through a fully-connected layer, outputting the expected reward of each of the 4 possible actions. The Q-value update is done on-policy, which means the Q update is done based on the policy chosen, a’, rather than maximum Q value a. The SARSA update rule is shown below: 

$$
Q(s,a) = Q(s,a) + \alpha \left[r + \gamma Q(s',a') - Q(s,a)\right]
$$

The neural network approximates the action-value function similarly to a Q-table and is defined by a set of parameters that represents the weights and biases in the network. The loss function is defined as the squared Temporal Difference error:

$$
\text{Loss} = \left[ r + \gamma Q(s', a') - Q(s, a) \right]^2
$$

On each step, we use the Adam optimizer and update the parameters in the direction that minimizes the loss. Adam was chosen as the optimizer as it has adaptive learning rates and considers momentum, which can reduce the amount of noise and promote convergence.

We continue to employ ε-greedy exploration in Deep SARSA as well, similarly to our discretized Q-learning procedure. Additionally, to avoid the effects of overfitting and deterioration of the neural network, we will store the best performing parameters in the training set for use against the testing set. 

We tune the alpha, gamma, and hidden layer sizes over 2,000 episodes.

#### Results
Our final SARSA results are shown in Fig. 9 and 10 below. The training data showed signs of convergence once we reached around 4,000 episodes, with the average rewards fluctuating around the 250 level. We maintained a minimum epsilon of 0.01 through the training process to prevent overfitting and continue exploring paths, which led to a bit of volatility in the latter half of the training data and higher errors compared to the test.

<figure>
  <img src="/assets/images/lunar_lander/sarsa_training.jpg" alt="SARSA Training">
  <figcaption>SARSA Training</figcaption>
</figure>

For the evaluation set, we set epsilon to 0 and turned off learning. We were able to solve the problem by averaging greater than 200 total rewards per episode over a 100-episode span. In fact, we averaged 256.3 with a standard deviation of 57.8 over the test of 1,000 episodes. Two strata of results began to appear, those in the 225 and above rewards and those below 200. After analyzing the animated episodes where the agent fails to achieve 200, the agent often continues firing fuel even when the Lander has landed. This behavior is often due to uneven floors and other edge cases. Further parameters can be introduced to account for these cases and allow the agent to maneuver out of them.

<figure>
  <img src="/assets/images/lunar_lander/sarsa_test.jpg" alt="SARSA Test">
  <figcaption>SARSA Test</figcaption>
</figure>

Overall, the Deep SARSA algorithm did an excellent job at training an agent to successfully navigate the Lunar Lander environment. The ability to map the continuous input state to (s, a) pairs via the neural network proved to be very effective in creating an functioning agent. 

### Comparison
Theoretically, Discretized Q-Learning and Deep SARSA are both ways that we can approximate values of Q(s,a) through learning from experience, that will converge given enough experiences. The issue with the discretized Q-Learning procedure is the difficultly in finding a useful discretization scheme such that a high-performing agent could be achieved. I could see how using a discretized process could be a viable choice in lower dimensional domains. It would be interesting to see if additional runs and varying the “high-value” area would yield results.

Deep SARSA likely performed better in the Lunar Lander environment is because the neural network was able to learn complex patterns through backpropagation and without explicit mapping to features like we did with discretized Q-Learning. Although vanilla Q-Learning would work well for small state spaces, a large continuous state space appears to be handled much better by a parametric function approximator such as a neural network.
