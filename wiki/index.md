---
layout: page 
title: CMUSphinx Documentation
---

This page contains collaboratively developed documentation for the CMU Sphinx 
speech recognition engines.

## Beginner User Documentation

This section contains links to documents which describe how to use Sphinx to 
recognize speech. 
Currently, we have very little in the way of end-user tools, so it may be a bit 
sparse for the 
forseeable future.


*  [Tutorial](tutorial): Getting started with CMUSphinx for developers
     * [ Basic concepts of speech](tutorialconcepts )
     * [ Overview of the CMUSphinx toolkit](tutorialoverview )
     * [ Before you start](tutorialbeforestart )
     * [ Building application using pocketsphinx](tutorialpocketsphinx)
     * [ Building application using sphinx4](tutorialsphinx4)
     * [ Using pocketsphinx on Android](tutorialandroid )
     * [ Building language models](tutoriallm)
     * [ Adapting existing acoustic model](tutorialadapt )
     * [ Building the acoustic model](tutorialam )
     * [ Building a dictionary](tutorialdict)
     * [ Debugging speech recognition accuracy](tutorialtuning )
     * [ PocketSphinx for pronunciation 
evaluation](pocketsphinx_pronunciation_evaluation )

**You are in trouble - read the [FAQ](faq)**

See also some more docs:


*  [ Decoder Versions](versions ): Description of the software packages

*  [ Download Details](download ): How to obtain CMUSphinx packages

*  [ How to get help and discuss things](communicate ): How to get help and 
discuss things

If you want to find out where CMUSphinx works, see 


*  [Projects that use Sphinx](sphinxinaction): These projects, both commercial 
and free, use Sphinx in one form or another.

------------------------------------------------

## Advanced User Documentation

These documents either describe some particular aspect of the Sphinx codebase 
in detail, or they serve as a
developer's guide to accomplishing some particular task.


*  [Building](building): Building Pocketsphinx on various platforms

*  [AsteriskDetails](asteriskdetails): How to use pocketsphinx in Asterisk.

*  [DecoderTuning](decodertuning): How to tune the decoder to be fast (or 
rather, not horribly slow)

*  [PocketsphinxHandhelds](pocketsphinxhandhelds) Pocketsphinx optimizations 
for embedded devices, same as above for Pocketsphinx.

*  [PhonemeRecognition](phonemerecognition): How to use pocketsphinx for 
phoneme recognition.

*  [SpeakerDiarization](speakerdiarization): Using LIUM tools for speech 
segmentation and speaker diarization

*  [LDAMLLT](ldamllt): How to train acoustic models with LDA and MLLT feature 
transforms

*  [GStreamer](gstreamer): How to use PocketSphinx with 
[GStreamer](http://gstreamer.freedesktop.org/) and [Python](http://python.org)

*  [InstallingPythonStuff](installingpythonstuff): How to install Python and 
necessary modules for SphinxTrain development

*  [MMIE_Train](mmie_train): How to perform MMIE training.

*  [ Installing on Raspberry Pi](raspberrypi )


### How To Contribute

Please consider project ideas [ProjectIdeas](projectideas), some of them are 
easy, some harder. If you want to start work on any of them, please let us know.

### Reference

These documents describe the excruciating detail of APIs, or provide other 
useful background information for CMUSphinx developers.


*  [Doxygen documentation for 
PocketSphinx](http://cmusphinx.github.io/doc/pocketsphinx/)

*  [Doxygen documentation for 
SphinxBase](http://cmusphinx.github.io/doc/sphinxbase/)

*  [ePyDoc documentation for SphinxTrain Python 
Modules](http://cmusphinx.github.io/doc/python/)

*  [JavaDocs for 
Sphinx4](http://cmusphinx.github.io/doc/sphinx4/javadoc/index.html)

------------------------------------------------

## Developer Documentation

This section contains various internal information for CMUSphinx developers. 
But we hope it will be still usable for you.


*  [Sphinx-4 Regression Tests](regressiontests): How to run regression tests

*  [SphinxTrainWalkthrough](sphinxtrainwalkthrough): An overview of the 
SphinxTrain source code for researchers and developers

*  [CMUCLMTK development](cmuclmtkdevelopment): Development guide for the 
CMU-Cambridge Language Modeling Toolkit.

*  [CodingStyle](codingstyle) for SphinxBase, SphinxThree, and SphinxTrain

*  [ReleaseSchedule](releaseschedule): Plans for upcoming releases of Sphinx

*  [ Release Check List](releaseprocess ): How to make a release

*  [ Web Site Layout](webresources ): How to organize information

*  [ Sphinx4 Space](sphinx4/webhome ) : Information about sphinx4, design, 
code, performance, history.

### File formats


*  [AcousticModelTypes](acousticmodeltypes)

*  [AcousticModelFormat](acousticmodelformat)

*  [MFCFormat](mfcformat)

*  [arpaformat](arpaformat)
### Data sources

Available data sources are covered on the page [SpeechData](speechdata)

### Speech Recognition Theory

This section tries to collect research ideas for specific problems in speech 
recognition


*  [ Lattices](asr/lattices )

*  [ WFST](asr/wfst )

*  [ Search Algorithms](asr/search )

*  [ Language Models](asr/languagemodels )

*  [ Features](asr/features )

*  [ Noise Robustness](asr/noise )

*  [ Adaptation](asr/adaptation )

*  [ Voice Activity Detection](asr/vad )
