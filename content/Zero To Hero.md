> [Zero to Hero](https://karpathy.ai/zero-to-hero.html) is a course by [Andrej Karpathy](karpathy.io) which focuses on helping people to understand the math behind deep learning networks. It's a pretty fantastic course and I highly recommend it.

# Micrograd

## Introduction

> Micrograd is a simplified implementation of scalar back propagation written in numpy and python. 
> 
> Notebook with code on how to implement this is [here](https://github.com/ivanleomk/Zero-To-Hero-notes/blob/master/lesson1/Lesson%201.ipynb)

A good way to think about Back Propagation is the flow of gradients through a network of operations. 

![[Sample Flow.png]]

We can resolve the gradients for some equations symbolically - i.e. we can solve them by solving for an abstract form that they match and then derive the gradients.

## Operations Supported

Here is a list of some simple equations that our micrograd implementation supports

| Original Equation | $\frac{da}{da}$ | $\frac{db}{da}$   | $\frac{dc}{da}$ |
| ----------------- | --------------- | ----------------- | --------------- |
| a = b + c         | 1               | 1                 | 1               |
| a = b * c         | 1               | c                 | b               |
| a = tan(b)        | 1               | 1 - $a^2$         | nil             |
| $a = e^{x}$       | 1               | e^{x}             | nil             |
| $a = \frac{b}{c}$ | 1               | $\frac{1}{c}$     | b               |
| $a = b^{n}$       | 1               | $c\times b^{c-1}$ | nil ( n is treated as a constant here)                |

## Chain Rule

But we don't just want a simple $a=b+c$ , we probably want to compute a more complex equation such as 

```
a = 3
b = 3
c = a * b

d = 12
e = 3
f = 12 + 3

z = c * f
```

and from there calculate the value of 

$$
\frac{dz}{da}
$$
We can't do this using our original formula but we can do so using the chain rule, which states that

$$
\frac{dz}{da} = \frac{dz}{dc} * \frac{dc}{da}
$$
And so if we chain many of these operations together, we eventually get the gradient we want - that is the change in the loss with respect to some value that's much further down the original chain of equations. 

The important thing to note here is that we can therefore compute the derivative for every single one of these variables by **recursively applying the chain rule and finding the product of it with the local derivative that you are calculating**.
## Applications

Ultimately when we have a whole cluster of these values together, we can form neural nets, which can be represented as matrix multiplication (see code for implementation)

# Bigram Models

> A Bigram is simply a collection of two characters (Eg. __ab__,__de__ ) and a Bigram model in this course is simply a model which takes in an input of two characters and then outputs a prediction of the next most likely token.





