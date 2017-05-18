---
layout: page 
title: Sphinx4 tutorial
---
# Sphinx-4 Application Programmer's Guide

WARNING: THIS TUTORIAL DESCRIBES SPHINX4 API FROM THE PRE-ALPHA RELEASE:

https://sourceforge.net/projects/cmusphinx/files/sphinx4/5prealpha/

**The API described here is not supported in earlier versions**

## Overview

Sphinx4 is a pure Java speech recognition library. It provides a quick and easy 
API to convert the speech recordings into text with the help CMUSphinx acoustic 
models. It can be used on servers and in desktop applications. Beside speech 
recognition Sphinx4 helps to identify speakers, adapt models, align existing 
transcription to audio for timestamping and more.

Sphinx4 supports US English and many other languages.

## Using in your projects

As any library in Java all you need to do to use sphinx4 is to add jars into 
dependencies of your project and then you can write code using the API.

The easiest way to use modern sphinx4 is to use modern build tools like [ 
Apache Maven](http://maven.apache.org/ref/3.1.0/ ) or [ 
Gradle](http://gradle.org). Sphinx-4 is available as a maven package in [ 
Sonatype OSS repository](https://oss.sonatype.org/ ). To use sphinx4 in your 
maven project specify this repository in your `pom.xml`:

	:::xml
	`<project>`
	...
	    `<repositories>`
	        `<repository>`
	            `<id>`snapshots-repo`</id>`
	            
`<url>`https://oss.sonatype.org/content/repositories/snapshots`</url>`
	            `<releases>``<enabled>`false`</enabled>``</releases>`
	        `<snapshots>``<enabled>`true`</enabled>``</snapshots>`
	        `</repository>`
	    `</repositories>`
	...
	`</project>`


Then add `sphinx4-core` to the project dependencies:

	:::xml
	`<dependency>`
	  `<groupId>`edu.cmu.sphinx`</groupId>`
	  `<artifactId>`sphinx4-core`</artifactId>`
	  `<version>`5prealpha-SNAPSHOT`</version>`
	`</dependency>`


Add `sphinx4-data` to dependencies as well if you want to use default 
acoustic and language models:

	:::xml
	`<dependency>`
	  `<groupId>`edu.cmu.sphinx`</groupId>`
	  `<artifactId>`sphinx4-data`</artifactId>`
	  `<version>`5prealpha-SNAPSHOT`</version>`
	`</dependency>`


In gradle you need to following lines in `build.gradle`

	:::javascript
	repositories {
	    mavenLocal()
	    maven { url 
"https://oss.sonatype.org/content/repositories/snapshots" }
	}
	
	dependencies {
	    compile group: 'edu.cmu.sphinx', name: 'sphinx4-core', 
version:'5prealpha-SNAPSHOT'
	    compile group: 'edu.cmu.sphinx', name: 'sphinx4-data', 
version:'5prealpha-SNAPSHOT'
	}


Many IDEs like Eclipse or Netbeans or Idea have support for Gradle either with 
plugin or with built-in features. In that case you can just include sphinx4 
libraries into your project with the help of IDE. Please check the relevant 
part of your IDE documentation, for example [ IDEA documentation on 
Gradle](https://www.jetbrains.com/idea/help/gradle.html ).

You can also use Sphinx4 in non-maven project, in that case you need to 
download jars from the [ repository](https://oss.sonatype.org/#nexus-search ) 
manually together with dependencies (which we try to keep small) and include 
them into your project. You need sphinx4-core jar and sphinx4-data jar if you 
are going to use US English acoustic model. See below

{{:sphinx4-download.png|Sphinx4 jar download}}

Here is for example how to include jars into eclipse:

{{http://i.stack.imgur.com/A6xgq.png|Include Jar into Eclipse project}}

## Basic Usage

To quickly start with sphinx4, create a java project as described above, add 
required dependencies and type the following simple code:

	
	package com.example;
	
	import java.io.File;
	import java.io.FileInputStream;
	import java.io.InputStream;
	
	import edu.cmu.sphinx.api.Configuration;
	import edu.cmu.sphinx.api.SpeechResult;
	import edu.cmu.sphinx.api.StreamSpeechRecognizer;
	
	public class TranscriberDemo {       
	                                     
	    public static void main(String[] args) throws Exception {
	                                     
	        Configuration configuration = new Configuration();
	
	        configuration
	                
.setAcousticModelPath("resource:/edu/cmu/sphinx/models/en-us/en-us");
	        configuration
	                
.setDictionaryPath("resource:/edu/cmu/sphinx/models/en-us/cmudict-en-us.dict");
	        configuration
	                
.setLanguageModelPath("resource:/edu/cmu/sphinx/models/en-us/en-us.lm.bin");
	
	        StreamSpeechRecognizer recognizer = new StreamSpeechRecognizer(
	                configuration);
	        InputStream stream = new FileInputStream(new File("test.wav"));
	
	        recognizer.startRecognition(stream);
	        SpeechResult result;
	        while ((result = recognizer.getResult()) != null) {
	            System.out.format("Hypothesis: %s\n", 
result.getHypothesis());
	        }
	        recognizer.stopRecognition();
	    }
	}


This simple code transcribes file test.wav, just make sure it exists in the 
project root.

There are several high-level recognition interfaces in Sphinx-4:

*  LiveSpeechRecognizer
*  StreamSpeechRecognizer
*  SpeechAligner

For the most of the speech recognition jobs high-levels interfaces should be 
enough. And basically you will have only to setup four attributes:

*  Acoustic model.
*  Dictionary.
*  Grammar/Language model.
*  Source of speech.

First three attributes are setup using Configuration object which is passed 
then to a recognizer. The way to point out to the speech source depends on a 
concrete recognizer and usually is passed as a method parameter.

### Configuration

Configuration is used to supply required and optional attributes to recognizer.

	:::java
	Configuration configuration = new Configuration();
	
	// Set path to acoustic model.
	
configuration.setAcousticModelPath("resource:/edu/cmu/sphinx/models/en-us/en-us"
);
	// Set path to dictionary.
	
configuration.setDictionaryPath("resource:/edu/cmu/sphinx/models/en-us/cmudict-e
n-us.dict");
	// Set language model.
	
configuration.setLanguageModelPath("resource:/edu/cmu/sphinx/models/en-us/en-us.
lm.bin");


### LiveSpeechRecognizer

LiveSpeechRecognizer uses microphone as the speech source.

	:::java
	LiveSpeechRecognizer recognizer = new 
LiveSpeechRecognizer(configuration);
	// Start recognition process pruning previously cached data.
	recognizer.startRecognition(true);
	SpeechResult result = recognizer.getResult();
	// Pause recognition process. It can be resumed then with 
startRecognition(false).
	recognizer.stopRecognition();


### StreamSpeechRecognizer

StreamSpeechRecognizer uses InputStream as the speech source, you can pass the 
data from the file this way, you can pass the data from the network socket or 
from existing byte array.

	:::java
	StreamSpeechRecognizer recognizer = new 
StreamSpeechRecognizer(configuration);
	recognizer.startRecognition(new FileInputStream("speech.wav"));
	SpeechResult result = recognizer.getResult();
	recognizer.stopRecognition();



Please note that the audio for this decoding must have one of the two specific 
format:

      RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, mono 16000 
Hz

or

      RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, mono 8000 Hz

Decoder does not support other formats. If audio format does not match, you 
will not get any results. You need to convert audio to a proper format before 
decoding. If you want to decode telephone quality audio with the sample rate 
8000 Hz, you also need to call

     configuration.setSampleRate(8000);

You can retreive multiple results until the file end:

	
	while ((result = recognizer.getResult()) != null) {
	    System.out.println(result.getHypothesis());
	}

### SpeechAligner

SpeechAligner time-aligns text with audio speech.

	:::java
	SpeechAligner aligner = new SpeechAligner(configuration);
	recognizer.align(new URL("101-42.wav"), "one oh one four two");


### SpeechResult

SpeechResult provides access to various parts of the recognition result, such 
as recognized utterance, list of words with time stamps, recognition lattice 
and so forth.

	:::java
	// Print utterance string without filler words.
	System.out.println(result.getHypothesis());
	
	// Get individual words and their times.
	for (WordResult r : result.getWords()) {
	    System.out.println(r);
	}
	
	// Save lattice in a graphviz format.
	result.getLattice().dumpDot("lattice.dot", "lattice");


## Demos

A number of sample demos are included in sphinx4 sources in order to give you 
understanding how to run Sphinx4. You can run them from sphinx4-samples jar:

   - [ 
Transcriber](https://github.com/cmusphinx/sphinx4/blob/master/sphinx4-samples/sr
c/main/java/edu/cmu/sphinx/demo/transcriber/TranscriberDemo.java ) - 
demonstrates how to transcribe a file
   - [ 
Dialog](https://github.com/cmusphinx/sphinx4/blob/master/sphinx4-samples/src/mai
n/java/edu/cmu/sphinx/demo/dialog/DialogDemo.java ) - demonstrates how to lead 
dialog with a user
   - [ 
SpeakerID](https://github.com/cmusphinx/sphinx4/blob/master/sphinx4-samples/src/
main/java/edu/cmu/sphinx/demo/speakerid/SpeakerIdentificationDemo.java) - 
speaker identification
   - [ 
Aligner](https://github.com/cmusphinx/sphinx4/blob/master/sphinx4-samples/src/ma
in/java/edu/cmu/sphinx/demo/aligner/AlignerDemo.java ) - demonstration of audio 
to transcription timestamping

If you are going to start with a demo please do not modify the demo inside 
sphinx4 sources, instead copy the code into your project and modify it there.


## Building from source

If you want to develop Sphinx4 itself you might want to build it from source. 
Sphinx4 uses [Gradle](http://gradle.org) build system. Simply type 'gradle 
build' in the top folder, it will compile and install everything including 
dependencies.

If you are going to use IDE, make sure it supports Gradle projects, then simply 
import sphinx4 source tree.

## Troubleshooting

You might meet different problems while using sphinx4, please check the 
[FAQ](faq) first before asking them on the forum.

In case you have issues with accuracy you need to provide audio recording you 
are trying to recognize, all the models you use and describes how results you 
see are different from your expectation.

 
