---
layout: page 
title: Web Data Collection for Language Modelling
---

## Project Details

There were various research projects for language model augmentation using web 
data that use search engines for obtaining the data[1][2][3][4]. While methods 
for generating queries and the nature of queries differ, the main approach is 
downloading a number of web pages from search results, parsing the downloaded 
pages and extracting text, and finally, applying a post-processing step, such 
as normalization, that will prepare the text for training.

This approach is efficient in terms of obtaining relevant text data as it uses 
advanced relevance matching features of search engines. But since automated 
queries put a strain on search engines, using robots or crawlers for making 
automated queries is forbidden in many search services.

Since search engines do not allow automated queries, we will need a focused web 
crawler that searches the web for documents containing a specific topic. There 
are existing FOSS focused crawlers such as 
[Combine](http://combine.it.lth.se/), 
[WebSPHINX](http://www.cs.cmu.edu/~rcm/websphinx/) and [Apache 
Nutch](http://nutch.apache.org/). Nutch is a good and mature crawler and will 
be used in this project.

For constructing and evaluating language models, we have decided to use [IRST 
LM Toolkit](http://sourceforge.net/projects/irstlm/). 
[Lucene](http://lucene.apache.org/core/) will be our search engine.

The idea is to construct domain-based language models from podcasts and web 
crawling. Given some audio and its transcripts belonging to some domain, we 
find additional text on the web which matches our domain. Then we build 
language models on this obtained text. Podcasts can have very different domains 
and also sometimes come with transcripts. The task therefore becomes building 
adaptive language models for podcasts. 

First, websites that contain high amounts of text such as news websites will be 
crawled. Then, the pages will be indexed by Lucene. We will identify key words 
in transcripts afterwards, retrieve results using different queries of those 
keywords in Lucene, use the articles to create a podcast specific language 
model interpolated with the background model and evaluated in terms of 
perplexity and WER.

In the end, we will have a configurable and easy to use Java tool that tries to 
encapsulate everything in one application, apart from installation of necessary 
components.

## Project plan and milestones

Full-time development for the project will start on the 6th of June.

The tool will be programmed in Java. I might make a user interface, depending 
on time left after the completion of required functionality. Progress will be 
reported weekly on CMU Sphinx website blog. I will extensively document the 
tool both for developers, in the form of detailed source code comments and for 
users, in the form of a manual and a tutorial.

The project is divided into a total of three milestones. Each unique task under 
a milestone is expected to take approximately half a week. 

### Milestone 1: Basic crawling

By the time we reach this milestone, I will have completed

    Crawl a website that allows crawling,
    Download the web page,
    Obtain text from it,
    Construct a language model from text using IRST LM.

This milestone is planned to be achieved in the bonding period.

### Milestone 2: Searching

Milestone 2 will be achieved when the tool can

    Crawl some websites with high amounts of text and index them,
    Take some text and extract (or take as input) N specific keywords that 
summarize the domain the most,
    Generate queries using those keywords,
    Construct domain-specific language models from resulting articles.
    
I will come up with a good method to extract strong keywords when I start 
working towards completing this milestone. A basic method that could possibly 
be used is tf*idf.

### Milestone 3: Tool development

When this milestone is achieved, the tool will be able to

    Wrap Nutch and Lucene for crawling and searching,
    Wrap IRST LM for language model construction,
    Construct domain-specific language models for Sphinx seamlessly.
    
Remaining time will be used to finish documentation, refactor code, add extra 
features and remove bugs.

## References

[1] Andreas Tsiartas, Panayiotis G. Georgiou, and Shrikanth S. Narayanan. 
“Language model adaptation using WWW documents obtained by utterance-based 
queries”. In: Proceedings of the International Conference on Acoustics, Speech, 
and Signal Processing (ICASSP). Dallas, TX, Mar. 2010.

[2] Mathias Creutz, Sami Virpioja, and Anna Kovaleva. “Web augmentation of 
language models for continuous speech recognition of SMS text messages”. In: 
Proceedings of the 12th Conference of the European Chapter of the Association 
for Computational Linguistics. EACL ’09. Athens, Greece: Association for 
Computational Linguistics, 2009, pp. 157–165. url: 
http://dl.acm.org/citation.cfm?id=1609067.1609084.

[3] Xiaojin Zhu and Ronald Rosenfeld. “Improving Trigram Language Modeling with 
the World Wide Web”. In: Acoustics, Speech, and Signal Processing, 2001. 
Proceedings.(ICASSP’01. 2000, pp. 533–536.

[4] Tim Ng et al. “Web-data Augmented Language Models For Mandarin 
Conversational Speech Recognition”. In: IEEE International Conference on 
Acoustics, Speech, and Signal Processing, 2005. Proceedings. (ICASSP ’05) 
(2005).
