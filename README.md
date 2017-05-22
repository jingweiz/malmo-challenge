# The Malmo Collaborative AI Challenge

# **bacon-reloaded**

This is **bacon-reloaded**:P Here's our summary for our work for [**The Malmo Collaborative AI Challenge**](https://www.microsoft.com/en-us/research/academic-program/collaborative-ai-challenge/#).

*******
# Summary

## Approach
* Asynchronous Advantage Actor-Critic (A3C) with Generalized Advantage Estimation (GAE)
> We implemented A3C [[1]](https://arxiv.org/abs/1602.01783) along with GAE [[2]](https://arxiv.org/abs/1506.02438) to train our agent. We choose to use A3C mainly due to the observation that the data is relatively expensive to collect from the malmo environment, thus we want to be able to collect experiences over multiple machines. 
* Implementation Details
> We use the simbolic view as input to the network, with a little bit of preprocessing: we cut off the borders of the symbolic view since they provide no information for the agent, then one symbolic view is fed into the network as a 4-channel image: one for all the objects in the environment, one to indicate the position of the pig, then one for each of the two agents. ....................................


## Novel Points
* Training over multiple machines with docker
> pass
* Curriculum Learning
> After training a vanilla A3C agent on the malmo challenge, we found that the agent is tended to go to the exit directly and get the small reward instead of trying to find a way to coorperate with its opponent and catch the pig. We suspect that this is due to the probability for the latter scenario to happen is relatively small, so the agent would have too few such examples to learn sufficiently how to get the big reward for catching the pig. This observation inspired us to utilize currilum learning in the training of our agent, in the belief that it would be much more probable for the agent to learn the optimal behavior if it starts from relatively simple tasks (by simple we mean that the probability for the agent to catch the pig is higher) then gradually transits to more difficult tasks [[3]](http://dl.acm.org/citation.cfm?id=1553380). We thus modify our training procedure so that in the beginning of the training, the opponent would always be executing A* actions, so the probability that our agent and the opponent could catch the pig together would be higher. The probability that the opponent is a ``RandomAgent`` instead of an ``AStarAgent`` is linearly annealed from ``0`` to ``0.75`` (which is the original setting in the malmo challenge). This gives us a big performance gain even though we have only trained this paradigm for (one day???) iterations; we believe this training procedure can result in much better performance if we have trained it for more iterations.
* Periodical Finetuning
> One thing we notice is that the malmo environment is relatively unstable when running on remote clusters: we start 8 malmo environments (2 malmo agents each) then many of them stop to communicate ......... in about 10 minutes. Thus our solution is to restart a new training instance every 10 minutes, by finetuning from the model saved from the last training instance.

> A3C multiprocess in Docker


*******
# Evaluation Results
Our final evaluation result is here: ``https://github.com/jingweiz/malmo-challenge/blob/master/ai_challenge/pig_chase/final_result.json``
```
{"500k": {"var": 18.835718608113272, "count": 519, "mean": 0.50481695568400775}, "100k": {"var": 3.4812551696770671, "count": 2158, "mean": -0.85171455050973122}, "experimentname": "My experiment 2"}
```

## How to reproduce the result
### Environment configuration
We train our model with 3 CPU servers. Firstly we build a docker swarm with docker swarm mode and let all of these three CPU servers join this docker swarm. We build a docker 
In manager node
```
docker swarm init --advertise-addr <MANAGER node ip>
```
In worker node
```
docker swarm join \
  --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
  <manager node ip>:<port>
```
We creat a docker swarm overlay network for the communications between the docker nodes.
```
docker network create --driver overlay --subnet 10.0.9.0/24 --opt encrypted malmo_net
```



*******
# Video
We generate a video showing the performance of our agent, which can be found [here](https://youtu.be/_lWTLc9VH1E).


*******
# References:
* [1] [Asynchronous Methods for Deep Reinforcement Learning] (https://arxiv.org/abs/1602.01783)
* [2] [High-Dimensional Continuous Control Using Generalized Advantage Estimation] (https://arxiv.org/abs/1506.02438)
* [3] [Curriculum learning] (http://dl.acm.org/citation.cfm?id=1553380)
