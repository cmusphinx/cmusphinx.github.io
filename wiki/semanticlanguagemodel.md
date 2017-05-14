---
layout: page 
---
# Semantic Language model


## Abstract

Current language models in CMUSphinx is statistical langauge models,such as N-gram LMs, which are based purely on how frequentlythings happen, and not on what they mean. It means that they don't really understand what's transcribed. In this project, the language model will be extended with semantic information so as to improve the speech recognition performance. In particular, it will create a decoder over the lattices that will select semantically correct path and create a perfectly readable result.

## Related Work

About Speech Recognition: 

   * WhitePaper: "Sphinx-4: A flexible open source framework for speech recognition" [1] (It introduces the pluggable framework of Sphinx, how different modules connected with each other) (Very useful for code undertanding)
   * An introduction to speech recognition [10] (It introduces speech recognition from the frontend to the decoder algorithm, especially HMM model)
   
About Semantic Model:

   * Integrating word relationships into language models [2] (It introduces how word relationships and co-occurrence be integrated in language model)
   * Data-Driven Semantic Language Modeling [3] (it shows LSA for language modeling, and it does help speech recognition)
   * Introduction to latent semantic analysis [4] (Good tutorial for LSA using specific example)
   * Latent Semantic Analysis (LSA) Tutorial [5] (A good start for LAS)
   * Using semantic analysis to improve speech recognition performance [6]
   * Semantics in Speech Recognition and Understanding : A Survey [7] (It groups the semantic methods in SR into four approaches, but it seems not very useful because it is out of date.)
   * Incorporating semantic knowledge to the language model in a speech understanding system [8] (This paper shows semantic knowledge improves speech recognition accuracy by a Woz experiment, which means they manually add semantic information into sentence. However, it is far to be used in practice)
   * A semantics-enhanced language model for unsupervised word sense disambiguation [9] (It assumes that every words has senses, and the probability of a word is determined by both the previous word and it's sense. In this way, a semantice LM is derived and can be trained using WordNet knowledge and un-annotated train data)

## How to implement semantic language model?

1) Method 1: WordNet

--Mathematical model--

Using WordNet to get a probability estimator is shown in [2]. In particular, we want to get P(w_i|w), where w_i and w are assumed to have a relationship in WordNet.
 
The formular is below:
 P(w_i|w) = \frac{c(w_i,w|W,L)}{\sum_{w_j} c(w_j,w|W,L)}.
 
Where, W is a window size, c(w_i,w|W,L) is the count that w_i and w appearing together within W-window. It can be got just by counting in certain corpus.
 
To smooth the model, we can apply interpolated absolute discount or Kneser-Ney smoothing strategies.
 
What relationships can be considered:

 * synonym
 * hypernym
 * hyponym
 * hierarchical distance between words
 
--Java code to use WordNet--

see repository below: 
http://code.google.com/p/cmusphinx-gsoc2012/source/browse/#svn%2Ftrunk
 
2) Method 2: Co-Occurring

Using the co-occurrence is the same as WordNet shown in [2]. Here, we only need to replace c(w_i,w|W,L) with c(w_i,w|W), which means that w_i and w don't need to have a link in WordNet now.
 
3) Method 3: LSA (Latent Semantic Analysis)

LSA is shown to be helpful in speech recognition [3] and has been successfully used in many applications. Thus, I believe that it is promising for CMUSphinx project and should be tried.
 
--Mathematical model--

*The high level idea of LSA is to convert words into concept representations and it assumes that if the occurrence pattern of words in documents is similar then the words are similar.

a) To build the LAS model, a co-occurrence matrix W will be built first, where w_{ij} is a weighted count of word w_j and document d_j.
    w_{ij} = G_i L_{ij} C_{ij}
where, C_{ij} is the count of w_i in document d_j; L_{ij} is local weight; G_i is global weight. Usually, L_{ij} and G_i can use TF/IDF.

b) Then, SVD Analysis will be applied to W, then
     W = U S V^T
where, W is a M*N matrix, (M is the Vocabulary size, N is document size); U is M*R, S is R*R and V is a R*N matrix. R is usually a predefined dimension number between 100 and 500.

