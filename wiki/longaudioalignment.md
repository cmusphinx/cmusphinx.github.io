---
layout: page 
title: Long Audio Alignment Overview
---

Aligning audio input with its corresponding text is a well studied research 
problem in speech processing.  This page contains an overview of the current 
state of the art algorithms used for audio alignment. 

## Requirements

An aligner is supposed to identify the time when each word in the transcription 
was spoken in the utterance. So ideally we would expect the following features 
from it:    

*  **Batch Alignment**: When audio is provided in long audio files along with 
it's transcription, it should be able to align them.

*  **Live Alignment**: Alignment is done on live audio, when transcription is 
available from before.

*  **Precise** : Aligner should be able to correct the transcription where 
incorrect, and then aligns it with the audio.

*  **Error recovery** : Incorrect alignment of a certain segment of audio 
should not effect alignment of later segments.

*  **Memory efficient**: Aligner should be careful in the amount of memory 
required to store and score all hypothesis under consideration, without 
compromising with error rate of aligner.

*  **Disfluency detection and correction**: Live audio contains skipped words, 
repeated words, phrases and self-corrections. Aligner should be able to detect 
such disfluencies and correctly align it with the correct parts of the 
utterance.

*  **Minimal ASR requirement**: Aligning audio data with text for a language 
for which we don't have a well trained acoustic and language models.



## Literature Survey

