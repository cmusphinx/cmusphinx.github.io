---
layout: page 
---
#  Project Ideas 

Note to students: Google Summer of Code proposals are closed. 

This page was updated March 2017. 

Please see also: [summerofcodeideas](summerofcodeideas) and 
[summerofcodestudents](summerofcodestudents)

For reference: 
http://write.flossmanuals.net/gsoc-mentoring/making-your-ideas-page/

#  About CMUSphinx 

The CMUSphinx project is a leading automatic speech recognition project in the 
open source world. Since being released as open source code in 1999, it 
provides a platform for building speech recognition applications. It's used in 
desktop control software, telephony platforms, intelligent houses, 
computer-assisted language learning tools, information retrieval and mobile 
applications. Traditionally, CMUSphinx provides support for low-resource and 
underdeveloped languages.

# Tasks for Contributors

This is a list of potential tasks for new and intermediate contributors to the 
project. If you have some other idea, feel free to add it. For any questions 
contact us on [ 
cmusphinx-devel@lists.sourceforge.net](https://lists.sourceforge.net/lists/listi
nfo/cmusphinx-devel ) mailing list or on #cmusphinx irc channel on Freenode.

## Intelligibility remediation for pronunciation assessment

In 2014 and 2015, a few speech recognition labs working on speech recognition 
for pronunciation assessment in language learning and computer assisted 
pronunciation training started using a new method for authentic intelligibility 
remediation. This technique vastly improves upon scoring a learner's 
pronunciation by trying to predict the assessment of pronunciation judges, 
which is the paradigm which has dominated the field since the Malvern UK STAR 
project in the late 1980s. Instead, we assess the quality of a student's 
pronunciation by trying to predict whether a panel of native and nonnative 
transcriptionists can correctly discern the words that the learner was supposed 
to say. This is a quantum-level improvement in the field which we want to 
implement with PocketSphinx.

Our initial plans are to provide working demonstration code in English and one 
other language (e.g., Kannada or Tamil) along with crowdsourcing transcription 
and exemplar pronunciation infrastructure, and sufficiency index statistics for 
the number of transcripts and exemplar utterances provided per target phrase. 

** Mentors **

James Salsman (GSoC 2012 mentor) jim at talknicer dot com

Srikanth Ronanki (GSoC 2012 student) srikanth dot 143 dot iiit at gmail dot com

** Interested student(s) email James Salsman if you are interested **

Balaji VenkataRamanan - full-time Ph.D. scholar at Christ University, 
Bangalore, India - balajiv211 at yahoo dot com

** Complexity **

Medium

** Skills **

C (PocketSphinx) and Python (2012 GSoC pronunciation assessment project)

** PLEASE SEE 
[pocketsphinx_pronunciation_evaluation](pocketsphinx_pronunciation_evaluation) 
**
## Diphone alignment and acoustic scores

We would like to modify PocketSphinx so that it creates meaningful time 
alignment and accurate acoustic scores from HMM posterior probability scores 
for phonemes as well as diphones. (A diphone is the second half of one phoneme, 
other than a dipthong or other composite, followed by the first half of 
another. There are about 1,400 diphones in Basic English.)

** Complexity **

Easy

** Skills **

C

** Possible mentors **

James Salsman - jim at talknicer dot com 

** Interested potential students **

Email James Salsman to add your name here.

## DNN acoustic models for pocketsphinx

For the best accuracy we need to introduce neural network based acoustic model

** Complexity **

Medium

** Skills ** 

C, Python, Tensorflow or other DNN toolkit

## CNN-LSTM models for sphinx4

Same as above, but more heavyweight models for large vocabulary sphinx4 decoder

** Complexity **

Medium

** Skills ** 

C, Python, Java, Deeplearning4j, Tensorflow or other DNN toolkit

## ROS integration

The Robot Operating System (ROS) is a set of software libraries and tools that 
help you build robot applications. From drivers to state-of-the-art algorithms, 
and with powerful developer tools, ROS has what you need for your next robotics 
project. And it's all open source.

ROS has support for pocketsphinx, but it is very initial stage. A full-scale 
support for recent features would be very welcome.

** Complexity **

Easy

**Skills**

C, Python

** Note **

http://www.ros.org
http://wiki.ros.org/pocketsphinx
## UniMRCP integration

UniMRCP is a popular telephony server for IVR speech recognition. It has some 
initial support for pocketsphinx, but, as usual it is not optimal configuration 
and very complex to install. It would be nice to enable full pocketsphinx 
support in UniMRCP to enable IVR developers use accurate and open source speech 
recognition.

** Complexity **

Easy

**Skills**

C

** Note **

https://code.google.com/p/unimrcp/wiki/PocketSphinxPlugin

## Speaker Verification and Identification framework

