To: Andrea Bernini (andreabe99@gmail.com), Fabrizio Silvestri (fsilvestri@diag.uniroma1.it), Gabriele Tolomei (tolomei@di.uniroma1.it).

Hello, my name is Dante Culaciati, I am a Computer Science student at the University of Buenos Aires, currently doing my undergraduate thesis with a former colleague of yours, Prof. and Dr. Esteban Feuerstein, and Prof. and Dr. Gabriel Tolosa, based on your paper "Evading Community Detection via Counterfactual Neighborhood Search".

The paper is very clear and well explained, and the code is very well structured and easy to understand and modify.
After reading through the code and doing some simple experiments, I came up with some questions in order to fully understand the problem. Before asking, I will tell you my current understanding of the code.

The first thing that caught my eye were these lines:

```
PREFERRED_COMMUNITY_SIZE = [0.2, 0.5, 0.8]

# Method to change the target community
    # - 1: choose a random community
    # - 2: choose the community with the length closest to the a set percentage of the maximum length of the communities.
    # - 3: choose a community based on the distribution of the number of
    #       nodes in the communities
    
COMMUNITY_CHANGE_METHOD = 2
```

From what I could gather, it seems that in training you are only focusing on communities that fit a certain condition (`self.preferred_community_size = HyperParams.PREFERRED_COMMUNITY_SIZE.value[0]` on `graph_env.py`).

Whenever you run `change_target_community()`, you use `self.fixed_community()` (method 2), which returns a community whose length is closest to a set percentage of the community with the maximum length. I considered it important that this method is deterministic.

When testing, you are using all preferred sizes (in this case, which was the code as it was present in the original repository, the values are 0.2, 0.5 and 0.8) and averaging over them.
(just for clarity, each time `reset_experiment()` is run, a new community is chosen, so the chosen method can have a great impact on testing).

I was curious to see what would happen if I ran the same original tests (with the original model), but only altering the community changing method to random, and to my surprise the F1 score was considerably lower for "small" datasets (kar, words, vote), and varied randomly for the larger datasets (pow, fb-75).

Perhaps, some of this is my fault, due to poor understanding, reasoning or implementation, or simply that the hyperparameters in the code and the ones used in the paper do not coincide.

My questions are:
Was using method 2 a deliberate and important choice? 
Could it be considered "overfitting" since you are only training the model with communities with a certain relative size compared to the others? 
Should testing be done with the same community changing method as training, or is always using the random method the correct approach (to test that the agent can handle all situations)?

Thank you in advance for your time and attention, and sorry for the long email.
Best regards.