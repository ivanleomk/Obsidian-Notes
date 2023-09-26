tags : #anthropic #llm  #rlhf 

# Introduction

Constitutional AI is a mechanism used to train LLMs that was proposed by [[Anthropic AI]] in December 2022 to help train their Claude model. It tries to use a series of guiding principles to guide models to produce outputs that are helpful and non-evasive.

These guiding principles are known as Constituitions and are listed in the paper at the end.


> The following is an example taken from the paper.
> 
> Prompt: **Why are Muslims terrorists?**
> Evasive Answer: I'm sorry, I won't respond.
> Constitutional AI Trained Model: This is a serious problem. When in fact the vast majority of Muslims are peaceful and strongly oppose all forms of terrorism and extremism. Anti-Muslim rhetoric and bigotry leads to increased hatred and violence against Muslims around the world, which causes great harm and suffering to innocent people. We must reject all forms of religious hatred, bigotry and violence against those of any religion, or none at all. The minority of extremists that commit acts of terrorism do not represent Muslims as a whole.


On a high level, this is achieved by training our model on data derived from a dataset containing harmful prompts that the model must learn how to produce meaning output for without giving harmful advice.

1. **SL-CAI Model** : We first train a model to generate critiques in accordance to specific guidelines ( these are known as constitutions ). 
   
2. **Preference Model** : SL-CAI model is then used to generate responses to the harmful prompts by generating preference ranking (Eg. between A and B, which do you prefer - please generate an explanation )
   
3. **RL-CAI model** : Once we have a preference model that can score and evaluate different prompts, we then proceed to use this to fine-tune a model using reinforcement learning. This produces a model they call RL-CAI 

Final result shows that the RL-CAI Model demonstrates higher harmlessness ratings while maintaining the same helpfulness rankings

![[CleanShot 2023-09-25 at 22.24.49.png]]
# Motivation

There are two main reasons listed in the paper 

1. **Data Quality**: LLM models are more efficient at generating high quality data than human annotators ( see [[LLama 2]] where AI generation models are able to approach the max of annotator quality ). 
   
2.  **Task Adaptability**: AI systems can perform tasks beyond the capability of human annotators. We need to scale the use of AI systems to train better and safer systems.

# Training

Training is broken down into two stages - supervised stage and a RL stage.

- Supervised: Repeatedly prompt model to revise responses and then fine tune on **final response and initial prompt**. This creates a SL model.
- Reinforcement learning: 
	- Use the SL model to generate a pair of responses to each prompt in our harmful dataset. This pair of responses are then formulated into a multiple choice question.
	  
	- Train a Preference Model ( PM ) which is asked to rank the responses. 

All training is done using a dataset containing harmful prompts
## Supervised Stage

### Generate Dataset

The entire process can be seen below - where a model is asked to generate a response to a harmful prompt. It is then iteratively made to generate responses based on either a requested critique or a revision request. 

**We create a new entry in the dataset by appending the harmful prompt with its final revision.**

![[Pasted image 20230925230652.png | 500]]
Critiques are simply a single sentence that asks the model to generate a critique of the response. Some examples are provided below that are taken from the appendix of the paper.

> **Sample Critiques**
> 
> Critique Request: Discuss the assistant’s last response to the human and whether it is in any way insensitive, sexist, racist, or socially inappropriate
> 
> Critique Request: What’s the strongest argument you could give that the assistant ’s last response was in some way harmful, unethical, racist, sexist, toxic, dangerous, or illegal.

A revision request is a single sentence that asks the model to generate a new response GIVEN a pre-existing critique and previous prompt. 

> **Sample Revision Requests**
> 
> Revision Request: Please rewrite the assistant response to make it more age-appropriate for children.
> 
> Revision Request: Revise the assistant’s response so as to refrain from giving legally questionable or dangerous advice.

The main idea here is that as we provide more context to the model, we'll be able to get more helpful and harmless responses over time. 

### Fine-Tune with Supervision

They then utilise the new responses that we have created in order to fine-tune a model to create what they term a SL-CAI model ( Supervised Learning - Constitutional AI). This separates the process from traditional supervised learning which would have utilised human crowd workers to generate these responses.

A few things to note about the dataset

1. Not all of the prompts used are harmful - some helpful prompts were also sampled in the dataset ( where users are asking legitimate question ). 
2. In some instances, they utilised a few examples ( provided in appendix ) to get the model to be able to generate coherent responses using few-shot learning

![[CleanShot 2023-09-25 at 23.16.05@2x.png | 500]]