c) After that, each word w_i can be denoted as a new vector U_i = u_i*S

d) Based on this new vector, a distance between two words is defined:
   K(U_i, U_j) = \frac{u_i*S^2*u_m^T}{|u_i*S|*|u_m*S|}

e) Therefore, we can perform a clustering to words into K clusters, C_1, C_2, ...., C_K.

f) Let H_{q-1} be the history for word W_q, then we can get the probability of W_q given H_{q-1} by formula below:
   P(W_q|H_{q-1}) = P(W_q|W_{q-1},W_{q-2},...W_{q-n+1}, d_{q_1})
       = P(W_q|W_{q-1},W_{q-2},...W_{q-n+1})*P(W_q|d_{q_1}|)
 where, P(W_q|W_{q-1},W_{q-2},...W_{q-n+1}) is ngram model;
P(d_{q_1}|W_q) is the LSA model.

And, 
 P(W_q|d_{q_1}) = P(U_q|V_q) = K(U_q, V_{q_1})/Z(U,V)
 K(U_q, V_{q_1}) = \frac{U_q*S*V_{q-1}^T}{|U_q*S^{1/2}|*|V_{q-1}*S^{1/2}|}

 Z(U,V) is normalized factor.

g) We can apply word smoothing to the model based K-Clustering as follows:
P(W_q|d_{q_1}) = \sum_{k=1}^{K} P(W_q|C_k)P(C_k|d_{q_1})

where,  P(W_q|C_k), P(C_k|d_{q_1}) can be computer use the distance measurement given above  by a normalized factor.

h) In this way, N-gram and LSA model are combined into one language model.

--Python code for LSA--

Here is an implantation in python given by [5].
 
4) Method 4: FrameNet

FrameNet provides sementic roles for a word and shows restrictions how to use a word, which means that only several kinds of words can be followed by certain word. It might be a better way to use it in speech recognition.
 
We can use the model in [2] similarly to incorporate these relationships. We can also use the model introduced by [9], which assumes the probability of a word is determined by both the previous word and it's sense.
 
5) How to combine different models together?

Only if combined with semantic language model with traditional model like N-gram, we can have a better performance. In [9] or LSA [3], the semantic model itself is part of the language model, so it doesn't need to worry about how to combine them. However, in [2] , we need to combined them together. Here, EM algorithm can be used.
 
6) What codes are needed to change?

* Trainer
 a) For different model, we need to learn different parameters, so we need to  write a trainer.
 b) In addition, several methods about the semantic language model are needed: loader, saver, such as BinaryLoader. If the model is large, we also need to consider memory issue.
 c) What's more, the semantic language model can be trained with any text corpus, such as WSJ, so we don't need to worry about data sparsity problem.
 

* Recognizer (Take Sphinx4 for example)
In particular, we can implement a LanguageModel class, just like LargeTrigramModel, SimpleNGramModel.  The most important one is the function getProbability(WordSequence wordSequence) based on Semantic information introduced above. Here, the wordSequence will not need to be adjacent words, but can be any history words, concept or topic words.
 
In addition, since the lattice structure might be different, we also need to write a Linguist to manage the lattice, such as constructing, scoring.
 
- Pocketsphinx

It is similar as Sphinx4.
 
7) How to evaluate the model?
To test the performance under this new model, we can set up a Regression Test. 
Since this semantic model is used for large Vocabulary set, so we can use HUB4 as our test set. We can use Voxforge, too.
To measure the performance, Word Error Rate (WER) and other measurements can be used (see basic concepts in speech).
 
