---
layout: page 
title: Pronunciation evaluation for GSoC 2012
---

Introductory conference paper: 
https://docs.google.com/open?id=0B73LgocyHQnfS0g5ZEw1aFNKT2s

Source code repository: 
http://cmusphinx.svn.sourceforge.net/viewvc/cmusphinx/branches/speecheval/

Project blog: http://pronunciationeval.blogspot.com/

Please donate to support: http://talknicer.com/slics

## Plan

 | Task                                                                       | 
Status                                               | Assignee(s)         | 
Comments                                                             | 
 | ----                                                                       | 
------                                               | -----------         | 
--------                                                             | 
 | Alignment tests                                                            | 
done                                                 | both                | 
week 0 and 1                                                         | 
 | rtmplite upload                                                            | 
mostly done                                          | Troy                | 
week 0 and 1                                                         | 
 | wami-recorder upload                                                       | 
done                                                 | Ronanki             | 
week 0                                                               | 
 | speex quality testing                                                      | 
done                                                 | Troy                | 
week 1                                                               | 
 | Neighbor phoneme edit distance grammar generation                          | 
done                                                 | Ronanki             | 
week 1                                                               | 
 | Base edit distance/alignment scoring routines                              | 
done                                                 | Ronanki             | 
week 3                                                               | 
 | Database schema (see below)                                                | 
60-90%                                               | Troy                | 
completion level pending testing                                     | 
 | User login/password/cookies framework                                      | 
alpha release pending documentation                  | Troy                | 
Need docs for UI parts to use this                                   | 
 | Phrase data entry user interface                                           | 
50%                                                  | both                | 
both algorithm and schema components                                 | 
 | User interface for exemplar uploads                                        | 
on hold                                              | Troy?               | 
pending Flash/WebRTC audio upload pivot                              | 
 | User interface for learners                                                | 
25%                                                  | all                 | 
this is a general client implementation task                         | 
 | Manual scoring user interface                                              | 
begun                                                | Both                | 
involves both schema and client UI interface                         | 
 | Fail-over from Flash/rtmplite when unavailable to `<input type=file>`        
| begun                                                | Ronanki             |  
                                                                    | 
 | Exemplar phoneme acoustic score and duration means and std. deviations     | 
done                                                 | Ronanki             | 
document score aggregation functions in the top-three useful modules | 
 | Calculation of exemplar quantity sufficiency                               | 
begun                                                | Ronanki             | 
also converting Python to PHP                                        | 
 | Aggregate acoustic score, duration, and neighbor phoneme to phoneme scores | 
almost done                                          | joint               | 
only Ronanki at present; will swap during code reviews               | 
 | Aggregate phoneme scores to biphone score                                  | 
started                                              | joint               | 
both                                                                 | 
 | Aggregate phoneme scores to word scores                                    | 
almost done                                          | joint               | 
both                                                                 | 
 | Aggregation from word (and phoneme?) scores to phrase scores               | 
done                                                 | joint               | 
both                                                                 | 
 | Validation of exemplar recordings (outlier scores)                         | 
started                                              | joint               | ?  
                                                                  | 
 | Phrase library development                                                 | 
started                                              | Troy                | 
Mechanical Turk                                                      | 
 | Exemplar collection                                                        | 
pending Flash/WebRTC/wami-recorder/type=file outcome | Troy                | 
Mechanical Turk                                                      | 
 | Measurement against panel of native speakers' manual scores                | 
not started                                          | Ronanki             | 
need stats/graphs                                                    | 
 | Game authoring interface                                                   | 
see schema                                           | Troy                | 
vfront.org?                                                          | 
 | Game interface                                                             | 
see schema                                           | Troy                | ?  
                                                                  | 
 | Measurement of phonological features                                       | 
in progress                                          | Ronanki             | 
need stats/graphs                                                    | 
 | Android version                                                            | 
pending fundraising                                  | Troy and/or Guillem | ?  
                                                                  | 
 | iOS version                                                                | 
started                                              | Andrew Lauder       | 
similar to web client                                                | 
 | OLPC versions                                                              | 
