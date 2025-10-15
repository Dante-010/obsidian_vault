(Ver [[A tener en cuenta]] para un resumen)

### 1er correo
Nos dimos cuenta de una "incongruencia" o posible error en la forma de entrenar y testear el agente, por lo que enviamos el siguiente correo a los autores originales:

---

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

--- 

Obtuvimos las siguientes respuestas:

Andrea Bernini
Hi Dante,  
Thanks for your questions. It's been two years since I last saw the code or discussed the topic, so I don't remember every detail, but your analysis helped me reconstruct our approach.  
  
Most likely, the configuration you found is the one we used for the evaluation phase, not for training.  
  

- Training: As I recall, the training was done with a random community selection (method 1). The goal was to teach the agent a general strategy that would work for any type of community.
- Testing: For the tests, however, we deliberately used method 2 with the relative sizes [0.2, 0.5, 0.8]. This allowed us to controllably analyze performance on small, medium, and large communities and to see how the method scaled, especially on the larger datasets.

So, the code in the repository is likely set up to replicate the paper's tests. Your experiment, by testing in random mode, is the correct way to evaluate the model's actual generalization.  
  
I hope this clears things up. Good luck with your thesis!  
Best regards.

---

Gabriele Tolomei
Hello everyone,  
  
Many thanks to Esteban for following up on this, and to Dante for getting started on the topic. Thanks also to Andrea, who, despite no longer being involved in the project, still managed to provide an answer to the question raised.  
  
I've copied Matteo, Edoardo, and Dario on this message, as they've been working on new approaches (including the gradient-based one mentioned by Esteban). I believe they can support Dante with any technical issues that may arise.  
  
I hope this helps.  
  
Cheers,  
G.

---

Matteo Silvestri
Hello Dante,

Thank you very much for reaching out and for your thoughtful questions. We really appreciate your interest in our work.

- **On the concern about overfitting:** As Andrea mentioned, the training is done on the whole graph, so the agent does not overfit to the specific communities. To give you an idea, in a recent work, we resampled the target node every 5 episodes and the target community every 50 episodes.
    
- **On evaluation strategy:** We think that evaluating only on random nodes may introduce some experimental bias, since hiding a node in a small community is quite different from hiding one in a huge community. For this reason, we evaluate separately on small, medium, and large communities. Notably, randomness is still present in the evaluation, as we sample up to 100 nodes within the specified communities.
    
- **On performance drops:** We have observed that the problem includes what we call “hard nodes” and “soft nodes.” Some target nodes are essentially impossible to hide (e.g., the two hubs in the karate club graph). If we sample randomly across the graph at each iteration, the performance may vary significantly depending on the proportion of hard versus soft nodes. We find this to be a very interesting aspect worth further exploration.
    

Thanks again for your questions, and please feel free to share any further thoughts. We’d be happy to discuss more!  
  

Best regards,  
M.