---
layout: page 
---
# Postprocessing Framework


Postprocessing Framework refers to a part of the speech recognition process in 
which the word stream resulted in the basic recognition process is sentence 
segmented, punctuation is recovered, capitalization is performed and 
abbreviations are made when needed. Also, numbers and other types of special 
data should be converted to usual form from the words. This aims to improve 
legibility and enhance information for future human and machine processing.

Speech segmentation in sentences is an important sub-problem of speech 
recognition and depends on context, grammar and semantics. This task requires 
non-trivial techniques, such as statistic decision making.

Spoken language is typically less organized than textual material, making it a 
challenge to bridge the gap between spoken and written material.

The insertion of punctuation marks into spoken texts is a way of approximating 
such texts, even if a given punctuation mark may assume a slightly different 
behavior in speech. A large number of punctuation marks can be considered in 
text: full stops, commas, exclamation mark, question mark, colon, semicolon and 
quotation marks. For our task one usually only considers full stops and commas, 
as they have higher corpus frequency. The other punctuation marks rarely occur, 
and are difficult to insert or evaluate. The capitalization task consists of 
rewriting every word with it's proper case depending. This is an opposite 
transform which text-to-speech systems are usually doing.

## Testing data

As any machine learning project, this project has been test-driven. Test have 
been made using a language model built on 95% of the gutenberg text database, 
on the rest of 5% of the texts.
The input file has to be lower-cased and with no punctuation.

## Implementation

This project is based on capitalized and punctuated language models. A similar 
implementation is the disambig tool from SRILM (which works only for 
capitalization).

The algorithm relies on iterating throught word symbols to create word 
sequences, which are evaluated and put into stacks. When a stack gets full (a 
maximum capacity is set) it gets sorted (by sequence probabilities) and the 
lowest scoring part is discarded. This way bad scoring sequences are discarded, 
and only the best ones are kept. The final solution is the sequence with the 
same size as the input, with the best probability.

## Language Model

For the post processing task the language model used has to contain capitalized 
words and punctuation mark word tokens. In the training data, commas are 
replaced with `<COMMA>` and periods are replaced with `<PERIOD>`. Also 
sentences should be grouped into paragraphs so that start and end of sentence 
markers (`<s>` and `</s>`) are not very frequent.  The language model need to 
be compressed from ARPA format to DMP format with sphinx_lm_convert (or 
sphinx3_lm_convert).

The gutenberg.DMP language model is correctly formatted and can be found in the 
language model download section on the project's sourceforge.

Example language model training data:

That will be in about fifteen minutes from now `<COMMA>` I figure `<COMMA>` 
murmured Frank Sheldon to his friend and comrade `<COMMA>` Bart Raymond 
`<COMMA>` as he glanced at the hands of his radio watch and then put it up to 
his ear to make sure that it had not stopped `<PERIOD>`  It'll seem more like 
fifteen hours `<COMMA>` muttered Tom Bradford `<COMMA>` who was on the other 
side of Sheldon `<PERIOD>`  Tom's in a hurry to get at the Huns `<COMMA>` 
chuckled Billy Waldon `<PERIOD>`  He wants to show them where they get off 
`<PERIOD>`  

## Usage

The project is available for download at 
https://cmusphinx.svn.sourceforge.net/svnroot/cmusphinx/branches/ppf.

To compile the project install ant and be sure to set the required enviroment 
variables.
Then type the following:

ant

To postprocess text use the postprocess.sh script:

sh ./postprocessing.sh -input_text path_to_file -lm path_to_lm

## Results

Results vary depending on the language model and the input text. Using the 
language model trained on the Gutenberg corpus on a few texts from the same 
project (test data was not used to train), the accuracy for commas prediction 
is 35% and for periods is 39%. Of course, ASR output is less strict and the 
readibility impact is pretty big.

This is the accuracy estimation for 3 big texts from the Gutenberg Project:

#### CAPITALIZATION

CORRECT: 494849

INCORRECT: 28328

#### COMMA

CORRECT:17142

EXTRA COMMA: 10049

MISSING FROM OUTPUT: 22181

#### PERIOD

CORRECT:13191

EXTRA PERIOD:9844

MISSING FROM OUTPUT: 10802

## References

 1.  [SENTENCE SEGMENTATION AND PUNCTUATION RECOVERY FOR SPOKEN LANGUAGE 
TRANSLATION](http://www.cs.cmu.edu/~ianlane/pub/PAULIK-icassp08.pdf)
 2.  
[[http://peer.ccsd.cnrs.fr/docs/00/49/92/19/PDF/PEER_stage2_10.1016%252Fj.specom
.2008.05.008.pdf|Recovering Capitalization and Punctuation
Marks for Automatic Speech Recognition]]
 1.  [RESTORING PUNCTUATION AND CAPITALIZATION IN TRANSCRIBED 
SPEECH](https://www.dropbox.com/s/2bwk2rf0xluziyj/RESTORING%20PUNCTUATION%20AND%
20CAPITALIZATION%20IN%20TRANSCRIBED%20SPEECH.pdf)
 2.  [Intro to Probability, Language 
Modeling](http://www.stanford.edu/class/cs224s/lec/224s.09.lec11.pdf)
 3.  [FSM 
Lecture](http://old-site.clsp.jhu.edu/ws04/calendar/School/FSMLecture.pdf)
 4.  [WFST Lecture - University of 
Tokio](http://www.gavo.t.u-tokyo.ac.jp/~novakj/wfst-algorithms.pdf)
 5.  [Probability 
estimation](http://www.uniroma2.it/didattica/WmIR/deposito/estimation_handout.pd
f)
