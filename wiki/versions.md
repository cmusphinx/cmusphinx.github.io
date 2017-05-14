---
layout: page 
---
# Versions Of Decoders

Over the years many different versions of decoders were created. They aren’t compatible as a code, consider 
them as independent branches. The decision about which version to use depends on how familiar you are with 
C/Python (pocketsphinx) or Java (sphinx4), and how easy it is to integrate these into your system. Currently
 we have the following choice for you:

**PocketSphinx**

PocketSphinx is CMU’s fastest speech recognition system. It’s a library written in pure C which is 
optimal for development of your C applications as well as for development of language bindings. At 
real time speed it’s the most accurate engine, and therefore it is a good choice for live applications.

It's good for desktop applications, command and control and dictation where fast response and low resource
consumption are the goals.

Also it includes support for embedded devices with fixed-point ariphmetics and is successfully used on 
IPhone, Nokia Maemon devices and on Windows Mobile. You can find further documentation about PocketSphinx 
in the release documentation, or at the online documentation.

**Sphinx-4**

Sphinx-4 is a state-of-the-art speech recognition system written entirely in the Java(tm) programming language. 
It's best for implementation of complex server or cloud-based system with deep interaction with NLP modules, web services and cloud computing. For further detail, please check the [ Sphinx-4 page](http://cmusphinx.sourceforge.net/sphinx4 ).

**Sphinx-3**

Sphinx-3 is CMU’s large vocabulary speech recognition system. It’s older C based decoder that we 
continue to maintain. It’s still most accurate decoder for the large vocabulary tasks. We are using it as a baseline to check the recognizer accuracy. This decoder is only intended for researchers who want to evaluate bleeding edge methods in ASR like tree search method.


**Obsolete decoders**

The following decoders are obsolete and not supported nowdays. We don’t recommend you to use them unless you 
know what you doing. They mostly have interest for speech recognition researchers and history specialists

Sphinx-2 is a fast speech recognition system, the predecessor of PocketSphinx. It is not being actively 
developed at this time, but is still widely used in interactive applications. It uses Hidden Markov Models 
(HMM) with semi-continuous output probability density functions (PDF). Even though it is not as accurate as
Sphinx-3 or Sphinx-4, it runs at real time, and therefore it is a good choice for live applications. You 
can find further documentation about Sphinx-2 in the release documentation, or at the online documentation.

** Other Decoders **

For competition with other decoders see [competition](competition)

