---
layout: page 
---
First of all make sure you are using fixed point operations and your dictionary 
and language models are small enough.

How do I make it fast?


The default settings are not enough to achieve sub-realtime performance on most 
tasks. Here are some command-line flags you should experiment with:

`-beam, -pbeam, -wbeam`

Main parameters to configure search width and thus accuracy-performance balance.

`-ds`

This is the dsratio. In most cases -ds 2 gives the best performance, though 
accuracy suffers a bit. (Frame GMM computation downsampling ratio) 
Thus lower should be better and higher should be less accurate.

`-topn`

The default value is 4, the fastest value is 2, but accuracy can suffer a bit 
depending on your acoustic model.

`-lpbeam`

This beam is quite important for performance, however the default setting is 
pretty narrow already.  Run pocketsphinx_batch with no arguments to see what it 
is.

`-lponlybeam`

Likewise here as with `-lpbeam`.  If you are finding it hard to get enough 
accuracy, you can widen these beams.

`-maxwpf`

This can be set quite low and still give you reasonable performance - try 5.

`-maxhmmpf`

Depending on the acoustic and language model this can be very helpful.  Try 
3000.

`-pl_window`

Phonetic lookahead is a specific technique which is used to speedup decoding by 
reducing the amount of computation. Basically everything is decoded with 
phonetic decoder first and then detailed search is restricted by the results of 
the fast phonetic search. It's also called "Fast match". For details and 
evaluations see the chapter "4.5 Phonetic Fast Match" in 
[Efficient Algorithms for Speech Recognition Mosur K. Ravishankar](
http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.72.3560)

pl_window specifies lookahead distance in frames. Typical values are from 0 
(don't use lookahead) to 10 (decode 10 frames ahead). Bigger values give faster 
decoding but reduced accuracy.
