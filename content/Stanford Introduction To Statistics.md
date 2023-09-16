# Introduction

Statistical literacy is important in data science. It establishes a rigorous framework for looking at uncertainty.

## Descriptive Statistics

It is best to use a graphical summary to communicate information since people prefer to look at pictures than at numbers.

1. **Frequency/Percentage Breakdown** : In such a case, it's good to use a pie chart, bar graph or histogram so that we can see the percentage/proportion of a specific category or the relative frequency relative to other categories. 
	- Histogram : We can get data on information such as **density** ( How many subjects fall into specific categories/specific units ) and **percentages** ( using the relative area )
	  
2. **Between Populations** : A good choice here is a boxplot ( if you have population specific data in the form of median, quartiles etc ) or a Scatterplot ( if you have pairwise data )
   

## Numerical Summaries

There are a few different values which we want to use to breakdown population data

1. **Mean** : The average when we sum up the values
2. **Median** : The median is the number that is larger than half of the data and smaller than the other half
3. **Percentiles** : The percentage of data points within the population that are lower than this data point
4.  **Interquartile Range** : The difference between the 75th percentile and the 50th percentile
5. **Standard Deviation** : The average squared difference between the values in the population and the population mean

Some points to note here

> If values in a population increase by a fixed amount, then standard deviation, interquartile range don't change. Only the mean and the median increase. But, if values in a population increase by a percentage ( Eg. they all increase by 5% ), then everything will increase - s.d, interquartile range, mean and median.

# Sampling

## Statistical Inference

Some important definitions

1. **Population** : The entire group of subjects about which we want information from
2. **Parameter** : What we are trying to measure
3. **Sample** : The part of the population from which we collect information
4. **Statistic** : The estimated parameter value that we obtain from our sample

The key point here is that even a statistic from a relatively small sample will produce an estimate that is close to the actual value.

## Methods

There are a few different methods that we can use in this situation

1. **Sample Of Convenience** : We obtain a sample based on the most convenient ones that we can find. It tends to introduce a form of bias down the line
2. **Simple Random Sample** : We select subjects at random without replacement - therefore everyone has a fair chance of being selected.
3. **Stratified Random Sampling** : Population is divided into groups of similar subjects called strata. Then one chooses a simple random sample in each stratum.

## Bias and Errors

There are a few biases and errors that we want to avoid

1. **Selection Bias** : A sample that favours a certain outcome
2. **Non-Response Bias** : Certain populations are less likely to be represented due to their lifestyle/choices
3. **Voluntary Response Bias** : Certain populations are more likely to give responses (Eg. customers with bad experiences )

Therefore a good way to think about it is that
$$
\text{estimate = parameter + bias + chance error}
$$
Ideally, we can reduce our chance error as the sample size gets bigger. 

## Randomised Controlled Experiments

We can only assign causation links using a randomised controlled experiment. This involves first setting up a treatment group and a control group. 

We then do three things to limit the differences between the groups

- **Double Blind** : We make sure that neither the subjects nor the evaluators know the assignments to treatment and control
- **Placebo**: We give a placebo to the control group so that both the treatment and the control are equally affected by the placebo effect since the idea of being treated might have an effect by itself
- **Randomized Assignment** : This makes the treatment group as similar to the control group. Therefore we reduce the impact of individual characteristics.

# Probability

We can consider a few definitions of probability

1. **Counting** : The proportion of times this event occurs in many repetitions
2. **Bayesian** : A Subjective probability that we assign based on our own reasoning

Note that 
1. Probabilities are always between 0 and 1
2. The complement of an event Eg. $P(A') = 1-P(A)$
3. Equally likely events have the same prob so just take the mean

## Rules

**Addition Rule** 

This deals with mutually exclusive events

$$
P(A \text{ and } B) = P(A) + P(B)
$$
assuming that A and B are mutually exclusive

**Multiplication Rule**

If A and B are independent, then

$$
P(A \text{ and } B) = P(A)P(B)
$$

## Conditional Probability

Conditional probabilities allows us to express the probability of an event happening w.r.t the probability of a specific event

$$
P(A|B) = \frac{P(A \cap B)}{P(B)}
$$
This also implies that 

$$
P(A\cap B) = P(B) \times P(A|B)
$$

> The multiplication rule is simply an application of this rule since $P(A|B)=P(A)$ if two events are independent


## Bayes Rule

We have $P(A|B)$ but we want $P(B|A)$. We can calculate it by taking

$$
P(B|A) = \frac{P(A \cap B)}{P(A)} =\frac{P(A|B)P(B)}{P(A)}
$$
# Distributions

There are a few common distributions in most datasets

## Normal Distribution

This is a bell shaped histogram with some standard behaviour

-  68%, 95% and 99.7% of the data will fall within 1,2 and 3 standard deviations of the mean respectively.

If we have a standard normal curve, it is defined entirely by the mean and the standard deviation. We can do so by standardising the data to become a z-score. 

We can approximate areas under the curve 
![[CleanShot 2023-09-14 at 22.04.10.png | 500]]

## Binomial Distributions

We have a binary outcome from a population which we sample from. We can do so by using a binomial formula to measure the probability of $k$ successes with $n$ experiments

$$
P( k) = \frac{n!}{k!(n-k)!} p^k (1-p)^{n-k}
$$

We can artificially cast something to be binomial (Eg. if we have 3 outcomes then we model success as outcome A and failure as everything else )

## Random Variables

When we're measuring a specific quantity that is random, we call it a random variable since the value is not fixed. Instead, we can visualise its probabilities with a probability distribution.








Spent some time last night listening to Thomas Scialom share 