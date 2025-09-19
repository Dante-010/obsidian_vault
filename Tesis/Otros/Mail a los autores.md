To: Andrea Bernini (andreabe99@gmail.com), Fabrizio Silvestri (fsilvestri@diag.uniroma1.it), Gabriele Tolomei (tolomei@di.uniroma1.it).

Hello, my name is Dante Culaciati, I am a Computer Science student at the University of Buenos Aires, currently doing my undergraduate thesis with a former colleague of yours, Prof. and Dr. Esteban Feuerstein, based on your paper "Evading Community Detection via Counterfactual Neighborhood Search".

The paper is very clear and well explained, and the code is very well structured and easy to understand and modify.
One thing that intrigued me when looking through the code were these lines: (and the behaviour related to it).

```
PREFERRED_COMMUNITY_SIZE = [0.2, 0.5, 0.8]
-----------
# Method to change the target community
    # - 1: choose a random community
    # - 2: choose the community with the length closest to the a set percentage of the maximum
    #       length of the communities.
    # - 3: choose a community based on the distribution of the number of
    #       nodes in the communities
    COMMUNITY_CHANGE_METHOD = 2
```

From what I could gather, it seems that in training you are only focusing on communities that fit a certain condition (`self.preferred_community_size = HyperParams.PREFERRED_COMMUNITY_SIZE.value[0]` on `graph_env.py`).

When you run `change_target_community()`, you use `self.fixed_community()`, which returns a community whose length is closest to a set percentage of the maximum length of all communities.

When testing, you are using all preferred sizes (in this case, which was the code as it was present in the repository, the values are 0.2, 0.5 and 0.8) and averaging over them.
(Just for reference, each time `reset_experiment()` is run, a new community is chosen, so the chosen method matters a lot).

I was curious to see what would happen if I ran the same tests, but only changing the community change method to random (the model is the same), and to my surprise, the F1 score was much less for "small" datasets, except for fb-75, where it was unusually high.
Furthermore, when trying to run various "random" tests, the results were all over the place, in the sense that no run was able to replicate the behaviour of the other ones consistenly (as is the case with using method 2).

Perhaps, some of this is due to poor implementations or reasoning on my part, or simply that the parameters in the code and the ones used in the paper simply do not coincide.

My question is, was using method 2 a deliberate and important choice? 
Could it be considered "overfitting" since you are only training the model to handle communities with a certain relative size compared to the others? 
Should testing be done with the same method as training, or is always using the random method the correct approach (to test that the agent can handle all situations)?

Thanks in advance for your time and attention, I hope to hear from you soon.