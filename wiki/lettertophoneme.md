---
layout: page 
title: Letter to Phoneme Conversion in CMU Sphinx-4
---

Project progress will be also available at my personal blog at: 
<http://jsalatas.ictpro.gr/category/projects/gsoc-2012/>

## Abstract

Currently sphinx4 uses a predefined dictionary for mapping words to sequence of 
phonemes. I propose modifications in the sphinx4 code that will enable it to 
use trained models (through some king of machine learning algorithm) to map 
letters to phonemes and thus map words to sequence of phonemes without the need 
of a predefined dictionary. A dictionary will be only used to train the 
required models.


## Literature Review

Grapheme to Phoneme (G2S) (or Letter to Sound – L2S) conversion is an active 
research field with applications to both text-to-speech and speech recognition 
systems. There are many different approaches used for the G2S conversion 
proposed by different researchers. Some of them that I have already review are 
the following.

 

Hein [1] proposes a method that use a feedforward neural network for the G2S 
conversion process and proposes a simple algorithm for the creation of the 
grapheme-to-phoneme matching database with a phonetic dictionary as input. The 
method has been tested in both English and German languages. 99.2% of 305000 
entries of the German CELEX could be completely mapped and 91.8% of 59000 
entries of the English PRONLEX.

 

Stoianov et al. [2] propose the use of Simple Recurrent Network (SRN) [3] for 
learning grapheme-to-phoneme mapping in Dutch. They conclude that SRN performs 
well on training and unseen test data sets even after very limited number of 
training epochs. Also, there were significant consistency and frequency effects 
on error.

 

Daelemans and Van Den Bosch [4] propose a data-oriented language-independent 
approach to grapheme-to-phoneme conversion problem. Their method takes as input 
a set of spelling words with their associative pronunciation, which do not have 
to be aligned, and produces as its output the phonetic transcription according 
to the implicit rules in the training dataset. The method is evaluated for the 
Dutch language with a 95,1% accuracy in unseen words.

 

Most recent approaches (e.g. Jiampojamarn et al. [5] and Bisani and Ney [6]) 
show that multiple letter-to-phoneme alignments perform better than single 
letter-to-phoneme alignment. Also the proposed method in [5] tries to combine 
the various stages of letter-to-phoneme conversion (Letter segmentation, 
Phoneme classifier, Sequence model) into one single step which updates 
parameters according to a comparison of the current system output to the 
desired output.


##  Timeline

An initial timeline that I propose would be as follows:
##### 13 May — 20 May Task 1: openFST

Complete of the openFST basic classes (arc and fst) in java and 
serialization/deserialization related code.

##### 21 May — 06 June Port Task 2: m2m-aligner.py to C++

The first step of training procedure (the dictionary alignment) will be done in 
C++ (the allignment code will be used also in Kaldi project)

##### 04 June — 17 June Task 3: Simplify the model training process

The training script will be ported to C++. The original script uses the MIT 
Language Modeling (MITLM) toolkit [15], we will use the OpenGrm NGram Library 
[16]¸ which is currently using the openFST library. 

##### 18 June — 15 July Task 4: Port evaluate.py in java

First step is to write the required code to load the binary fst model in java. 
The model evaluation script contains the most of the openfst operations needed. 
After that step all the required openfst functionality will be available in 
java.

##### 16 July — 29 July Task 5: Integration with sphinx4

Most of the code required will be already available. The task here is to 
integrate it into sphinx4. 

##### 30 July — 05 August Task 6: Testing

Code review and testing. Creation/review of junit tests. Review and 
complete/correct documentation.

##### 06 August — 19 August Task 7: Training of additional language models

All of the previous steps will be completed using english language. Eventually 
at this point we will have already an english language model. 
In this step I will create models for additional languages like spanish, 
french, etc. 

##  Current Progress

Tasks 1 to 3 are complete. 

Task 4 is about 90% complete, however code is not submitted yet as I'm still 
refactoring it to be more comprehensible and easy to understand/maintain. 
hopefully the fst java library will be completed at about 12th of July and the 
java decoder and evaluation code until 16th July as per the schedule.


## References

[1] H. U. Hein, “Automation of the training procedures for neural networks 
performing multi-lingual grapheme to phoneme conversion”, EUROSPEECH'99, pp. 
2087-2090, 1999.

 

[2] I. Stoianov, L. Stowe, and J. Nerbonne, “Connectionist learning to read 
aloud and correlation to human data”, Proceedings of the 21st Annual Conference 
of the Cognitive Science Society. Hillsdale, NJ: Erlbaum, 1999.

 

[3] J.L. Elman, “Finding structure in time”, Cognitive Science, 14, pp. 
213-252, 1990.

 

[4] W. Daelemans, A. V.D. Bosch, "Language-independent data-oriented 
grapheme-to-phoneme conversion", Progress in Speech Synthesis, pp. 77–89. New 
York, USA, 1997.

 

[5] S. Jiampojamarn, C. Cherry, G. Kondrak, “Joint Processing and 
Discriminative Training for Letter-to-Phoneme Conversion”, Proceeding of the 
Annual Meeting of the Association for Computational Linguistics: Human Language 
Technologies (ACL-08: HLT), pp.905-913, Columbus, OH, June 2008.

 

[6] M. Bisani, H. Ney, “Joint-Sequence Models for Grapheme-to-Phoneme 
Conversion”, Speech Communication, Volume 50, Issue 5, pp. 434-451, May 2008.


[7] <http://code.google.com/p/phonetisaurus/>


[8] <http://www-i6.informatik.rwth-aachen.de/web/Software/g2p.html>


[9] D. Jouvet, D. Fohr, I. Illina, "Evaluating Grapheme-to-Phoneme Converters 
in Automatic Speech Recognition Context", IEE International Conference on 
Acoustics, 2012


[10] CRF Project Page, <http://crf.sourceforge.net/>

 

[11] OpenFST Library, <http://www.openfst.org/twiki/bin/view/FST/WebHome>

 

[12] Javadoc for package marytts.fst, 
<http://mary.dfki.de/javadoc/4.0.0/marytts/fst/package-summary.html>

 

[13] The MARY Text-to-Speech System, <http://mary.dfki.de/>

 

[14] [Phonetisaurus: A WFST-driven Phoneticizer – Framework 
Review](http://jsalatas.ictpro.gr/phonetisaurus-a-wfst-driven-phoneticizer-frame
work-review/) 


[15] [ MIT Language Modeling Toolkit](http://code.google.com/p/mitlm/ )



[16] [ OpenGrm NGram 
Library](http://www.openfst.org/twiki/bin/view/GRM/NGramLibrary )


[17] [ Porting openFST to java: Part 
2](http://cmusphinx.github.io/2012/05/porting-openfst-to-java-part-2/ )