Speech biometrics gaining more and more attention recently. We don't just need 
to decode the speech, we also need to understand who said what. Beside 
biometrics speaker identification is important for speech recognition itself 
because it enables better adaptation.

** Complexity **

Medium

**Skills**

C, Java

** Note **

There are many publications about speaker identification, it's probably better 
to start with a book

Fundamentals of Speaker Recognition by Homayoon Beigi
http://www.springer.com/de/book/9780387775913

##  Java LM Training code

All C language model training tools have issues with portability, it is very 
hard to run them on Windows. It would be nice to implement a full-scale LM 
training tool in Java to be used instead of CMUCLMTK. That should support 
variety of unformatted text inputs and provide competitive language model for 
further use in our decoders.

** Complexity **

Medium

**Skills**

Java

** Note **

The best toolkit nowdays is SRILM, http://www.speech.sri.com/projects/srilm/, 
we can take a lot of ideas from it. Language models and training algorithms are 
best explained in "Spoken Language Processing" textbook.
 
## Very large scale grid-based model training

Have you heard about SETI project or Folding@HOME? The idea is that you can 
give everyone an easy opportunity to share CPU power to some project. So people 
could simply install the screensaver on their desktops and help to train a very 
large acoustic model. That seems to be a great opportunity to increase 
computational performance of the project resources.

The goal is to port CMUSphinx acoustic model training process to a distributed 
grid framework so everyone could donate the resources for best possible speech 
recognition accuracy. The port should outsource most computationally hard tasks 
to the grid, mainly it's a Baum-Welch estimation (bw binary in acoustic model 
training tutorial).

The project can use BOINC for the grid middleware or some Java analog of BOINC 
if you prefer to code in Java. We prefer Java.

** Resources ** 

A server for grid experiments will be provided

** Expected results **

Training process can submit bw task and features for it to the grid and train 
the acoustic model this way.

The setup and code is well documented in the project wiki

** Complexity **

Easy

**Skills**

C, Java

** Note **

