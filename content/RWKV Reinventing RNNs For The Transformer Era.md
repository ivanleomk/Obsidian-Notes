- tags : #llm  #rwkv #rnn




> Paper : https://arxiv.org/abs/2305.13048 ( Reinventing RNNs for the Transformer Era )
> 
> Articles:
> https://fullstackdeeplearning.com/blog/posts/rwkv-explainer/
> [Picocreator's Writeup on RWKV](https://github.com/PicoCreator/2nd-brain/blob/main/A%20-%20TechTalkCTO/P%20-%20RWKV%20Musings/The%20RWKV%20architecture%20-%20scaling%20RNN%20to%20transformer%20scale/RWKV%20architecture%20-%20Scaling%20an%20RNN%20to%20transformer%20scale.md)
> 
> Code examples here are taken from https://johanwind.github.io/2023/03/23/rwkv_details.html which is very very well written. 
> 
> I also put together a [small colab notebook](https://colab.research.google.com/drive/1ZRHKtJsYY8DSh09Mm2WH7iHayX7NUrMX?usp=sharing) while putting together these notes - you can fork it and experiment with the architecture.



# Introduction

RWKVs are a new architecture that utilise the [[RNN]] architecture with some modifications. It's been shown to perform on par with other transformer models that have an equivalent parameter size.

The main problem with the [[Transformer]] architecture is that the input will scale exponentially with the number of tokens we pass in. This is because for every token that comes in, we need to compute a unique portion of the attention matrix for it.

![[CleanShot 2023-09-21 at 11.38.22.png]]

RWKVs aim to solve this by using a linear operation to approximate attention instead. With the new architecture proposed, they can replace the original [[LSTM]] mechanisms in previous RNN-based architecture and the QKV mechanism ( shown above ) that [[Transformer]] architectures rely on to capture semantic and positional meaning.

This results in ~1-1.5x cheaper training costs and ~10x cheaper inference costs, especially as the number of tokens passed into the model scale. 

![[CleanShot 2023-09-21 at 11.39.46.png | 400]]

## High Level Understanding

RWKVs operate using two main mechanisms

1. A World state : This is used to store information and previously computed tokens
2. Temporal Mechanism: We utilise a time decay on information that was previously computed and store in the world state in order to replace the [[Positional Embeddings]] in the [[Transformer]] model architecture

## Weakness

As a result of the use of a world state, the RWKV model has a major weakness - it might end up discarding content which is relevant BUT not explicitly requested in the prompt. 

As a result, prompt engineering is significantly more important when it comes to RWKVs. A suggested workaround is to first list the question/demand before subsequently including information about the context.

## Architecture

## Sigmoid

Intuitively, the sigmoid function is used as a activation function in both the time-mixing and acts as a forget gate in our [[#Time-Mixing]] and [[#Channel Mixing]] blocks. Since all values are effectively coerced from 0 to 1, A value closer to 0 means the information should be forgotten, while a value closer to 1 means the information should be retained.

This plays a crucial role in determining the relevance of information from prior steps, allowing the network to selectively retain or discard information based on the current input and previous hidden state
## State

In essence, we can think of the state as being comprised of the following components

`[layerState,layerState,.... ]`

Inside each layerState, we have a total of 4 subarrays

`state = np.zeros((N_LAYER, 4, N_EMBD), dtype=np.float32)`

inside the state, we allocate space for each element as

- 0 : prevX computation
- 1 : prevNum Computation
- 2 : prevDen Computation
- 3 : prevChannelMixing result

This helps our model to be able to get some concept of time. Note that (3) is mostly used to replace the short-term memory ( since it can only look back a single step )

### Time-Mixing

Time mixing approximates attention and can be likened to the LSTM component of a traditional RNN architecture. This is because it has access to two things

- Previous World State
- Learned weights to determine how to combine previous computations and new computations.

> Intuitively as time $t$ increases, then the vector is dependent on a long history and a summation of an increasing number of terms. This creates a memory which can keep track of information provided in an earlier context.

Let's look at a function to compute time-mixing

```python
# Time mix layer with the various params
def time_mixing(
		# Incoming state of the current token
		x, 
		# Previous token shift state 
		last_x, 
		# Previous state, split across 2 values to prevent overflows
		last_num, last_den, 
		# Various weights, all trainable
		decay, bonus, mix_k, mix_v, mix_r, Wk, Wv, Wr, Wout
	):
	
	# Given the incoming state, and the previous token shift state
	# compute R,K,V values. The `x * mix + last * (1-mix)` pattern
	# helps the model have trained weights to decide which factors 
	# it wants to use for the respective process
    k = Wk @ ( x * mix_k + last_x * (1 - mix_k) )
    v = Wv @ ( x * mix_v + last_x * (1 - mix_v) )

	# Since R is used for the final gating of output (similar to attention score)
	# you can view this as a lite form of Q @ K
    r = Wr @ ( x * mix_r + last_x * (1 - mix_r) )

	# Here we effectively do magic(last_state + k) * v
	#
	# But in essence, its the "summation with decay" of all the past expotents
	# divided by another set of expotents for numeric stability
	# (aka anti exploding gradients). 
	# 
	# Bonus is used to boost the signal for the WKV value
    wkv = (last_num + exp(bonus + k) * v) / \
          (last_den + exp(bonus + k))
    
	# We compute the cumulative sum, for the next round, where it 
	# is stored and forwarded as a state, with a gradual
	# decay value on the previous value. 
	# 
	# `exp(-exp(decay))` is essentialy a math hack to ensure decay
	# is between 0-1, and will gradually fade out past value if desired
	# 
	# `exp(k) / exp(k) * v` is the summation accumulation of the current state
	# to be used in the next time mix
    num = exp(-exp(decay)) * last_num + exp(k) * v
    den = exp(-exp(decay)) * last_den + exp(k)

	# sigmoid then acts looseley as both a part of Q in QKV, 
	# and as a forget gate in LSTM (together with decay)
	# for the WKV values
    rwkv = sigmoid(r) * wkv

	# And finally that gets normalized into an output, and next state
    return Wout @ rwkv, (x,num,den)
```


This function implements the following equations

![[CleanShot 2023-09-21 at 21.16.33.png | 400]]
In our function, We have the parameters of 
- `last_x,last_num, last_den` : These are all stored in a huge `(N_Layers, 4,N_embeddings)` array and simply store the previous computations
	- `last_x`: $x_{t-1}$
	- `last_num`: $wkv_{t-1}$ numerator
	- `last_den`: $wkv_{t-1}$ denominator
- `decay, bonus, mix_k, mix_v, mix_r, Wk, Wv, Wr, Wout`
	- `decay` : $w$ parameter can be treated as $e^{-\text{decay}}$
	- `bonus` : This is equivalent to the `u` parameter above
	- `mix_k`,`mix_r` and `mix_v` are equal to $\mu_k,\mu_r,\mu_t$
	- `Wk,Wv,Wr and wout` are equivalent to $W_k,W_v,W_r$ and $W_o$ above
- Note here that $\sigma$ simply represents the sigmoid of the function

In the code above, it might be a little bit difficult to visualise the dimensions so 

- (1024,1024) : $W_r, W_k, W_v$ 
- (1024, ): $\mu_r,x_t,x_{t-1},\mu_k,\mu_v$ , decay, bonus

This means that essentially $r_t,k_t$ and $v_t$ are all going to be of the dimension $(1024,)$ and the final output of this function will be of $(1024,)$. 

> This is an interesting property since it means that the information on all previous tokens seen is ...essentially stored in a single scalar value in a 1024 dimensional array. That's pretty impressive since transformers require the use of a `n_seq`$\times$ `n_embd` array to store and compute the information from all preceeding tokens


![[CleanShot 2023-09-21 at 21.33.40.png]]

We can visualise the entire time-channel that we've implemented as a single giant block.

### Inductive Proof

One of the parts that confused me were these specific line of code

```python
wkv = (last_num + exp(bonus + k) * v) /      \
          (last_den + exp(bonus + k))
rwkv = sigmoid(r) * wkv

num = exp(-exp(decay)) * last_num + exp(k) * v
den = exp(-exp(decay)) * last_den + exp(k)

return Wout @ rwkv, (x,num,den)
```

Specifically, they're trying to implement

$$
wkv_t=\frac{\sum_{i=1}^{t-1}e^{-(t-1-i)w+k_i}v_{i} + e^{\mu+k_t}v_t}{\sum_{i=1}^{t-1}e^{-(t-1-i)w+k_i} + e^{\mu+k_t}}
$$


Let me try to break it down in my own way

If we look at the summation term, $\sum_{i=1}^{t-1}e^{-(t-1-i)w+k_i}v_{i}$ , you can see that the final value of the term is going to be $e^{-(t-1-(t-1))w+k_i}$ , which can be reduced to $e^{k_{t-i}}$. Well, what about the second last value of the summation then? Well, it's simply going to be $e^{-(t-1-(t-2))w+k_{t-2}}$ which can be reduces to $e^{-w+k_{t-2}}$. This means that we can therefore rewrite 

$$
\sum_{i=1}^{t-1}e^{-(t-1-i)w+k_i}v_{i} = \sum_{i=1}^{t-2}e^{-(t-1-i)w+k_i}v_{i} + e^{k_{t-1}}
$$

If we do substitute $\alpha_{t-1}=\sum_{i=1}^{t-1}e^{-(t-1-i)w+k_i}v_{i}$ , then we can also perform the substitution of $\alpha_{t-1} = \sum_{i=1}^{t-1}e^{-(t-1-i)w+k_i}v_{i} = \alpha_{t-2}+e^{k_{t-1}}v_{t-1}$. We can perform a similar substitution to get $\beta_{t} =\beta_{t-1}+ e^{k_{t-1}}$. 

This corresponds to the following two lines of 

```python
num = exp(-exp(decay)) * last_num + exp(k) * v
den = exp(-exp(decay)) * last_den + exp(k)
```

In this case , num and den would correspond to $\alpha_{t-1}$ and $\beta_{t-1}$ for a time step of $t$. Therefore, this means that our equation to get $wkv_t$ is simply then going to be 

$$
wkv_{t} = \frac{\alpha_{t-1} + e^{\text{bonus} + k_{t}}v_t }{\beta_{t-1} + e^{\text{bonus} + k_{t}}}
$$

which corresponds to our implementation of 

```python
wkv = (last_num + exp(bonus + k) * v) /      \
          (last_den + exp(bonus + k))
    rwkv = sigmoid(r) * wkv
```




## Channel Mixing

The channel mixing is the short term component of the system - it only has access to the previous value ( and to some degree )
![[CleanShot 2023-09-21 at 21.58.03.png]]

It's significantly easier to understand. **Note that all these parameters are learned parameters with the exception of the $x_{t-1}$ values**.

Our channel mixing function is implemented as 

```python
def channel_mixing(x, last_x, mix_k, mix_r, Wk, Wr, Wv):
    k = Wk @ ( x * mix_k + last_x * (1 - mix_k) )
    r = Wr @ ( x * mix_r + last_x * (1 - mix_r) )
    vk = Wv @ np.maximum(k, 0)**2
    return sigmoid(r) * vk, x
```

- It's interesting here to note that 
	- $w_k$ has a dimensionality of $[4096,1024]$
	- $w_v$ has a dimensionality of $[1024,4096]$

which means that our values are essentially scaled from a dimensionality of 1024 up to a dimensionality of 4096. It's therefore, most useful to think of this like a feed forward network in a traditional transformer where embedding layers are scaled up 4x too in dimensionality.

## Forgetfulness

Models exhibit interesting behaviour whereby the first 200 dimensions seem to exhibit perfect recall where as the last remaining dimensions in this case seem to simply discard previous information.

![[CleanShot 2023-09-21 at 22.32.42.png]]