some work                                            | James               | 
have XO-1.75, getting -1 and -1.5                                    | 
 | Stand-alone versions                                                       | 
not started                                          | everyone            | 
measure memory and speed                                             | 
 | Write up                                                                   | 
accepted for publication                             | everyone            | 
http://docs.google.com/file/d/0B73LgocyHQnfS0g5ZEw1aFNKT2s/edit      | 

Draft: http://talknicer.net/w/To_do_list

## Database schema

Please see http://talknicer.net/w/Database_schema which will be included here 
as its portions pass testing.

## HTML/RTMP client protocol(s)

Update: RTMP from Flash microphone upload, or port 80 with multipart background 
HTTP form uploads, in Speex quality 8/10 or better quality, to PCM 16,000 
samples/second, 16 bit samples. Looking forward to WebRTC, but until then 
https://code.google.com/p/wami-recorder/ is okay.

# Ronanki

## Title

** Web-Based Pronunciation Evaluation Using Acoustic, Duration and Phonological 
Scoring with CMU Sphinx3 **

[Accepted GSoC 2012 Project 
Proposal](http://www.google-melange.com/gsoc/proposal/review/google/gsoc2012/ron
anki/1) 

## Project Short Description

Feedback on pronunciation is vital for spoken language teaching. Automatic 
pronunciation evaluation and feedback can help non-native speakers to identify 
their errors, learn sounds and vocabulary, and improve their pronunciation 
performance. Such speech recognition can be performed using Sphinx trained on a 
database of native exemplar pronunciation and non-native examples of frequent 
mistakes. Adaptation techniques based on such databases can obtain better 
recognition of non-native speech. Pronunciation scores can be calculated for 
each phoneme, word, and phrase by means of Hidden Markov Model alignment with 
the phonemes of the expected text. In addition to such acoustic alignment 
scores, we can also use edit distance scoring to compare the scores of the 
spoken phrase with those of models for various mispronunciations and alternate 
correct pronunciations. These scores may be augmented with factors such as 
expected duration and relative pitch to achieve more accurate agreement with 
expert phoneticians’ average manual subjective pronunciation scores.  Such a 
system will be built and documented using the CMU Sphinx3 system and an Adobe 
Flash microphone recording, HTML/JavaScript, and rtmplite/Python user interface.

## Project Goals

