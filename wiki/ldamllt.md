---
layout: page
title: Training an acoustic model with LDA and MLLT feature transforms
---

Attention!  This feature is for models trained on a single-stream features (for 
example, continuous models) For semi-continuous models this feature is not 
supported.

On of the SphinxTrain features is the ability to train feature-space 
transformations for acoustic models.  There are a couple of benefits to using 
these.  First of all, it can dramatically reduce the word error rate (up to 25% 
relative in some of our tests).  Second, it also makes the decoder slightly 
faster since it reduces the dimensionality of the features, and also reduces 
the size of the acoustic model.

Unfortunately the training process becomes a bit more involved when using this 
feature.  The reason is that it's necessary to do some parts of training 
several times over.  Specifically, you have to train a basic model in order to 
train each feature transformation, then retrain the model with the 
transformation applied to the input features.  This has to be done for each 
feature transformation (currently there are two of them as they have been found 
to have additive effects).

These feature transformations are "discriminative" in the sense that they try 
to improve the separability of acoustic classes in the feature space.  This 
means that it's necessary to define the set of acoustic classes on which they 
are trained.  There are two obvious choices which both seem to work well - the 
simplest one and the quickest to train is simply the context-independent 
phonemes.  The more involved one is to use the context-dependent tied triphones 
(senones).  In both cases, the SphinxTrain scripts try to automate the whole 
process for you.

### Required software components

First, you **need** to have the necessary [Python 
modules](InstallingPythonStuff) installed in order to do LDA and MLLT.  You 
should (obviously) also have Python 2.3 or newer.  To make sure that you have 
the necessary modules installed, make sure that you can run Python and load the 
`numpy` and `scipy.optimize` modules:

	
	dhuggins@lima:~$ python
	Python 2.5.1 (r251:54863, Oct  5 2007, 13:36:32)
	[GCC 4.1.3 20070929 (prerelease) (Ubuntu 4.1.2-16ubuntu2)] on linux2
	Type "help", "copyright", "credits" or "license" for more information.
	>>> import numpy
	>>> import scipy
	>>> import scipy.optimize
	>>>


### Configuration changes

Once you're sure that Python is working, you need to make sure that you are 
using the most recent version of SphinxTrain.

Finally, you need to turn on LDA and MLLT in your `sphinx_train.cfg` file.  To 
do this make sure it contains two lines reading:

	
	$CFG_LDA_MLLT = 'yes';
	$CFG_LDA_DIMENSION = 32;


You can adjust `$CFG_LDA_DIMENSION` if you like, though 32 seems to be a 
nearly-optimal value for many data sets.

### Decoding with the MLLT model

Models trained with LDA/MLLT have a file with feature transform called 
`feature_transform` in the model folder. This file will be used automatically 
by the sphinx4 and pocketsphinx decoders. There is no need to perform any 
specific adjustment except you should probably experiment with the optimal 
language weight since language weight depends on the transform.


### Expected results using MLLT

Using MLLT, you can hope for roughly a 25% improvement.  For example, if you 
had 70% accuracy:

	
	 . 70 + (100 - 70) * 0.25 = 77.5% 


You would now get a 7.5% improvement to 77.5%.

### Cepstral Window Features

It's possible to use automatically trained linear transform to bypass some last 
feature extraction steps like DCT matrix multiplication and extraction of 
derivatives, which are linear transforms too. You can just join frame features 
together and hope that LDA/MLLT matrix will extract important components from 
them. In theory, it could give some improvement in accuracy. To do that, you 
need the following:


*  Set feature type "1s_3c" for window 3 or "1s_4c" for window 4 in training 
configuration file

*  Enable MLLT and set vector size to reasonable value (around 40).

*  Train

*  Decode

