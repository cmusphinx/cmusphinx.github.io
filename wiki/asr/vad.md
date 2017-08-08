---
layout: page 
---
The current state-of-the art is pretty ad-hoc, a lot of algorithms are applied 
together in order to get a good performance and most of them require carefully 
hand-crafted parameters in order to operate reliably in noise.

The main idea of most VADs is that we track and suppress noise first, then we 
apply some classifier to decide if the frame is speech or not and then we might 
want to apply some hangover to select significant speech regions and ignore 
small random variations in decision. 

Few core ideas used are:

VAD operates in spectral instead of time domain, noise tracking is performed in 
mel bands.

Statistical-based noise removal method is applied in order to separate signal 
from stationary noise:

[ A Statistical Model-Based Voice Activity 
Detection](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.34.550 )

[ Noise Spectrum Estimation in Adverse Environments: Improved Minima ]( 
http///citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.3.7758 )

If there are multiple microphones, microphone-array signal separation can be 
used to find out signal sources and space and separate noise from speech using 
spatial properties.

[VAD with microphone array]( 
http///www.mirlab.org/conference_papers/International_Conference/ICASSP%202012/p
dfs/0004901.pdf )

GMM and other machine-trained classifiers are used for several features like 
pitch, signal levels and so on. Recent research includes recurrent neural 
networks for example:

[ Neural Networks For Voice Activity 
Detection](http://static.googleusercontent.com/media/research.google.com/en//pub
s/archive/41186.pdf )

Most of the VAD methods deal with stationary or almost-stationary noise and 
there is a great variety of tweaks you can apply here. For example you can 
replace simple Wiener noise suppression filter with IMCRA one and get a new 
noise suppression algorithm and, consequently, new VAD algorithm. Same way, you 
can replace the classifier or add a new features to it and it will give you a 
new algorithm. And every new parameter in the algorithms will need tuning, in 
particular tuning of the computational expenses.

This ends into fusion of the various systems, like the one available here:

<https://github.com/mvansegbroeck/vad>

One of the modern fast VADs available in public is VAD from WebRTC codec, it 
incorporates almost all the features existing:

<https://code.google.com/p/webrtc/source/browse/trunk/#trunk%2Fwebrtc%2Fcommon_audio%2Fvad>

and it's pretty reliable.

The major issue with VAD is that speech signal is considered alone and the 
methods for arbitrary audio signal recognition are in a pretty initial stage. 
So you can't distinguish speech from other sounds because you don't know what 
other sounds are. Also, the theory of separation of overlapped signals is also 
in a very initial stage. So most of the modern VADs operate on stationary noise 
only and can not deal with complex noises and overlapped speech. Things like 
bird singing in the background can make things pretty complex.