1. To build a website capable of collecting recorded audio speech utterances 
for use as both pronunciation exemplars and student utterances to be evaluated 
as part of the system’s user interface. I will use an open source Adobe 
Flex/Flash microphone recording and audio upload web client applet and rtmplite 
(Python) server which the project mentor has agreed to provide. I will use this 
website as the user interface to the project and to collect data which might 
not be available from my existing speech databases (or e.g. http://librivox.org 
and http://www.voxforge.org.) This website will also allow me to collect human 
judges’ pronunciation assessment scores for use in training and validation of 
the automatic pronunciation evaluation.

2. To build a pronunciation evaluation system that can detect mispronunciations 
at the phoneme level and provide feedback scores at the phoneme, bi-phone, 
word, and phrase level. I will use scoring techniques such as standardizing 
acoustic phoneme scores, edit distance scoring with alternate pronunciation 
grammars, phoneme duration standard scores, and standard score aggregation 
across biphones, words, and phrases. 

3. If time permits, I hope to investigate mapping the acoustic speech features 
of each phoneme or diphone derived from machine phonetic transcription to 
articulation-based phonological features. Each phoneme is characterized by 
place of tongue articulation, manner of articulation, lip rounding, voiced or 
unvoiced, duration, height, frontness etc., These features represent the 
physical characteristics of the speech production process. Using this mapping, 
mispronunciations at the phone level can be identified using phonological 
features along with acoustic pronunciation scores and edit distances.

## Milestones

1. Benchmark the memory utilization and performance of Sphinx3 on forced 
alignment and edit distance scoring tasks on the initial website’s server 
equipment for parameter optimization and resource planning purposes.

2. Download the Adobe Actionscript Flex compiler and use it to build the open 
source Flex/Flash microphone audio input and upload applet to be provided by 
the mentor.

3. Install the rtmplite server on website and test it with microphone upload 
applet.

4. Write and test JavaScript and back-end server software for audio and 
pronunciation score data collection.

5. Write and test back-end server software for phrase word phoneme string 
selection, using CMUDICT, part-of-speech selection, and phrase audio exemplar 
collection.

6. Write and test phoneme acoustic score and duration standard score 
aggregation routines from exemplar utterance recordings.

7. Write and test audio upload and pronunciation evaluation for per-phoneme 
standard scores.

8. Write and test score aggregation by words, phrase, and biphones.

9. Write and test score output display, possibly using an open source 
JavaScript mouseover library.

10. Collect sufficient transcribed phrases and corresponding exemplar 
recordings such that the system can be shown to be useful for general 
educational purposes.

11. Measure the agreement of the automatic evaluation scores with experts’ 
average subjective scores.

12. Write and test learner user accounts and adaptive feedback system to prompt 
learners to pronounce phrases which include phonemes, biphones, diphones, or 
words which they need to practice.

13. Publicize the system so that it can be tested by large numbers of users and 
monitored while in use.

14. Write and test an edit distance score enhancement system using an 
automatically generated mispronunciation grammar to the phoneme 

15. Measure the benefit of including edit distance scoring in the calculation 
of phoneme scores.

16. (If the schedule at this point in the project permits:) 


*  Build a CART model for the training dataset so that the test phonemes can be 
compared to the correct phone with reference to their context.


*  Build a table to transform phonemes and/or biphones derived from machine 
phonetic transcription to a new set of articulation-based phonological  
features.


*  Write and test a supplementary scoring routine to aggregate 
articulation-based phonological features with existing standard scores.


*  Measure the improvement of including articulation-based phonological 
features.

17. Write initial project documentation.

18. Proofread project documentation with mentor review.

19. Update the wiki at 
http://cmusphinx.github.io/wiki/faq#qhow_to_implement_pronunciation_evalua
tion with the project description, pointing to the project documentation and 
including source code download instructions.

20. Prepare a research report on the project, proofread with mentor review, and 
submit it to a preprint server and a peer reviewed academic journal.

21. (Ongoing after the summer, if necessary.) Respond to journal submission 
comments, questions, and requests, and if rejected submit to other journal(s).

## Schedule

 1.  Prior to May 21:  Become familiar with (1) wami-recorder, (2) Adobe Flex 
mxmlc, (3) audio upload using those two tools and conversion to PCM, (4) using 
subversion to upload the codes, and (5) the JavaScript interface to the OverLib 
mouseover library, (6) forced alignment and edit distance grammars using 
Sphinx3, and if the project is approved on April 23 and time away from studies 
permits, get a head start on the milestones below.
 2.  May 21 to 27: milestones 1-4
 3.  May 28 to June 3: milestones 5-7
 4.  June 4 to 10: milestones 8 and 9
 5.  June 11 to 17: milestones 10 and 11
 6.  June 18 to 24: milestones 12 and 13
 7.  June 25 to July 1: milestones 14
 8.  July 2 to 8: milestone 15
 9.  July 9 (firm): Complete and submit mid-term evaluation.
 10.  July 9 to 15: milestone 16 part (a)
 11.  July 16 to 22: milestone 16 part (b)
 12.  July 23 to 29: milestone 16 parts (c) and (d)
 13.  July 30 to August 5: milestones 17 and 18
 14.  August 6 to 13: milestones 19 and 20
 15.  August 14 to 19: buffer in case of schedule overrun

# Troy

## Title

** Mobile Pronunciation Evaluation for Language Learning Using Edit Distance 
Scoring with CMU Sphinx3, Copious Speech Data Collection, and a Game-Based 
Interface ** 

[Accepted GSoC 2012 project 
proposal](http://www.google-melange.com/gsoc/proposal/review/google/gsoc2012/tro
ylee2008/1)

## Project Short Description

Pronunciation learning is one of the most important parts for second language 
acquisition. The aim of this project is to utilize the automatic speech 
recognition technologies to facilitate spoken language learning. This project 
will mainly focus on developing accurate and efficient pronunciation evaluation 
system using CMU Sphinx3 and maximizing the adoption population by implementing 
mobile apps with our evaluation system. Additionally, we also plan to design 
and implement game based pronunciation learning to make the learning process 
much more fun. Four specific sub-tasks are involved in this project, namely, 
automatic edit distance based grammar generation, exemplar pronunciation 
database building, Android pronunciation evaluation app interface 
implementation and game based learning interface development.

## Project Goals

1. To automatically generate edit distance grammars for pronunciation 
evaluation. Language learners tend to make similar pronunciation mistakes, 
especially for learners from the same region or sharing the same mother tongue 
language. Identifying these mispronunciation patterns would greatly reduce the 
search space and improve the evaluation efficiency while maintaining the 
evaluation accuracy. This task would involve automatic mispronunciation pattern 
learning using native and non-native speech data, grammar generation using 
those patterns mined from speech data and testing the recognition grammar. The 
grammar will be finally represented as phoneme network/lattice. This would be 
in Python using Sphinx3 and would probably take 3-5 weeks.

2. To build a exemplar pronunciation database for pronunciation evaluation. As 
mentioned in the first task, to achieve efficient and accurate pronunciation 
evaluation, we need non-native speech data for mispronunciation pattern mining. 
In this task, we need to recruit people to come and visit a website and record 
their pronunciation of phrases. Then we need post-process the recorded speech 
samples and build a database for future system development. The recording 
website will be provided by the mentor and I will invite my friends to 
contribute their speech to this database and do data post-processing with some 
automatic approaches, such as outlier analysis for rejecting obvious bad 
pronunciations and speech detection for cropping the signal etc. This would be 
an ongoing thing to take 4-6 hours per week.

3. To implement an Android interface to a pronunciation evaluation system. To 
make our pronunciation evaluation system accessible to more users, we plan to 
build an Android app for our pronunciation evaluation system.  This would be a 
Java task to take an existing pronunciation evaluation system for the web and 
make an Android interface to it. I will implement the audio recording and 
playback functions on Android platform and client-server interaction for 
transferring recorded speech signals and evaluation results. Simple HTTP based 
client-server communication will be adopted for this taks. This would take 2-4 
weeks.

4. To develop a game front end for a pronunciation evaluation system. The best 
way to attract users is to implement the product as a game. We will explore 
this idea to make our system more popular. This task would be design and 
implementation of a simple game front end on web and/or Android to increase the 
attractiveness of working on pronunciation evaluation practice tasks for 
students. This task is a much more challenging one. The final game mechanism 
and implementation will be further discussed with the mentor. Generally 
speaking, I will design the basic game play functionalities and the interfaces. 
Similar client-server communication as the previous one will be also adopted 
here. This would take the remainder of the time, as fancy as you want to make 
it.

## Milestones

1.      Get familiar with Sphinx3 and setup the baseline of the existing 
pronunciation evaluation system to be provided by the mentor.

2.      Get both the native and non-native speech data from the mentor and 
extract MFCC features for recognition.

3.      Automatically decode the speech data with acoustic model and language 
model provided by the mentor.

4.      Extract mispronunciation patterns in those data.

5.      Automatically construct grammars to include both the correct 
pronunciation and possible mispronunciation patterns.

6.      Testing the generated grammars in the baseline evaluation system and 
comparing the performance.

7.      Analyze the results, if necessary repeat 3-6 to improve the evaluation 
performance until a better grammar is learnt.

8.      Collect information about the exemplar pronunciation data collection 
website and process from the mentor, if needed, help setup the website and 
promote the data collection among my friends.

9.      Implement automatic data post-process programs for the recording data 
post-processing.

10.    Guarding the post-processing process and occasionally do verifications 
manually if necessary.

11.    Settle down the functionality design for the Android app.

12.    Settle down the interface design for the Android app.

13.    Implement the audio recording and playback on Android platform.

14)    Implement the HTTP based client-server file communication.

15.    Testing the Android app and fixing bugs.

16.    Discuss with the mentor about the game mechanism.

17.    Settle down the game play logic.

18.    Settle down the functionality design.

19.    Settle down the interface design.

20.    Game implementation and testing.

21.    Prepare the final project report.
