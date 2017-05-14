---
layout: page 
---
Standard adaptation methods are:


*  MLLR

*  MAP

*  FMLLR/CMLLR

*  FMAPLR

*  Eigenvectors

More advanced ones


*  Structured MAP

*  Sparse MAP

The recent research recommends to consider factored MLLR where specific transforms correspond to speaker, channel and environment

For semi-continuous models only MAP adaptation make sense. If one needs to adapt semi-continuous model with less parameters he need to consider some other methods like this one

[ CROSSLINGUAL ADAPTATION OF SEMI-CONTINUOUS HMMS USING ACOUSTIC SUB-SIMPLEX PROJECTION](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.74.7313&rep=rep1&type=pdf )
Frank Diehl, Asuncion Moreno, Enric Monte

Adaptation is good together with SAT training.

For MAP we can't estimate tau smoothing parameter reliably yet.

