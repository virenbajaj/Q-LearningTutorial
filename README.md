
#  Q Learning!
______________________

- Q Learning is a type of unsupervised reinforcement learning technique.

- Reinforcement Learning is a class of machine learning algorithms in which the the program determines the ideal behavior within a specific context based on reward feedback for a transition from a given state to another.

In this tutorial we learn how to find an optimal path from one node of a graph to another using Q-learning. But before we get into that, here is a little motivation and theoretical introduction:

## Humans vs. Deep Q-networks 

Q-learning algorithms were used in Google's DeepMind to develop the so called "deep Q-networks".Using only pixel data and the game score as inputs, this algorithm learned 49 Atari 2600 games and proved to be as succesful as expert humans. A paper about the same was published in Nature in February 2015.(linked here : http://www.nature.com/nature/journal/v518/n7540/full/nature14236.html - look at the videos linked on the website!)


## Markov Decision Process
More specifically, Q-learning is used to find an optimal action selection policy for a given finite Markov Decision Process(MDP).We shall now see what is a Markov Decision Process. 

A Markov Decision Process model contains: 

1. Set of world states S 
2. Set of possible actions A
3. A real valued reward function R(s,a) where $s \in S$ $a \in A $
4. A description T of each action's effects in each state

This decision process satisfies the **Markov Property** which means that the effects of the action taken in a state depend on that state only, and not any prior state or action in the process. An MDP is a _"discrete time stochastic control process"_ which means that we specify a probability distribution to the all the states reachable from a given state by performing a specific action. More precisely, T: S X A -> Prob(S), which represents the distribution P(s' | s, a) 

### Policy $\pi$
A policy $\pi$ is a mapping from S to A, which means that it is choice of action given a state. The algorithms objective is to make this policy such that cumulative rewards of the process (all successive steps) is maximized. Q-learning has been proven to find such an optimal policy eventually. 

Following a policy is what you think it means: given a state s select and execute an action $\pi(s)$, and repeat for the new state reached by executing $\pi(s)$.

__Which policy is the best policy?__

We can evalute a policy by finding the expected value of the total reward. To compare different policies with infinite expected values we use an objective function (reward function, which is essentially a negative loss function) that maps an infinte sequence of rewards to a single real value.We can create this function by discounting to favour "earlier" rewards. More formally, for a given discount rate __$0<\gamma<1$__, we discount a reward which is n actions away by $\gamma^n$.

Notice that an infinite sum $$\sum_{i=0}^{\infty}r*\gamma^i = r*\frac{1}{1-\gamma}$$ from the infinite sum of a geometric series. Such an approach is most widely used and, as you can see, analytically tractable.

### Value funtion
Before we proceed our analysis, we define a value funtion (objective function) $V_{\pi}:S->\Re$ that gives us the expected objective value by following the policy $\pi$ for each state $s \in S$.

Thus this value function maps every policy to a real "utility" value, with which we can order the policies, atleast partially. The functions are partially ordered because some policies can result in the same real value. This model ensures that ATLEAST one optimal policy exists: the one with the maximum expected reward. Also notice that every optimal policy will have the same value function, say V*. 

We can mathematically define the value function as:
$$V_{\pi}(s) = R(s,\pi(s)) + \sum_{s'\in S} T(s,\pi(s),s') . \gamma . V_{\pi}(s')$$
where $\pi(s)$ represents an action 'a' to be performed for a given state s dictated by the policy $\pi$, s' are all the states reachable from s with probability described by $T(s,\pi(s),s') = P(s' | s, a) $. And so the optimal policy from the state s is just :
$$V^*(s) = max[\ \forall a \in A\ (\ R(s,a) + \sum_{s'\in S} T(s,a,s') . \gamma . V^*_{\pi}(s')\ )\ ]$$

Naturally, there is only one V* for every s in S.
These equations are known as Bellman Equations.

### The Algorithm
The Q-learning algorithm is just an iterative value updation form of the value function($Q:S X A -> \Re$):
$$Q'(s,a) = Q(s,a) + \alpha * (\ R(s,a) + \gamma . max Q(s',a) - Q(s,a)\ )$$
where $\alpha(s,a)$ is the learning rate and is between 0,1, which could be the same for every pair of (s,a).Here, we must define an "episode" of the algorithm which ends when s' represents the final state.Just like before, we wish to find the optimal action for each state, which is done by maximizing the cumulative reward from that state. Note that $Q(s_f,a)$ where $s_f$ is the final state is never updated and corresponds to fixed reward r. 

The algorithm will be simplified and further explained in the example prose below.



# Sample Problem
In this tutorial I will illustrate how this algorithm, i.e, finding an optimal action selection policy for a finite MDP, works to find the best path from a one vertex in a graph to another. To solidy the problem more, consider a building with 5 rooms, with some doors that connect the rooms, while some of the them open up to the outside of the building. Our goal in this problem is to find a way out of the building, given we are placed in a given room. Specifically, consider this set up:

![alt text](building_pic.gif)

Clearly, this can be represented as a graph with each room as a vertex and a door as a bi-directional edge. And so our problem is to find a graph path to the goal state, which is represented by vertex 5 in the following graph repsentation of the building:

![alt text](graph1.gif)

Now, notice that world of states, S, is the set of vertices, which represent the rooms.
The set of possible actions, A, is represented by the outgoing edges from a vertex, which correspond to travelling from one room to another through the door connecting the two. 
And so, next, we must assign a Reward function R(s,a). We set R(s,a) = 100 for all those actions which take s to the goal state = 5, and 0 otherwise. These rewards become the edge weights, as represented by the following graph:
![alt text](graph2.gif)

# Reward/Environment Matrix
Notice that a graph of this sort can be represented by a 5x5 reward matrix in which the row index, i, represents the current state, column index, j, the final state by going through the door connecting the room i to the room j, and the value at (i,j) the reward. Since we knew all the states a priori, we can initialize this 5x5 matrix, but even we didnt know all the states, we can just program the code to add another column when it discovers a new state.This type of code is capable of handling a large amount of unkown states and probably is more data driven. We represent the impossible actions(non-existent edges), like going directly from room 2 to 4, by 'None' values in the matrix.



```python
import random
```

    



```python
# This is the reward/environment matrix, representing the graph
# of states to states transitions, with the associated
# reward for the given transition (action). A row represents
# the current state, and the columns in that row represent the
# possible states you can reach and their associated reward.
r = [[None, None, None, None, 0, None],
     [None, None, None, 0, None, 100],
     [None, None, None, 0, None, None],
     [None, 0, 0, None, 0, None],
     [0, None, None, 0, None, 100],
     [None, 0, None, None, 0, 100]]

r2 = [] 
#We can also initialize the goal state here:
goal_state = 5

```

# Q-matrix
Next, we define the Q-matrix, that will hold the reward value at postion (i,j) for the corresponding action as calculated by our algorithm. Every iteration of the Q-learning algorithm is referred to as an 'episode' which begins with selecting a random state and ends the moment our 'graph seach' or traversal of the DFS reaches the goal state. In simpler terms, our algorithm is is as follows:

## Simplified Q-Learning Algorithm

1. Set the gamma parameter, and environment rewards in matrix R.

2. Initialize matrix Q to zero.

3. For each episode:
    
    Select a random initial state.

    Do While the goal state hasn't been reached:

        A. Select one among all possible actions for the current state.
        B. Using this possible action, consider going to the next state.
        C. Get maximum Q value for this next state based on all possible actions.
        D. Compute: Q(state, action) = R(state, action) + Gamma * Max[Q(next state, all actions)]
        E. Set the next state as the current state.
    End Do

   End For

And thus this Q-matrix will represent the policy $\pi$ that we must follow to get to the goal state from any given state. After sufficient experience, i.e enough episodes, our algorithm will result in a Q-matrix with converged and stabalized values, such that any more learning would not change the Q-matrix. This proof is not covered in this tutorial. 


```python
#INITIALIZE THE Q-MATRIX
q = None
def init_q():
    # Assumes r is a square matrix
    l = len(r)
    # Matrix to be initialized
    global q
    q = []

    # Create a matrix the same size as r
    # initialized with all 0's
    for row in xrange(l):
        q_inner = []
        for col in xrange(l):
            append_value = 0.0
            q_inner.append(append_value)
        rown = q_inner
        q.append(rown)
    result = q
    return result

```

We build our agorithm from a top-down approach, but write the code down-up. 

# possible_actions
Here we define a function **possible_actions** that takes in **current state** and returns a **list of (reachable state,reward for taking that path)**


```python
def possible_actions(current_state):
    # Grab the possible actions for the current state
    row = r[current_state]
    # Includes the possible actions, with their rewards.
    current_actions = []

    next_state = 0
    # Remove the transitions that are not possible,
    # and add in the next_state, reward pair.
    for reward in row:
        condition = not(reward == None)
        if (condition):
            append_value = (next_state, reward)
            current_actions.append(append_value)
        next_state += 1
    result = current_actions
    return result

```

# choose_action
Next, define **choose_action** that selects the next action from all possible actions for a given state. which is either optimal (if e = 0) or suboptimal (if e>0) because e 
The function takes in **current_state**,**current_actions**(returned from possible_actions) and the parameters **e** and **g**. 
1. __e__ or epsilon is used in the action selection policies in such a way that it represents a percentage of times we wish to choose an action randomly, i.e.,we want the algorithm to 'explore' more states and not follow a deterministic 'optimal' path. In other words, e represents the probability of selecting a random action at a given state, versus selecting the most optimal action. Hence if e is closer to 1, a more random action selection policy is used, and the agent is more of an 'explorer' than 'exploiter'. If closer to 0,it is the opposite.
2. __g__, as discussed in the theory, is the gamma constant. If g is closer to 1, the algorithm is more likely to optimize for future rewards versus immediate rewards. If closer to 0, then it will look more heavily to gain immediate rewards(only looking at the current reward if 0)and if vice versa if closer to 1.

The function returns a tuple of the **next_state, reward**.


```python
def choose_action(current_state, current_actions, exploring,e,g):
    # Find the best action to take from the list.
    best_next_state = None
    best_reward = None
    best_update = None

    # Loop through to find the best action for the given state.
    for (next_state, reward) in current_actions:
        # Calculate the Q-value for this state, and action pair.
        q_update = r[current_state][next_state] + g * max(q[next_state])

        # See if it is the most optimal so far
        isNone = (best_update == None)
        betterUpdate = (q_update > best_update)
        condition = isNone or betterUpdate
        if (condition):
            best_update = q_update
            best_reward = reward
            best_next_state = next_state

    # Now we will choose either the optimal action, or
    # a random action depending on the probability
    # e. e->1 means high probability of exploring and vice versa for e->0.
    
    optimal = (best_next_state, best_reward)
    # To do that, create a list of booleans that
    # determines whether or not to take the optimal,
    # or a random action. If a True is chosen,
    # take the optimal, else take a random action.
   
    choice_list = []
    number_random = int(e * 100)
    choice_list = []
    for i in xrange(number_random):
        append_value = False 
        choice_list.append(append_value)
        
    for i in xrange(100-number_random):
        append_value = True
        choice_list.append(append_value)

    # Select a choice randomly, with the probability e that a random
    # selection will be made.
    
    index = random.randint(0, 99)
    
    choice = choice_list[index]
    if (choice or not exploring):
        result = optimal
        return result 
    else:
        random_choice = random.randint(0, len(current_actions)-1)
        result = current_actions[random_choice]
        return result 
```

# run_episodes
**run_episodes** is the top-level function for the algorithm that makes the final Q-matrix. It takes in the parameters **episodes**,**e**,and **g**. 
Recall, **episodes** are the number of runs of our algorithm desired to train the Q-Matrix through
unsupervised learning. An episode begins at a random state and ends when the current_state = goal_state.
It returns an optimized **Q-matrix**, which represents an action policy $\pi$. We can simply follow the highest reward from a given state to find the best path to our goal. 


```python
def run_episodes(episodes,e,g):
    
    # Initialize q to all 0's
    init_q()
    assert(q != None)
    
    
    # Useful variables
    l = len(r)

    # Run through the episodes
    for episode in xrange(episodes):

        # Select a random initial state
        current_state = random.randint(0, l-1)
    
        # Reached goal state
        exploring = True
        
        while (exploring):
            
            # Check if this is the last state
            if (current_state == goal_state):
                exploring = False

            # Grab the possible transition pairs that
            # contain (state, reward).
            current_actions = possible_actions(current_state)
            
            # This function selects an action from the current_actions
            # list and takes it based upon the selection policy. If exploring
            # is false, then automatically take the best action because we are in
            # the goal_state.
            next_state, reward = choose_action(current_state, current_actions, exploring,e,g)
            
            # Modify the q matrix based on the selected action.
            q_update = reward + g * max(q[next_state])
            q[current_state][next_state] = q_update

            # Take the action, and set the current_state to the new state
            current_state = next_state
    return q

def run_session(): 
    g = 0.8
    e = 0.9
    
    print "g = ", g
    print "e = ", e
    print "goal_state = ", goal_state 
    episodes_list = [1,2,5,100,1000,10000]
    
    # Run the training.
    
    for episodes in episodes_list:
        q = run_episodes(episodes,e,g)    
        print "Q-matrix after ", episodes, " episodes"
        print str(q)
    

```


```python
run_session()
```

    g =  0.8
    e =  0.9
    goal_state =  5
    Q-matrix after  1  episodes
    [[0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 100.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 100.0]]
    Q-matrix after  2  episodes
    [[0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 180.0], [0.0, 0.0, 0.0, 64.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 80.0, 0.0], [0.0, 0.0, 0.0, 64.0, 0.0, 100.0], [0.0, 0.0, 0.0, 0.0, 0.0, 180.0]]
    Q-matrix after  5  episodes
    [[0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 64.0, 0.0, 336.1600000000001], [0.0, 0.0, 0.0, 64.0, 0.0, 0.0], [0.0, 51.2, 0.0, 0.0, 144.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 244.0], [0.0, 0.0, 0.0, 0.0, 0.0, 336.1600000000001]]
    Q-matrix after  100  episodes
    [[0.0, 0.0, 0.0, 0.0, 399.99999987268535, 0.0], [0.0, 0.0, 0.0, 319.9999998010708, 0.0, 499.99999987268535], [0.0, 0.0, 0.0, 319.99999968917314, 0.0, 0.0], [0.0, 399.9999998408567, 255.99999939291612, 0.0, 399.99999905143136, 0.0], [319.9999998408567, 0.0, 0.0, 319.9999998010708, 0.0, 499.9999998981483], [0.0, 0.0, 0.0, 0.0, 0.0, 499.9999998981483]]
    Q-matrix after  1000  episodes
    [[0.0, 0.0, 0.0, 0.0, 400.0, 0.0], [0.0, 0.0, 0.0, 320.0, 0.0, 500.0], [0.0, 0.0, 0.0, 320.0, 0.0, 0.0], [0.0, 400.0, 256.0, 0.0, 400.0, 0.0], [320.0, 0.0, 0.0, 320.0, 0.0, 500.0], [0.0, 0.0, 0.0, 0.0, 0.0, 500.0]]
    Q-matrix after  10000  episodes
    [[0.0, 0.0, 0.0, 0.0, 400.0, 0.0], [0.0, 0.0, 0.0, 320.0, 0.0, 500.0], [0.0, 0.0, 0.0, 320.0, 0.0, 0.0], [0.0, 400.0, 256.0, 0.0, 400.0, 0.0], [320.0, 0.0, 0.0, 320.0, 0.0, 500.0], [0.0, 0.0, 0.0, 0.0, 0.0, 500.0]]


# Analysis of Results
As you can see the values of the Q-Matrix converge as the number of episodes increase. 
The Q-matrix optimized for g = 0.8 is:
![alt text](finMatrix.gif)
We can normalize the values of the Q-Matrix with the highest reward(500) to get:
![alt text](normFinMatrix.gif)
Finally, to illustrate the shortest path from room 2 to 5 using the Q-Matrix values:
![alt text](finGraph.gif)

This represents the path (2,3,1,5) or (2,3,4,5), which are equivalent and correct (look at the pictures of the room at the top to verify).


```python
# Another run with g = 0 to show that this will  only consider the immediate reward so no Q-learning will happen
def run_session2(): 
    g = 0.0
    e = 0.9
    
    print "g = ", g
    print "e = ", e
    print "goal_state = ", goal_state 
    episodes_list = [1,2,5,100,1000,10000]
    
    # Run the training.
    
    for episodes in episodes_list:
        q = run_episodes(episodes,e,g)    
        print "Q-matrix after ", episodes, " episodes"
        print str(q)
```


```python
run_session2()
```

    g =  0.0
    e =  0.9
    goal_state =  5
    Q-matrix after  1  episodes
    [[0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 100.0], [0.0, 0.0, 0.0, 0.0, 0.0, 100.0]]
    Q-matrix after  2  episodes
    [[0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 100.0], [0.0, 0.0, 0.0, 0.0, 0.0, 100.0]]
    Q-matrix after  5  episodes
    [[0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 100.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 100.0], [0.0, 0.0, 0.0, 0.0, 0.0, 100.0]]
    Q-matrix after  100  episodes
    [[0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 100.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 100.0], [0.0, 0.0, 0.0, 0.0, 0.0, 100.0]]
    Q-matrix after  1000  episodes
    [[0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 100.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 100.0], [0.0, 0.0, 0.0, 0.0, 0.0, 100.0]]
    Q-matrix after  10000  episodes
    [[0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 100.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], [0.0, 0.0, 0.0, 0.0, 0.0, 100.0], [0.0, 0.0, 0.0, 0.0, 0.0, 100.0]]



```python

```

# Improvements

The matrix representation of a graph is very memory inefficient because we have 0's for all the impossible actions. We could instead map only the exisent actions to their Q-value using a dictionary.

# Further Reading
Detailed paper about general Q-learning techniques:
http://www.gatsby.ucl.ac.uk/~dayan/papers/cjch.pdf

Paper about choosing the correct 'e' parameter using Bayesian Methods in order to balance 'exploration' vs. 'exploitation':
http://www.cs.huji.ac.il/~nir/Papers/DFR1.pdf



# References
https://en.wikipedia.org/wiki/Q-learning

http://reinforcementlearning.ai-depot.com/ 

http://www.cs.rice.edu/~vardi/dag01/givan1.pdf

http://mnemstudio.org/path-finding-q-learning-tutorial.htm
