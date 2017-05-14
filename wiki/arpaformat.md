---
layout: page 
---
## ARPA Language models

Statistical language describe probabilities of the texts, they are trained on large corpora of text data. They can be stored in various text and binary format, but the common format supported by language modeling toolkits is a text format called ARPA format. This format fits well for interoperability between packages. It is not as efficient as most efficient binary formats though, so for production it is better to convert ARPA to binary.


### Statistical N-gram models in the ARPA format

ARPA language models are essentially "everything is possible" kind of models of the language. Given any sequence of N or less-that-N words, they provide a probability of that sequence being seen in a sufficiently large representative sample of that language.

Consider the text

	
	wood pittsburgh cindy jean
	jean wood


In this text we had a vocabulary of 4 words:
pittsburgh, cindy, wood, jean. The vocabulary size is actually
4+3 words, the 3 others being "sentence-begining(silence, but not
strictly in the acoustic sense)", sentence-ending(silence,but..), and
unknown-word. These are denoted in the sphinx community as `<s>`, `</s>`, and `<UNK>` respectively. These symbols are helpful for more consistent processing of the texts. You always start a sentence with `<s>` and extend it further.

The language model is a list of possible word sequences. Each sequence listed has its statistically estimated language probability tagged to it. It may or may not have a "backoff-weight" associated with it. In an N-gram LM, all N-1 grams usually have backoff weights associated with them.

If a particular N-gram is NOT listed, then its probability can be
calculated from the language model as follows:

	
	P( word_N | word_{N-1}, word_{N-2}, ...., word_1 ) =
	P( word_N | word_{N-1}, word_{N-2}, ...., word_2 ) * backoff-weight( word_{N-1} | word_{N-2}, ...., word_1 )


If the sequence ''( word_{N-1}, word_{N-2}, ...., word_1 )'' is also not listed, then the term ''backoff-weight( word_{N-1} | word_{N-2}, ...., word_1 )'' gets replaced with ''1.0'' and the recursion continues so. 

The following is a **random** example we constructed of a 2-gram LM with toy vocabulary. This is an example of a standard ARPA format LM. The format is simply ''P(N-gram sequence) sequence BP(N-gram sequence)''
These (the numbers associated with unigrams and bigrams in the example  below)  are actual probabilities. 
The format of "sequence" is ''A B C D := D'' after ''C'' after ''B'' after ''A'' (as spoken or written in the language)

So if you don't see the sequence "wood pittsburgh", you can get its probability by reading off 

''P(pittsburgh|wood)= P(pittsburgh) * BWt(wood).''

	
	
	\data\
	ngram 1=7
	ngram 2=7
	
	\1-grams:
	0.1 `<UNK>`	0.5555
	0 `<s>`	 0.4939
	0.1 `</s>`	 1.0
	0.2 wood	 0.5555
	0.2 cindy	0.5555
	0.2 pittsburgh		0.5555
	0.2 jean	 0.6349
	
	\2-grams:
	0.5555 `<UNK>` wood
	0.5555 `<s>` `<UNK>`
	0.5555 wood pittsburgh
	0.5555 cindy jean
	0.5555 pittsburgh cindy
	0.2778 jean `</s>`
	0.2778 jean wood 
	
	\end\
	
	


In most N-gram LMs, the actual probabilities are replaced by their
logs. Usually the logbase is 10. The LM below is thus the more
frequently used version of the example above:

	
	
	\data\
	ngram 1=7
	ngram 2=7
	
	\1-grams:
	-1.0000 `<UNK>`	-0.2553
	-98.9366 `<s>`	 -0.3064
	-1.0000 `</s>`	 0.0000
	-0.6990 wood	 -0.2553
	-0.6990 cindy	-0.2553
	-0.6990 pittsburgh		-0.2553
	-0.6990 jean	 -0.1973
	
	\2-grams:
	-0.2553 `<UNK>` wood
	-0.2553 `<s>` `<UNK>`
	-0.2553 wood pittsburgh
	-0.2553 cindy jean
	-0.2553 pittsburgh cindy
	-0.5563 jean `</s>`
	-0.5563 jean wood 
	
	\end\
	


The ARPA models could include 3-grams and even multigrams. 7-grams and 10-grams are rare but still used. ARPA models could be compressed with gzip to save space.