| S.No | Algorithm | Link to paper  | Author     | Live Alignment | Precise | Error Recovery | MemoryEfficient | Disfluency correction | ASR required | 
 | ---- | ---------                           | -------------                                                                                                                                          | ------     | -------------- | ------- | -------------- | ---------------- | --------------------- | ------------ | 
 | 1    | "FST Aligner"                       | [http://www.sls.csail.mit.edu/sls/publications/2006/IS061258.pdf](http://www.sls.csail.mit.edu/sls/publications/2006/IS061258.pdf)                     | Hazen      | No             | Yes     | Yes            | No               | No                    | Yes          | 
 | 2    | "FST Aligner with disfluency model" | [http://www.cs.columbia.edu/~julia/papers/liu03.pdf](http://www.cs.columbia.edu/~julia/papers/liu03.pdf)                                               | Liu        | No             | Yes     | No             | No               | Yes                   | Yes          | 
 | 3    | "Anchor Points"                     | [http://www.isca-speech.org/archive/archive_papers/icslp_1998/i98_0068.pdf](http://www.isca-speech.org/archive/archive_papers/icslp_1998/i98_0068.pdf) | Moreno     | No             | No      | Yes            | No               | No                    | Yes          | 
 | 4    | "Viterbi Alignment"                 | [http://www.cs.cmu.edu/~skishore/ksp_phdthesis.pdf](http://www.cs.cmu.edu/~skishore/ksp_phdthesis.pdf)                                                 | Kishore P. | Yes            | No      | No             | Yes              | No                    | No           | 
 | 5    | "Partial Traceback"                 | [http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=1171441](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=1171441)                           | P.F.Brown  | Yes            | No      | No             | Yes              | No                    | Yes          | 

We model the speech input as a Hidden Markov Model i.e. the states of speech 
follow a Markov model which is unknown (hidden), and the sequence of observed 
features form tokens that can be used to decide which state system is in. For 
force aligning the audio data with a transcription, we assume that the 
transcription resembles the actual content in the utterance. However, this 
assumption is not always true. In real life data, utterances may contain 
disfluency or the transcription itself might not be accurate.

*  **Disfluencies** are ways in which spontaneous speech differs from written 
text, which includes repetitions, revisions and/or restarts in a sentence. 
Disfluency in an utterance is often accompanied with a change in prosodic 
features. Prosodic features reflect various features of the speaker or the 
utterance like the emotional state of the speaker; the form of the utterance 
(statement, question, or command); the presence of irony or sarcasm; emphasis, 
contrast, and focus. Liu [2] uses change in prosodic features as a clue for a 
possible disfluency in the utterance (we will call such points as interruption 
points).A Hidden Event Language Model is then used to look for repeated words 
around interruption points. If any, appropriate corrections in the alignment is 
made by correcting the disfluency.

*  If on the other hand, the **transcription is inaccurate**, aligner has to be 
designed such that it can correct such erroneous points. Hazen [1] and Moreno 
[3] use a speech recogniser with a n-gram language model that is strongly 
biased by the available transcription. Points where recogniser's output match 
the transcription for a sequence of words are considered to be accurate and are 
marked as **anchors**. Identifying anchors serves the purpose of fragmenting 
the audio hence enabling the speech recogniser to process these fragments 
separately (this also has several computational advantages which we discuss 
later).This prevents the error of alignment of one fragment to propagate to 
another fragment and also reduces the number of recursions for recognition, 
which solves the problem of **error recovery**. Moreno recursively performs the 
same operation on fragments obtained between the anchors, and this recursion 
terminates only when either the transcription is completely aligned or when the 
duration of an unaligned audio segment is less than a pre-determined threshold. 
Hence this method does not correct small errors in the transcription. In each 
of these recursions, the language model is built specific to the words in the 
transcription between the anchors. Rather than recursively choosing anchors, 
Hazen prefers to choose anchors only once with the anchor size as small as two 
words. The choice of anchor size not only determines the number of anchors, but 
also the probability of error in it's selection (smaller anchor size means more 
error).Next, a **Finite state Transducer** (FST) with out-of-vocabulary (OOV) 
filler model is used to align words between the anchors, which allows for 
insertion, substitution and deletion of existing words in transcription.A 
finite state transducer defines the set of allowed word transitions for the 
recogniser. Non-anchor words at this stage are marked for 
insertion/substitution/deletion. Lastly, the speech recogniser is re-run with 
full vocabulary over areas with marked words for possible 
insertion,substitution or deletion. Hazen's approach hence corrects the 
transcription and is **precise**.
Both Hazen and Moreno's approach rely on availability of the full audio before 
alignment for anchor selection. However, **live audio alignment** is also an 
important requirement and viterbi algorithm on large audio files forms a very 
large beam for backtracking, which requires a lot of runtime memory making it 
impractical to use on a hardware with limited memory. It is observed that as 
the size of a viterbi lattice increases, the candidates for the optimal path 
mostly share a common predecessor. For example "Open Source" , "Open Suse" and 
"Open Sauce" might all be the hypothesis under consideration, but the word 
"Open" is a common predecessor in each one of these. **Partial backtrack**[5] 
uses this observation to reduce the lattice size significantly and also gives 
almost live results without compromising with the recogniser's error rate. The 
key to this algorithm is finding an **immortal node**. An immortal node is the 
node that terminates the common predecessor in all candidates for the optimal 
path.In our previous example, "Open" was an immortal node. By definition, word 
sequence preceding an immortal node is a part of the optimal path and hence can 
be recognised and taken out of the lattice.
In **Automatic speech and speaker recognition: Advanced topics**, Kuldip 
Paliwal chalks out a simplified algorithm for Partial traceback. The procedure 
involves tracing back from the current active nodes and incrementing counter 
for the number of first descendants for each node encountered, until the last 
immortal node is reached. All **dead nodes** (ones with no first descendants) 
are deactivated after this step and the last immortal node is checked for it's 
number of descendants. While this number is exactly one, it's descendant can be 
treated as the new immortal node. This way we constantly reduce the size of the 
lattice, keeping bounds on the memory requirements for alignment. Another 
important and very commonly occurring  issue that remains to be answered is the 
case when well trained acoustic model are not available for the language used 
in the utterance. Kishore [4] uses the available transcription for a first 
rough estimate of acoustic model and the uses **Baum-Welch algorithm** to 
re-estimate the model using another well trained acoustic model.

## Recovery From Mis-Alignment

Long audio aligner in Sphinx 4 uses Out-of-Vocabulary model and grammar 
modifications to model dis-fluencies and to recover from mis-alignments. In 
real world scenarios, mis-alignments are inevitable, hence a modest objective 
would be to have a good recovery from such misalignment.
Following is a table of experiments that can give a better understanding of 
optimal aligner's variables for long audio aligner:

