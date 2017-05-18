---
layout: page 
---
## The Incomplete Guide to Sphinx-3 Performance Tuning

The [Continuous Density Acoustic 
Models](http://www.speech.cs.cmu.edu/sphinx/models/) used in SphinxThree have 
large numbers of free parameters, and so can be effectively trained from very 
large amounts of speech data, which allows them to produce very accurate 
recognition results.

However, the downside of this is that evaluating all of those parameters at 
runtime can be horribly slow.  In fact, so slow that no amount of machine 
optimization (short of offloading things onto a high-end GPU) can make it fast 
enough.  For example, the Communicator model on that page has 4132 tied states, 
each of which is modeled by 64 multivariate Gaussian densities of 
dimensionality 39.  To evaluate all these Gaussians for each 10 millisecond 
frame of acoustic features would require nearly four million arithmetic 
operations, half of which are multiplications.  To put this in perspective, 
even assuming zero memory latency and single-cycle execution for every 
operation, we would still be spending 2ms for each 10ms frame on Gaussian 
computation alone on a 2GHz processor.

Therefore, it is necessary to use approximate computation techniques to compute 
only the states and component densities which are most likely for any given 
frame.  This presumes that we have some way to determine ahead of time which 
ones are going to be likely.  Fortunately, this is broadly true.  We rely on 
the following observations and facts in order to make these shortcuts:

 1.  The set of likely states and densities changes relatively slowly from 
frame to frame.
 2.  If state N of a given HMM is not likely in this frame, then all states >N 
are not going to be likely in future frames.
 3.  If some simplified model of a state or class of states is not likely, then 
those states are probably not going to be likely either.

## How To Tune the Decoder

There is a general series of steps you can use to obtain a satisfactory 
tradeoff between speed and accuracy.  While we can't tell you what the exact 
parameter values you'll need to use for your particular task are, we'll give 
you a general idea of what you need to tweak and which way to tweak it.

The first thing you need to do is have a setup for evaluation.  For this you 
will need:

 1.  A set of test data
 2.  Transcriptions for the test data
 3.  A script to calculate word error rate, like this one: 
[attachment:word_align.pl](attachment/word_align.pl).

This preparation is not discussed here.  If someone would like to create a Wiki 
page on this that would be great!

## Establishing a Baseline

It's important to first figure out how the decoder performs with no tuning.  
There are a small set of command-line arguments which we are going to modify in 
the course of tuning the decoder.  We'll start with them at these "untuned" 
values:

	
	-beam        1e-100
	-wbeam       1e-80
	-maxcdsenpf  100000
	-maxhmmpf    100000
	-maxwpf      40
	-subvqbeam   1e-3
	-ci_pbeam    1e-80
	-ds          1


## Interpreting the SUMMARY line

After running the decoder, you should find a line near the end of the log file 
which gives a summary of how much time was spent in decoding and where it was 
spent.  It will look something like this:

	
	INFO: stat.c(204): SUMMARY:  15220 fr;  2584 cdsen/fr, 132 cisen/fr, 
82712 cdgau/fr, 4224 cigau/fr, 
	2.22 xCPU 3.40 xClk [Ovhrd 0.10 xCPU  0 xClk];  8452 hmm/fr, 108 wd/fr, 
1.30 xCPU 1.99 xClk;  
	tot: 3.54 xCPU, 5.44 xClk 6

This line can be interpreted as:


*  There were 15220 frames of speech (152.2 seconds)

*  The average speed of the decoder was 3.54 times real-time (CPU).  That is, 
for each second of speech, 3.54 seconds of CPU time were needed for recognition.

*  Of these 3.54 seconds, 2.22 were spent in Gaussian density computation.

*  Of these 3.54 seconds, 1.30 were spent in Viterbi beam search.

*  The average number of context-dependent states evaluated per frame was 2584.

*  The average number of context-dependent Gaussians evaluated per frame was 
82712.

*  The average number of HMMs active per frame was 8452.

*  The average number of words active per frame was 108.

The numbers of states, Gaussians, HMMs, and words per frame are directly 
related to the speed of the decoder.  If we reduce these, then we will make the 
decoder faster.  Different tuning parameters have different effects on these.

The ''-beam'' argument affects the number of HMMs active per frame, which 
indirectly affects all of the other parameters.  If fewer HMMs are active, then 
there will be fewer words which "survive" to their ends, fewer states will be 
computed, and hence fewer densities will be computed.  This parameter is the 
coarsest form of tuning, and in general, making it too narrow (i.e. too large 
numerically) will increase the error rate more than tuning more specific 
parameters, as discussed below.

In general, you will find that the beam can be narrowed a bit without reducing 
accuracy.  You should narrow it only this far and no further.  Likewise with 
''-wbeam'' which has similarly broad effects.  We usually set ''-wbeam'' a bit 
narrower than ''-beam'' and leave it alone...  (actually ''-wbeam'' has 
interactions with the language model weight which we won't discuss here)

## Reducing Gaussian computation

Typically, Gaussian computation is the biggest chunk of time used by the 
decoder.  Therefore we want to start by speeding things up at this level.  The 
goal is to reduce the ''cdgau/fr'' number, that is, the number of Gaussians 
evaluated per frame.

There are a few ways to do this but the most effective one implemented in 
SphinxThree is called SubVectorQuantization.  Basically this involves creating 
a highly simplified version of the acoustic model and evaluating it first to 
determine which Gaussians are worth evaluating in their full form.  The 
acoustic models on [the Sphinx Open Source Models 
Page](http://www.speech.cs.cmu.edu/sphinx/models/) include pre-built subvector 
quantized files which are called ''subvq'' in the acoustic model directory.  To 
use this, you have to pass it to the decoder with the ''-subvq'' argument.  
This should immediately give you a speedup without much effect on accuracy.

To speed things up further you can tighten the ''-subvqbeam'' parameter.  The 
default value is 0.001, so you can try 0.01 or 0.005.

## Reducing GMM computation

If you've already made the decoder fast enough for your taste, you can stop 
now.  Otherwise, the next thing you can do is to reduce the number of Gaussian 
mixture models (i.e. the number of tied states) evaluated at each frame.  This 
is the ''cdsen/fr'' number in the ''SUMMARY'' line.

There are two complimentary ways to do this.  The more intelligent of the two 
is called ContextIndependentGmmSelection.  This means that the 
context-independent phones' states are evaluated first and  for those which 
don't score well, all context-dependent phones related to them are eliminated.  
This can be done by tightening the ''-ci_pbeam'' argument.  The default value 
for it is so wide as to be completely ineffective, so you can try first 
increasing it to 1e-10.

The next way to reduce GMM computation is called AbsolutePruning and can be 
achieved with the ''-maxcdsenpf'' argument.  Instead of computing all GMMs 
which are considered "likely to succeed", this enforces a hard limit on how 
many can be computed.  The best way to set this parameter is to look at the 
''cdsen/fr'' number in the ''SUMMARY'' line, and set it to some number slightly 
lower than that.  So for the example line above, we could put ''-maxcdsenpf 
2000'' in the command-line.

## Reducing HMM computation and Search

You can also apply AbsolutePruning to entire HMMs and words.  This is a 
somewhat more precise version of ''-beam''.  Again, look at the ''hmm/fr'' and 
''wd/fr'' numbers in the ''SUMMARY'' line and set ''-maxhmmpf'' and ''-maxwpf'' 
to something smaller than them.  In the case of ''-maxwpf'' the hard limit is 
not particularly hard at all, so you can set it to something a lot smaller than 
the number of words per frame.

## The Big Guns: Frame Downsampling

If you really can't get it fast enough, it's time to try the "nuclear option", 
known as FrameDownsampling.  This does something you might think would not 
work, but actually does, which is computing Gaussian scores at every Nth frame. 
 This can be particularly effective for careful speech, where the durations of 
speech sounds tend to be long and hence the acoustic characteristics of the 
speech change slowly.

Frame downsampling can be enabled with the ''-ds'' argument.  Try ''-ds 2'' at 
first.  It's probably not a good idea to go higher than 3.

## Acknowledgements

The general structure of this document is inspired by Arthur Chan's paper:

 * Arthur Chan, Jahanzeb Sherwani, Ravishankar Mosur, Alexander I. Rudnicky. 
[Four-Layer Categorization Scheme of Fast GMM Computation Techniques in Large 
Vocabulary Continuous Speech Recognition 
Systems](http://www.cs.cmu.edu/~jsherwan/pubs/icslp2004.pdf).  Proceedings of 
ICSLP 2004.

