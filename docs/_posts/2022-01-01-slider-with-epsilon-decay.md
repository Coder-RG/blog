---
layout: post
title: Q-Learning with Epsilon-greedy
author: Coder-RG
categories: reinforcement-learning
---

In this article, we will implement Q-learning with epsilon-decay. We implemented
Q-learning in the previous article and therefore won't be going into any detail.
Come back after reading [this][slider] article if you haven't already.

**what is epsilon-greedy?**

It is an algorithm in reinforcement learning
that controls the agent's exploration v/s eploitation tradeoff.

**Why is this a tradeoff?**

Because eploration helps the agent find the best possible actions, with a low episodic
reward being the possibility. Eploitation on the other hand refers to taking
actions that maximise the short-term reward. This becomes an issue when the agent
has not explored its environment and takes actions based on its current knowledge
when it is possible that a higher reward action might be unexplored.

## Epsilon-Greedy Algorithm



## Performance

![Epsilon-Greedy]({{site.baseurl}}/assets/images/SliderEpsilonDecay0.png)

![Epsilon-Greedy]({{site.baseurl}}/assets/images/SliderEpsilonDecay1.png)

![Epsilon-Greedy]({{site.baseurl}}/assets/images/SliderEpsilonDecay2.png)

[slider]: {{site.baseurl}}/{% post_url 2021-12-18-slider %}