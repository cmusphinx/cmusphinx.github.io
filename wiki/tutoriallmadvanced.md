---
layout: page 
title: Large scale language model
---
## Building a large scale language model for domain-specific transcription

Language model describes the probabilities of the sequences of words in the 
text and is required for speech recognition. Generic models are very large 
(several gigabytes and thus impractical). Most recognition systems have models 
tuned to the specific domain. For example, medical language model describes 
medical dictation. If you are looking for your domain you most likely will have 
to build the language model for that domain yourself. You can mix that specific 
domain with generic domain to get some fallback, but specific domain is still 
needed. Generic language models are created from large texts.

The language modeling toolkit is <http://www.speech.sri.com/projects/srilm>, it
contains most tools for language modeling. In modern systems there is a lot of 
success with neural-network based RNNLM language models. Those are supported in 
rnnlm toolkit.

### Data collection

The first step of language model building is a collection of the data. The 
amount of data you need depends on the domain and vocabulary. Usually for a 
good model you need a significant amount of texts - at least 100mb. You can get 
this text by transcribing existing recordings, collecting data from the web, 
generating it artificially with scripts. The most valuable data is a real-life 
data anyway.

### Text cleanup

Once data is collected it must be cleaned - punctuation removed, sentences 
split on lines, numbers expanded to text representation. This can be done with 
scripting language

### Train/test separation

In machine learning it is practical to control the quality of the model you 
build. For that you need to separate test set from training set and use test 
set for evaluation. The practical split is 1 to 10.

The metric to use for evaluation is called perplexity. It controls the quality 
of the language model. The smaller perplexity is, the smaller is WER. You can 
calculate perplexity with SRILM:

       ngram -lm your.lm -ppl test.txt
       
Most other toolkits can also calculate perplexity.

Another parameter is OOV rate. SRILM also prints that with -ppl option. The OOV 
rate should not be large, the WER because of OOV is usually double the OOV 
rate. For example if you have 3% OOV rate you add extra 6% to WER. So you need 
to try to minimize OOV rate by extending your training set.

### Initial model estimation

Language model can be estimated with different parameters. The parameters 
depend on a task and require some understanding for underlying algorithms. For 
example, for book-like texts you need to use Knesser-Ney discounting. For 
command-like texts you should use Witten-Bell discounting or Absolute 
discounting. You can try different methods and see which gives better 
perplexity on a test set.

The command is

     ngram-count -kndiscount -text text.txt -lm text.lm

### Language model interpolation with generic model


To improve language model coverage you can mix LM with generic LM created from 
WEB texts. There are few of them:

[ CMUSphinx generic 
model](http://sourceforge.net/projects/cmusphinx/files/Acoustic%20and%20Language
%20Models/US%20English%20Generic%20Language%20Model/cmusphinx-5.0-en-us.lm.gz/do
wnload )

[ Language model from Cantab Research trained on gigaword 
corpus](http://cantabresearch.com/cantab-TEDLIUM.tar.bz2 )

To perform the mix you need to estimate mixture parameters. For that you can 
evaluate model on test set:

     ngram -lm your.lm -ppl test.txt -debug 2 > your.ppl
     ngram -lm generic.lm -ppl test.txt -debug 2 > generic.ppl
     compute-best-mix your.ppl generic.ppl
     ngram -lm your.lm -mix-lm generic.lm -lambda `<factor from above>` 
-write-lm mixed.lm

The mixed LM usually has slightly better perplexity than specific LM.

### Language model pruning

Most language models are large and it is impractical to use them in decoder. 
For example, you can not pack 1Gb LM into WFST decoder. For that reason you can 
prune them to reduce their size:

     ngram -lm mixed.lm -prune 1e-8 -write-lm mixed_pruned.lm
     
You can try different factors to get right model size. Usually model must be 
within 30mb. You perform decoding with pruned model instead of full model.

To compensate reduction of accuracy due to pruning you still can use original 
model in lattice rescoring mode to get the best accuracy. Most decoders can 
dump lattices and lattices can be rescoring with very large models. 

You can also use RNNLM in rescoring model to rescore n-best lists to get the 
best accuracy.

### Dictionary extension

You might need to extend the dictionary with the words from the language model 
because they might be missing. You can use G2P tool like phonetisaurus for 
that, however, remember that such tools are usually inaccurate. For that reason 
it is recommended to manually review G2P output and correct it if needed.

