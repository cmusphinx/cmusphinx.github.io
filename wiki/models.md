---
layout: page 
---
# Models

#### Introduction

A complete speech recognition system will include data prepared using tools from outside sources,
as well as programs available from this site.

Minimally, such a system will have an acoustic model trainer and a decoder, using audio data, a 
dictionary, and a language model possibly created outside. This page gives you pointers to tools 
and data that will allow you to create a full speech recognition system. Keep in mind, though, 
that building a working system requires knowledge in speech processing that this site cannot provide.


*  Audio Data

*  Open Source Models

*  Dictionary

*  Language Model

*  Acoustic Model Trainer

*  Decoder

#### Audio data

Most of the reported results in speech recognition use data made available via the [ Linguistic Data Consortium (LDC)](http://ldc.upenn.edu/ ). 
There you will find audio/text data in several levels of complexity, but most of it is licensed, and you will 
need to pay for it.

CMU has made available the [ AN4 database]( http///www.speech.cs.cmu.edu/databases ), both in its original 
format and rerecorded through a microphone array. The database is publicly available. Note that it is a 
small database, which can be used to build a toy or test system, but which does not yield a system with 
high accuracy.

[ Voxforge]( http///voxforge.org ) builds a free acoustic database for many languages.

#### Open Source Models

If you prefer to skip the data preparation tools, you may retrieve acoustic models, language models,
 and dictionaries directly from the [ Open Source Models](http://www.speech.cs.cmu.edu/sphinx/models/ ) page. 
These models were trained from large databases, and may just work for your needs.

You will also find packages containing acoustic models in the
[ Sphinx release page]( http///sourceforge.net/project/showfiles.php?group_id=1904&amp;package_id=117949 ).

Finally, you can find models for the Spanish language at 
[ ITESM ]( http///speech.mty.itesm.mx/%7Ejnolazco/proyectos.htm ), in Mexico, with a mirror at 
[ CMU](http://www.speech.cs.cmu.edu/sphinx/models/hub4spanish_itesm/ )

Few prebuilt model for different languages are available at [ Voxforge](http://voxforge.org/ )

#### Dictionary

A dictionary is a file containing a mapping between words to be recognizer and its phonetic transcription. 
The phonetic transcription uses the phonetic unit used by the system. Most commonly, the system is designed
to use phonemes as the phonetic unit, but it is also common that the system is designed to use a word or 
even a whole phrase as the phonetic unit. 

CMU has made available the [ cmudict ]( http///www.speech.cs.cmu.edu/cgi-bin/cmudict ), which maps a large 
dictionary (100k+ words) to their phonemes.

#### Language Model

Language is commonly modeled through a statistical language models (SLM) or through the use of a finite
 state grammar (FSG). Sphinx-2, Sphinx3, and Sphinx-4 can handle both SLM and FSG. CMU provides tools 
for building statistical language models. FSGs have to be built by hand, or using tools not provided here. 

To build a language model, you can use an online [ LM tool ](http://www.speech.cs.cmu.edu/tools/lmtool.html ), or 
you can download and compile the CMU Statistical Language Model toolkit.

#### Acoustic Model Trainer

CMU provides an acoustic model trainer that can be used to produce continuous or semi-continuous HMMs. 
It produces models compatible with PocketSphinx and Sphinx-4. You have several 
options to retrieve [ SphinxTrain]( download )

#### Decoder

CMU offers several versions of the Sphinx decoder. You can check a quick 
comparision between the versions. You can check the [download](download) instructions.
