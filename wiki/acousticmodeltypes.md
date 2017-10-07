---
layout: page 
title: Acoustic Model Types
---

CMUSphinx supports different types of the acoustic models: continuous, 
semi-continuous and phonetically tied (PTM). 

The difference between PTM, semi-continuous and continuous models is the 
following. We use mixture of gaussians to compute the score of each frame, the 
difference is how do we build such mixture. In continuous model every senone 
has it’s own set of gaussians thus the total number of gaussians in the model 
is about 150 thousand. That’s too much to compute the mixture efficiently. In 
semi-continuous model we have just 700 gaussians, way less than in continuous 
and we only use them with different mixtures to score the frame. Due to the 
smaller number of gaussians semi-continuous models are fast, but because of 
more hardcoded structure they are also a bit less accurate. 

PTM models is a gold middle here. It uses about 5000 gaussians thus providing 
better accuracy than semi-continuous, but it is still significantly faster than 
continuous thus it can be used in mobile applications. Accuracy of PTM model is 
almost the same as accuracy of continuous model.

In modern packages of pocketsphinx and sphinx4 PTM models are used by default.

To understand the model type you need to look in feat.params file inside the 
model. PTM models have `-model ptm` there. Continuous model have `-feat 
1s_c_d_dd`. You can also infer the type from the model name, semi-continuous 
models usually have "semi" in the name.