[ BOINC Project ](http://boinc.berkeley.edu/ )

## Large scale grid-based model optimization

Previous project deals with existing training framework, but we can try 
something very new. We can try to build the best possible acoustic model with 
brute force. You know, it's better to spend electricity on speech recognition 
than on search for the aliens who have already contacted us first.
    
The acoustic model is just a set of about 1000 parameters. You can apply a 
simple large-scale optimization algorithm to create the best acoustic model 
possible. You need to distribute evaluation data on the grid and just select 
the best parameters possible. You can start from existing model approximation 
and try to improve it.

** Resources ** 

A server for grid experiments will be provided

** Expected results **

The setup is in place and the model training is running. The model demonstrates 
reasonable improvement over baseline on our task.

** Complexity **

Hard

** Skills **

C, Java, Optimization methods

** Note **

[ BOINC Project ](http://boinc.berkeley.edu/ )

[ Numerical Optimization Texbook 
](ttp///www.springer.com/mathematics/book/978-0-387-30303-1 )

## Collect pronunciation dictionaries from Wikipedia

It's critical to reuse existing information sources to improve system 
performance and support more languages. We also need to incorporate quickly the 
pronunciation for many new words which appeared just last year and missing in 
the common dictionaries. Think about how to pronounce the word "gangnam". One 
valuable source of phonetic pronunciations is Wikipedia, in particular 
Wiktionary project. It often contains pronunciations for many words in IPA 
format created by the dictionary authors. However, it's not trivial to parse 
the pages in uniform way since page format is different from language to 
language.

You need to write a code to parse existing Wiktionary pronunciations and create 
the dictionary for many langauges. It's not that trivial task as it might seem 
as the code has to work for many Wikipedia languages.

** Expected results **

Phonetic dictionaries for 10 languages are collected from Wikipedia. The tool 
is documented and is easy to use to discover pronunciations for the new words.

** Complexity **

Easy

** Skills **

Python

** Note **

http://en.wiktionary.org/wiki/Wiktionary:Main_Page

## Implement Java trainer on Hadoop framework

Hadoop is a framework for distributed computation an it's enabled processing of 
the huge databases. Sphinx4 implements java training already, but this training 
is not parallel. The port of the Java trainer to support training from Hadoop 
would allow us to scale significantly beyond the simple training setups.

** Expected results **

Train Sphinxtrain acoustic models using Hadoop framework

** Complexity **

Easy

** Skills **

Java, Hadoop

** Notes **

[ Apache Mahout, machine learning using Hadoop ](http://mahout.apache.org/ )
[ Apache Hadoop ](http://hadoop.apache.org/ )


http://en.wikipedia.org/wiki/Mixture_model

## Implement WER evaluation framework

Our current tools for WER evaluation are pretty simplistic, they only accept 
preformatted text in a certain format, they do not provide extendend statistics 
required for model optimization, they do not allow to estimate the quality of 
the engine and suggest help on the model issues. The goal is to implement an 
advanced WER scoring tool which is both able to process large files and at the 
same time has extensive reporting functionality like NIST sclite toolkit.

** Expected results **

Advanced error scoring tool implemented

** Complexity **

Easy

** Skills **

Java, C, Python

** Notes **

[ SCLite scoring tool 
](http://www1.icsi.berkeley.edu/Speech/docs/sctk-1.2/sclite.htm )

## Support PocketSphinx in JavaScript

https://github.com/syl22-00/pocketsphinx.js [(live 
demo)](https://syl22-00.github.io/pocketsphinx.js/live-demo.html) has been 
progressing well, and it's fast enough to handle many tasks in better than real 
time on low-end semi-loaded laptops, and probably tablets. It won't be long 
before it's a viable fully client-side solution on phones.

** Expected results **

Support the existing code base with examples and cookbook-style solutions.

** Complexity **

Easy to medium

** Skills **

JavsScript (including WebRTC GetUserMedia), C, possibly Adobe Flash Actionscript

** Contact **

jim at talknicer dot com

## Deep neural networks for Sphinx

In this project we aim to develop DNN frameworks for incorporation into the 
open source Sphinx systems. Over the course of the summer we will train, test 
and finally incorporate two frameworks into one or both of Sphinx4 and 
Pocketsphinx.  The project is expected to require two students.

Mentors:  Bhiksha Raj (bhiksha@cs.cmu.edu) and Evandro Gouvea 
(egouvea@gmail.com)

## Javascript simulation of the brain's motor control of the vocal tract

We need someone to do [one of these, but translucent with labels and more 
degrees of freedom to swivel and zoom, and 
animations](https://www.koshland-science-museum.org/explore-the-science/interact
ives/brain-anatomy) showing the projections from 
[these](https://www.researchgate.net/profile/David_Reby/publication/293176005_Vo
ice_Modulation_A_Window_into_the_Origins_of_Human_Vocal_Control/links/56b6810708
aebbde1a79ecb7.pdf) 
[three](https://www.researchgate.net/profile/Daniel_Carey2/publication/303866796
_Magnetic_resonance_imaging_of_the_brain_and_vocal_tract_Applications_to_the_stu
dy_of_speech_production_and_language_learning/links/57602e3008ae227f4a3eff31/Mag
netic-resonance-imaging-of-the-brain-and-vocal-tract-Applications-to-the-study-o
f-speech-production-and-language-learning.pdf) 
[articles](https://www.researchgate.net/profile/Samuel_Evans3/publication/312179
402_Comprehending_auditory_speech_previous_and_potential_contributions_of_functi
onal_MRI/links/5874e1fc08ae8fce4927e6c8.pdf) using [these 
polygons](https://code.google.com/archive/p/open-3d-viewer/source/default/source
) with [these kinds of 
animations](https://youtube.com/watch?v=nREdqCM8RXs&feature=youtu.be&t=35s) 
using [canvas-based](http://www.kevs3d.co.uk/dev/phoria/test0r.html) Phoria 
[including translucence](http://www.kevs3d.co.uk/dev/phoria/test1t.html) like a 
3D version of [this thing.](http://smu-facweb.smu.ca/~s0949176/sammy/)

** Expected results **

We can embed such interactive JavaScript animations wherever we have difficult 
tongue position diagrams, among many other places, e.g. at 
https://en.wikipedia.org/wiki/Diphthong#Closing.2C_opening.2C_and_centering

** Complexity **

Medium

** Skills **

JavsScript

** Contact **

jim at talknicer dot com

Please see also http://dood.al/pinktrombone/

## Computer-aided instruction system capable of teaching speaking skills

We would like to make a general, open-source computer aided instruction which 
can teach speaking skills along with reading, listening, and writing.

** Expected results **

[Accomplish these 
tasks](https://drive.google.com/file/d/0B73LgocyHQnfU2F6NVhHRzFNWTA/view) to 
make a general purpose  [computer-aided instruction system capable of teaching 
speaking 
skills.](https://drive.google.com/file/d/0B73LgocyHQnfVjlHb1cwenROcG8/view) Can 
you use [the accuracy review 
system](https://priyankamandikal.github.io/posts/gsoc-2016-project-overview/) 
to provide default instructional content questions while allowing voice 
response of phrases describing the approach to each question?

** Complexity **

Hard

** Skills **

JavsScript, Python, Android and iOS microphone input integration with 
Javascript, possibly Adobe Flash ActionScript. Please consider integration with 
Moodle in your project proposal.

** Contact **

jim at talknicer dot com


