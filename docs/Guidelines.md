---
layout: default
title: Guidelines
---

# Deep Learning Competition club Guidelines
### A.k.A. How should we work on a project.
Created by: Bence Dudás

# 1. Metrics
The foundation of every competition is that you have to optimize your model to some specific metric. Therefore our main goal is to **gather**, **understand** and **implement** the metrics used in the competition scoring.
## How to understand them?
The minimal understanding of these metrics/scores we want to have:
- What is their minimum and maximum range? (if it's something like mean squared error, 0-$\infty$, if it's accuracy, 0-1).
- What is it that they are sensitive to? Does over or under-prediction influence them differently?
- Is there any physical limitation to them? For example doing some regression based in images and their resolution enables only to achieve a certain value.
- Are there multiple versions of this metric, that yield different result for different circumstances?
## Implementation
### Gather
Try to stick to the most simple implementation or predefined library function. I know it's very easy to just ask a chatbot to do it for you, but do understand which version of that function you are implementing.

Some examples we had problems with:
- Integrated Depth Dose (IDD) - Measures the integrated deposited dose from radiation. Problem: This has angle dependency, so we either had to rotate all images to the dose curves are aligned or implement a version that is angle independent.
- Maximum Mean Discrepancy (MMD) - Measures the distance between two distributions. Problem: There are versions of this calculations that are processing the data in multiple bandwith scales. We had to make sure which version is used in the competition scoring.

Additionally, we always have to look out if any of the scores can be turned into loss functions or we can find some loss function that optimizes for the same result.

## Code
Finally we have created our functions, that's all right? Right?
Well, not exactly. Having code that does not crash is one side of the story, we still need to make sure that the results are aligned with reality! Some basic unit test to include:
- Identity score: If the input and the target are the same it should give the best possible score.
- Small perturbation: If we make a slight noise to the input - in the way that the score should not be too sensitive to that - we should obtain a somewhat worse score. To know what the model is sensitive to, we have to first understand it.
- Worst score (if possible): If there is a completly worst case scenario, check if the model give the worst score to that.
- Terrible score (if possible): If we know some extremely bad scenarios, which is random guessing, we should check how does the model score on that.

# 2. Data exploration
We always learn that data exploration is super important, and I also try to explain why, but here I emphasize it again. Don't just do data analysis for the sake of making nice plots! Try to figure out relations between the data and the output, something like:
- Is the final prediction strongly dependent on these features? If yes, how can we include it in the model?
- How is our data distributed? Is something over/underrepresented?
- Are there extreme outliers?
- Visualize the data if it's possible.
- Are there physical limitations? 
You can't know your data well enough.

# 3. Working
This is the part which will be the most different from project to project. Some basic guidelines I would like to lay down:
- Build a strong evaluation pipeline, that checks all the metrics for the competition and the summarize our model performance. In this way we will always know if we are in the right track or not.
- Maximize the number of submissions. If the total number of submissions is not limited, try to hit the daily limit all the time. In this way we can see better if we are improving or not.
- In order to improve on something always ask why is this metric is stucked? If we have nice MSE but the generated images are blurry, and we have a metric which is sensitive to that, think on what could enforce sharpness.
- Do not code for streaming services. We are building pipeline for scientific competitions/research. Don't build 200 try-except blocks! It's hard to check through on them and not necessary, when we are using a fixed environment. 

## Technicality
It's okay to use Chatbots, but please think in blocks, not in full projects and please understand the code you are using. Same as with the losses, know if it's any sensitivity or limitation for the certain block you are using. Always have someone from the competition club to check your code! 
I usually like to use pytorch lightning for the training, therefore the usual scripts structure I use is the following:

- metrics.py - This is where we store the scores of the competition and other loss objects we want to use. I like to build an overall lossClass, which calculates the specific losses we want to use and returns them in a dictionary, for easier logging. For example, {"mse_loss": 0.123, "mmd_loss": 0.456, "bce_loss": 0.789}.
- callbacks.py - Since we are using lightning, we can define callbacks that are called at the end of an epoch or before/after every batch. If we want to do some custom logging, the best way is to define a callback for that, instead of creating a super long training loop or model class.
- models.py - This contains every torch.nn.Module inherited class, the neural network blocks we want to use. The specific convBlocks, UnetBlocks, transformers and so on. 
- dataloaders.py - Define here every data processing step along with the dataloaders.
- training_backbones.py - This is where I like to define the recipe for our training. Basically go with the standard lightning modules, and write the configure_optimizers, training_step, validation_step and everything else that we need from the model.

In this way if we have a training.py, there we only have to specify which callbacks I want to add to the model, which training pipeline I want to use, and so on.

## Final thoughts
If the competition has a forum we need to always check that, see if there is any problem with something or is there any new information. Did someone find out that the scores can be hacked and so on. I wish you all a lot of fun and success in the competitions!  
[HOME](../README.md)
