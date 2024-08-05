---
title: "Overcooked AI with Multi-Agent Reinforcement Learning"
classes: wide
defaults:
  # _posts
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

<figure class="half">
    <a href="/assets/images/overcooked/gif_layout3.gif"><img src="/assets/images/overcooked/gif_layout3.gif"></a>
    <a href="/assets/images/overcooked/gif_layout4.gif"><img src="/assets/images/overcooked/gif_layout4.gif"></a>
</figure>

<figure class="half">
    <a href="/assets/images/overcooked/gif_layout2.gif"><img src="/assets/images/overcooked/gif_layout2.gif"></a>
    <a href="/assets/images/overcooked/gif_layout5.gif"><img src="/assets/images/overcooked/gif_layout5.gif"></a>
    <figcaption>Results from Overcooked Multi-Agent Learner </figcaption>
</figure>

### Summary
We explore solving the Overcooked AI Gym environment (https://github.com/HumanCompatibleAI/overcooked_ai) using a Multi Agent Reinforcement Learning method based on a function of the individual agents' Q-values. We implement a system of two separate agents with a shared Q-value based on the weighted sum of the Q-values of the independent agents. This system was able to perform substantially better, clearing all five layouts, and performing especially well on the ones requiring cooperation (shown above).

## Problem Statement
The Overcooked AI environment is a simplified replication of a multi-player cooperative video game. Rewards are sparse, with only environment rewards coming from the completion of a soup delivery. Unlike single agent problems, the Overcooked problem has a distinct observation space for each agent. Execution must be decentralized while training can be based on global observations. The difficulty of this problem comes from the proper attribution of rewards and the need to train each agent with total rewards from the environment, while also connecting the causality of the rewards to each agent’s actions. We opt to go for a modified approach of Value Decomposition Networks of combining the Q-values of the two separate agents.


## Value Decomposition Setup
### Reward Shaping
The first step in building our multi-agent network is to define a reward shaping system that is suitable for our multi-agent environment. We use the reward shaping to impart domain knowledge of the environment to the agents. The global reward from these events will have to be tuned to be large enough to noticeably improve learning but not too large that they interfere with the global objective. The rewards must also not be exploitable and counterproductive.

<img src="/assets/images/overcooked/vdn_rewards.png" alt="Reward Shaping" width="500"/>

After some ad-hoc testing in our simple environments, we decided on adding the rewards shown below:
- +3: begin cooking with 3 onions
- -3: begin cooking with <3 onions
- +0.05: useful onion/dish pickup/drop
- -1: dropping a soup
- +0.5: viable onion potting

### Value Decomposition (VDN) Setup and Methods
Next, we implement value decomposition to allow for Q-values to be shared between the two agents. See below for our implementation. We will take a weighted sum of both agents’ Q-values for all state-action pairs. The sum is combined into a per-agent Q’, which is the blended sum. We calculate optimal actions and loss from this Q’. The loss is backpropagated through the new Q’, allowing each agent to factor the Q-values of the other.

The replay buffer will be updated so it will take both agents’ states and actions, thereby allowing agents to centrally train. In execution, however, the best action will be taken independently by each agent to maximize their own Q’.

<img src="/assets/images/overcooked/vdn_implementation.png" alt="VDN Implementation" width="500"/>

The formulas describing the calculation of the Q’ values are shown below. Note we will have to select weights \(w\) for the Q’ sum. For our experiments, we will use \(w_1, w_2 = 1\) as it is straightforward and symmetry of rewards.

$$
Q_1'(s_1, s_2, a_1, a_2) = Q_1(s_1, a_1) + w_1 \cdot Q_2(s_2, a_2)
$$

$$
Q_2'(s_1, s_2, a_1, a_2) = w_2 \cdot Q_1(s_1, a_1) + Q_2(s_2, a_2)
$$

$$
\text{Loss}_1 = (R_1 + R_2 + \gamma \max_{a'} Q_1'(s', a') - Q_1'(s, a))^2
$$

$$
\text{Loss}_2 = (R_1 + R_2 + \gamma \max_{a'} Q_2'(s', a') - Q_2'(s, a))^2
$$

## Hyperparameter Tuning
We tune alpha, gamma and epsilon decay hyperparameters using a Grid Search across the first 1000 episodes. 

### Training Performance
Our training performance (measured by soups made per episode) for layouts [1, 2, 3, 5] are shown  below. We were able to hit our target 7 soups delivered for [1, 2, 3, 5] during training. We note that the learning curves for layouts 1 to 3 were quite monotonic and smooth, while layout 5 has a more sporadic learning curve. While learning layout 5, our agent had a much more difficult time learning how to deliver a single soup, and took until around 1500 episodes before it was able to reliably generate soups. This was expected because layout 5 requires the agents to cooperate in order to achieve the goal.

<img src="/assets/images/overcooked/vdn_training.png" alt="VDN Training" width="200"/>

## Conclusion and Further Work
We saw the importance of global reward shaping and tailoring a natively multi-agent approach to this problem, where we saw a drastic performance improvement – despite using the same base DQN for the agents. The learning curve and peak performance was drastically improved from the inclusion of these elements. Based on the promising initial results, we believe a cleverer execution of the value decomposition method and further tuning of the reward mechanism could result in even better performance. Further work in the decomposition should allow for better credit assignment, which would improve our agent performance in layouts that require higher coordination.

1. Continue to modify the reward structure to capture more subgoal completions, such as when a completed soup moves closer to the serving area. One possible addition is using dish drops, mentioned in section IV.
2. Increase the number or size of hidden layers in each DQN to allow them to capture more complex relationships between features.
3. Adjust the Q’ weights systematically to assign credit and rewards to each agent with greater weight to their contribution.
4. Implement an additional neural network to select the mixing weights of the individual Q-networks, such as that used in QMIX [4].

## References

[1] Richard S Sutton and Andrew G Barto. Reinforcement learning: An introduction. 2nd Ed. MIT press, 2020.

[2] V. Mnih, K. Kavukcuoglu, D. Silver, A. Graves, I. Antonoglou, D. Wierstra, and M. Riedmiller, “Playing Atari with Deep Reinforcement Learning,” 2013.

[3] Sunehag, P., Lever, G., Gruslys, A., Czarnecki, W. M., Zambaldi, V., Jaderberg, M., et. al.. “Value-Decomposition Networks For Cooperative Multi-Agent Learning”, 2017

[4] Rashid, T., Samvelyan, M., De Witt, C. S., Farquhar, G., Foerster, J., & Whiteson, S. (2020). Monotonic value function factorisation for deep multi-agent reinforcement learning. The Journal of Machine Learning Research, 21(1), 7234-7284.

[5] Carroll, M., Shah, R., Ho, M. K., Griffiths, T. L., Seshia, S. A., Abbeel, P., & Dragan, A. D. (2019). "On the Utility of Learning about Humans for Human-AI Coordination". arXiv preprint arXiv:1910.05789. GitHub repository: [Human Compatible AI Overcooked AI Environment](https://github.com/HumanCompatibleAI/overcooked_ai).