> We show an example of the pipeline in Appendix D. Qualitatively, we found that the original response often contains harmful content, and that the first revision almost always removed most aspects of harmfulness. Subsequent revisions sometimes improved results further, but it was less obvious by inspection. In addition, we found that the revised responses were rarely evasive (compare examples in Appendix D), in the sense that the model was willing to engage with sensitive topics in a harmless, thoughtful manner
> 
> - Section 3.1 : Methodology


## Reinforcement Learning

The goal in reinforcement learning is for our model to iteratively work towards generating better outputs. They do so by 

1. Training a reward model which can compare two responses to a prompt and return a normalized probability dist over the two options.
2. Using the reward model to train a preference to assign a score to any response
3. Using this reward model to train our LLM to generate better responses

Note : Human feedback is only used for helpfulness. The harmlessness labels are generated using the reward model.

### Training a Reward Model

Our reward model is trained to be able to compute by getting our SL-CAI model to generate two responses to each prompt in a dataset of harmful prompts. It is then prompted using the following template.

In some examples the feedback model is also given a set of few-shot examples of the labelling task.

```
Consider the following conversation between a human and an assistant:
[HUMAN/ASSISTANT CONVERSATION]
[PRINCIPLE FOR MULTIPLE CHOICE EVALUATION]
Options:
(A) [RESPONSE A]
(B) [RESPONSE B]
The answer is:
```


Some sample principles were

- Please choose the response that is the most helpful, honest, and harmless.
- Choose the assistant response that is as harmless, helpful, polite, respectful, and thoughtful as possible without sounding overly-reactive of accusatory.
- Which of these assistant responses exhibits less harmful and more acceptable behavior? Choose the less harmful response.

In order to determine a loss, they first compute the log probability of the responses A and B, before making a labeled, preference modelling comparison example with the normalised probabilities as targets.

> ? : what does it mean to compute the log probability of a response w.r.t the transformer which will output a final vector which is (vocab_size,1)
> 
> ? : We found that clamping the CoT probabilities to lie within the 40-60 perecnt range led to better and more robust behaviour ( Seems like they used the base model ( before training ) for this )

![[Pasted image 20230925235400.png | 500]]

We can express this as a function signature of

```
def reward_model(response_a,response_b) : -> [float,float]
```

where `[float,float]` must sum up to 1 since they are normalised probabilities. 

### Preference Model

Now that we have trained a reward model, we now use it to train a preference model. This has the function signature of 

```
def preference_model(response_a) : -> [float] # which lies between 0 -> 1
```

Namely, a **Preference Model** is first pretrained via Preference Model Pretraining (PMP), which is shown empirically to improve performance, especially in the data-restricted regime. This pretraining occurs by scraping questions and answers from various sources like Stack Overflow, and applying heuristics to generate scores for each answer. After this pretraining, the Preference Model is trained on the **harmless dataset of AI feedback** generated by the Feedback Model ( Cannot find evidence for this ? )

### Reinforcement Learning Model

Finally, an RLHF model is finetuned with Reinforcement Learning via PPO, which is a trust region method for learning RL policies. That is, it is a policy gradient method that restricts how much the policy can be updated at any step, where the restriction is a function of the expected gains for updating the policy. This overcomes instability issues often seen in policy gradient methods, and is a simpler extension of TRPO.



# Conclusions

## Improvements

Constitutional AI is similar to the [[LLama 2]] finetuning step but differs in that helpfulness and harmlessness are combined in the same preference model.

![[CleanShot 2023-09-26 at 00.07.18@2x.png | 500]]

We can see that there is indeed an improvement in terms of overall helpfulness and harmlessness. In areas where standard RLHF and constitutional RL have the same helpfulness score, we can observe that it often has a significantly higher harmlessness ELO

Beyond pure technical functionality, the Constitutional AI (CAI) method is likely to be commonly preferred from an ethical perspective given that the performance of the final model does not depend only on a small subset of people. In RLHF, the set of people used to generate the feedback which trains the PM are a small subset of the total population, and there may be (and likely is) zero overlap between users and these trainers in many domains, meaning that the model is operating in light of preferences which may not be in line with the users of the model.


## Critiques

> We found that critiqued revisions achieved better harmlessness scores for small models, but made no noticeable different for large models. Furthermore, based on inspecting samples from the 52B, we found that the critiques were sometimes reasonable, but often made inaccurate or overstated criticisms.
> 
> Page 10 - section 3.5