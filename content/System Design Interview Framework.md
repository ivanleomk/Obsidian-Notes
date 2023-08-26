# Interviewing

System Design interviews will typically be ~ 45 minutes to an hour. It's important to note that **you are being evaluated on how well you work collaboratively and your ability to resolve ambiguity**.

Here is a rough breakdown of the components
- Understanding Requirements ( 3-10 minutes )
- Creating a High Level Design ( 10-15 minutes )
- Deep Dive into specific components ( 10-25 minutes )
- Conclusion ( 3 - 5 minutes )

## Understanding the Requirements

There are no marks for replying fast - in fact you'll be penalised for giving a quick response with a clear understanding of the question. Therefore, take time to understand the specific requirements of the question/

Here are some useful questions
- How is the application required to scale?
- What are the main features that we need to implement?
- What is the rough load that the application will need to handle?
- Are there any specific latency requirements?

## Creating a High Level Design

It's important here to **get the interviewer's buy in to your solution**. At this point, you should have 

1. Obtained a clear understanding of the features that you need to support
2. Identified any potential bottlenecks that your system might be forced to support.
3. Done some basic estimation as to whether your scale constraints can be matched by your proposed design
4. Have a clear implementation of a architecture that supports the features that you need to support

## Deep Dive

At this stage, depending on your experience level and the interviewer, you might need to focus on different things. So, look for hints to see what your interviewer might like.

## Wrap Up

Some good things to do here are

- Give a short recap on what you've done
- Talk about specific constraints that your system has - Eg. it cannot support Y feature at the moment and might have issues down the line with Z
- Talk about how to scale up - how can we improve its ability to do more
- Error Logging and Metrics to track - How can we verify that the system is indeed working as intended.