8) How to compile Sphinx4 in Eclipse?
- install Eclipse with plug-ins: subclipse (SVN support)
- install required software: Jave SE, Ant, subversion
- checkout Sphinx4 from SVN using Eclipse: File -> New Project -> SVN (Checkout Project from SVN) -> Create a new repository location (https://cmusphinx.svn.sourceforge.net/svnroot/cmusphinx/trunk/sphinx4) -> Finish
- Setup JSAPI 1.0, see [sphinx4/doc/jsapi_setup.html]
- setup build.xml, see http://www.lilwondermat.com/chatter/downloads/eclipse.pdf
- setup demo debug and test: Project -> Properties -> Jave Build Path -> Source -> sphinx4/src/apps (Double click Included: All) -> Add Multiple (select edu subfold) in Inclusion patterns

## Planning schedule

Before April 25
 

   * To familiarize myself to the CMUSphinx source code, including its main architecture, how to compile it, debug it, and alter it, especially for Pocketsphinx and Sphinx4. (Be quite familiar with the Sphinx4 code, but not  Pocketsphinx ) 
 

* Be familiar with Semantic language model in the cited paper, and how these word relationships are modeled. Then develop a mathematical model in ASR framework based on semantic language model.
 
 
April 25 – May 20 (Before official coding time)

* To doing some test coding stuff to further understand the usage of  Pocketsphinx and Sphinx4 .
 

* To discuss the details about my ideas with my future mentor to archive a final agreement. Based on the final agreement, I will be clear about all goals, such as which method and which semantic relationship will be used. Then I will see how to implement and do some documents work for those implementation details to further discuss with my mentor.
 
May 21 – June 17 (Official coding period starts)

* Start to design the framework with discussing with my mentor, including how to train the model, which dataset will be used, how to generate the lattice, how to score them, how to evaluate the model, and so on.
 

*  Begin Coding on Sphinx4: Word relationship extraction, parameters learning....
 
June 18 – July 1

* Fully Implement LSA or other model we decided in the beginning in Sphinx4
 
July 2 – July 13 (Mid-term)

* Finished the initial Regression test based on Sphinx4 to see whether the WER reduce or not
* Submit mid-term evaluation.
 
July 14 –  August  5

* Do some Error Analysis and consider to incorporated other semantics relationships, to see which ones are useful to improve the performance.
 
Augest 6 - August 16

* implement and test the model on Pocketsphinx  using C.
 
August 10 – August 24

* Redundant time for some unpredictable stuff to do.
* Submit final evaluation.

## References

[1] W. Walker, P. Lamere, P. Kwok, B. Raj, R. Singh, E. Gouvea, P. Wolf, J. Woelfel, Sphinx-4: A flexible open source framework for speech recognition, Tech. Rep., Sun Microsystems Inc., 2004

[2] G. Cao, J. Nie, J. Bai, Integrating word relationships into language models, Proceedings of the 28th annual international ACM SIGIR conference on Research and development in information retrieval, August 15-19, 2005

[3] J. R. Bellegarda, Data-Driven Semantic Language Modeling, Institute for Mathematics and Its Applications Workshop, 2000

[4] T.K. Landauer, P. W. Foltz, D. Laham, Introduction to latent semantic analysis. Discourse Processes, 25, 259–284, 1998

[5] Latent Semantic Analysis (LSA) Tutorial, http://www.puffinwarellc.com/index.php/news-and-articles/articles/33-latent-semantic-analysis-tutorial.html

[6] H. Erdogan, R. Sarikaya, S. F. Chen, Y. Gao, and M. Picheny,  Using semantic analysis to improve speech recognition performance,  Comput. Speech Language,  vol. 19,  pp.321 -343, 2005

[7] G. Demetriou, E. Atwell, Semantics in Speech Recognition and Understanding : A Survey, computational linguistics for speech and handwriting recognition, aisb'94 workshop, 1994

[8] S. Grau, E. Segarra, E. Sanchi´s, F. Garci´a, L.F. Hurtado, Incorporating semantic knowledge to the language model in a speech understanding system. In: IV Jornadas en Tecnologia del Habla, Zaragoza, Spain, pp 145–148, 2006

[9] S. Lin, K. Verspoor, A semantics-enhanced language model for unsupervised word sense disambiguation, Ninth International Conference on Computational Linguistics and Intelligent Text Processing (CICLing’08), Lecture Notes in Computer Science (LNCS), vol. 4919, pp. 287–298, 2008

[10] B. Plannerer, An introduction to speech recognition, Munich, Germany, 2005

