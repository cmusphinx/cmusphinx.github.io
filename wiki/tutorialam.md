---
layout: page 
title: Training Acoustic Model For CMUSphinx
---
# Training Acoustic Model For CMUSphinx

## Introduction

CMUSphinx project comes with several high-quality acoustic models. There are US English acoustic
models for microphone and broadcast speech as well as a model for speech over a telephone. You can also use French or Chinese
models trained on a huge amount of acoustic data. Those models were carefully optimized to achieve
best recognition performance and work well for almost all applications. We spent years
of our experience to make them perfect. Most command-and-control apps could use them directly 
as well as large vocabulary applications. 

Besides models, CMUSphinx provides ways for adaptation which is sufficient for most cases when more
accuracy is required. Adaptation is known to work well when you are using different recording environments 
(close-distance or far microphone or telephone channel), or when a slightly different accent is present (UK English or even
Indian English) or even another language. Adaptation, for example, works well if you need to quickly
add support for some new language just by mapping acoustic model phoneset to target phoneset with
the dictionary.

There are, however, applications where the current models won't work. For example, handwriting recognition
or dictation support for another language. In these cases, you will need to train your own model and this tutorial will show you how to do that for the CMUSphinx speech recognition engine. Before starting with 
training make sure you are familar with concepts, prepared the language model and you indeed 
need to train the model and have resources to do that.

### When you need to train


*  You want to create an acoustic model for new language/dialect

*  OR you need specialized model for small vocabulary application

*  **AND you have plenty of data to train on**:
     * 1 hour of recording for command and control for single speaker
     * 5 hour of recordings of 200 speakers for command and control for many speakers
     * 10 hours of recordings for single speaker dictation
     * 50 hours of recordings of 200 speakers for many speakers dictation

*  AND you have knowledge on phonetic structure of the language

*  AND you have time to train the model and optimize parameters (1 month)

### When you don't need to train


*  You need to improve accuracy - do acoustic model adaptation instead

*  You don't have enough data - do acoustic model adaptation instead

*  You don't have enough time

*  You don't have enough experience

Please note that the amounts of data listed here **are required** to train model. If you have significantly less data than listed you can not expect to train a good model. For example, you **can not** train a model with 1 minute of speech data.

## Data preparation

The trainer learns the parameters of the models of the sound units using a set of sample speech signals.
This is called a training database. A choice of already trained databases will also be provided to you. 

The database contains information required to extract statistics from the speech in form of the acoustic model.

The trainer needs to be told which sound units you want it to learn the parameters of, 
and at least the sequence in which they occur in every speech signal in your training database. 
This information is provided to the trainer through a file called the *transcript file*, 
in which the sequence of words and non-speech sounds are written exactly as they occurred in a speech signal, followed by a tag which can be used to associate this sequence with 
the corresponding speech signal. 

The trainer then looks into a *dictionary* which maps 
every word to a sequence of sound units, to derive the sequence of sound units associated 
with each signal. 

Thus, in addition to the speech signals, you will also be given a set of
*transcripts* for the database (in a single file) and two dictionaries, one in which 
legitimate words in the language are mapped sequences of sound units (or sub-word units), 
and another in which non-speech sounds are mapped to corresponding non-speech or speech-like 
sound units. We will refer to the former as the language *dictionary* and the latter as the *filler dictionary*. 

After training, it's mandatory to run the decoder to check training results. The Decoder takes a
model, tests part of the database and reference transcriptions and estimates the quality (WER)
of the model. During the testing stage we use the *language model* with the description of the order of words in the language.

First of all, you need to design a database for training or download an existing one. For example, you can purchase a
database from LDC. You'll have to convert it to a proper format.

A database should be a good representation of what speech you are going to recognize. For example if you are going to recognize telephone speech its preferred to use telephone recordings. If you want mobile speech, you should better find mobile recordings. Speech is significantly different across various recording channels. Broadcast news is different from telephone. Speech decoded from mp3 is significantly different from the microphone recording. However, if you do not have enough speech recorded in required condition you should definitely use other speech you have. For example you can use broadcast recordings. Sometimes it make sense to stream through the telephone codec to make audio similar. It's often possible to add noise to training data too.

Database should have enough speakers recording, variety of recording conditions, enough acoustic variations and all possible linguistic sentences. The size of the database depends on the complexity of the task you want to handle as mentioned above. A Database should have the two parts mentioned above - training part and test part. Usually test part is about 1/10th of the full data size, but we don't recommend you to have test data more than 4 hours of recordings.

The good ways to obtain a database for a new language are:


*  Manually segment audio recordings with existing transcription (podcasts, news, etc)

*  Record your friends and family and colleagues

*  Setup automated collection on Voxforge

You have to design database prompts and postprocess the results to ensure that audio actually corresponds
to prompts. The file structure for the database is:


*  etc
    * your_db.dic - *Phonetic dictionary*
    * your_db.phone - *Phoneset file*
    * your_db.lm.DMP - *Language model*
    * your_db.filler - *List of fillers*
    * your_db_train.fileids - *List of files for training*
    * your_db_train.transcription - *Transcription for training*
    * your_db_test.fileids - *List of files for testing*
    * your_db_test.transcription - *Transcription for testing*

*  wav
    * speaker_1
      * file_1.wav - *Recording of speech utterance*
    * speaker_2
      *  file_2.wav

Let's go through the files and describe their format and the way to prepare them:

*Fileids (your_db_train.fileids and your_db_test.fileids) * file is a text file listing the names of the recordings (utterance ids) one by line, for example

	
	   speaker_1/file_1
	   speaker_2/file_2


Fileids file contains the path in a filesystem relative to wav directory. Note that fileids file should have no extensions for audio files, just the names.

*Transcription file (your_db_train.transcription and your_db_test.transcription) * is a text file listing the transcription for each audio file
   

	
	   `<s>` hello world `</s>` (file_1)
	   `<s>` foo bar `</s>` (file_2)


It's important that each line starts with `<s>` and ends with `</s>` followed by id in parentheses. Also note that parenthesis contains only the file, without speaker_n directory. It's critical to have exact match between fileids file and the transcription file. The number of lines in both should be identical. Last part of the file id ''(speaker1/**file_1**)'' and the utterance id ''file_1'' must be the same on each line.

Below is an example of **incorrect** fileids file for the above transcription file. If you follow it, you will get an error as discussed [here](https///sourceforge.net/p/cmusphinx/discussion/help/thread/278ee211/)

	
	   speaker_2/file_2
	   speaker_1/file_1
	   //Error! Do not create fileids file like this!



*Speech recordings (wav files) * 

Audio recordings should contain training audio and that training audio should match the audio you want to recognize. In case of mismatch there could be drop of the accuracy, sometimes significant. For example, if you want to recognize continuous speech your training database should record continuous speech. For continuous speech audio files shouldn't be very long and shouldn't be very short. Optimal length is not less than 5 seconds and not more than 30 seconds. Very long files make training much harder. If you are going to recognize short isolated commands, your training database should contain the files with short isolated commands. It is better to design database to recognize continuous speech from the beginning though and not spend your time on commands. In the end you speak continuously anyway. Amount of silence in the beginning of the utterance and in the end of the utterance should not exceed 0.2 second.

Recording files must be in MS WAV format with specific sample rate - 16 kHz, 16 bit, mono for desktop application, 8kHz, 16bit, mono for telephone applications.  It's **critical** to have audio files in a specific format. sphinxtrain does support some variety of sample rates but by default it's configured to train from **16khz 16bit mono** files in MS WAV format. **YOU NEED TO MAKE SURE THAT YOU RECORDINGS ARE AT A SAMPLING RATE OF 16 KHZ (or 8 kHz if you train a telephone model) IN MONO WITH SINGLE CHANNEL.**

If you train from 8khz model you need to make sure you configured feature extraction properly. Please note that you **CAN NOT UPSAMPLE** your audio, that means you can not train 16 khz model with 8khz data. 

Audio format mismatch is the most common training problem.

*Phonetic Dictionary (your_db.dict) * should have one line per word with word following the phonetic transcription
   

	
	HELLO HH AH L OW
	WORLD W AO R L D


If you need to find phonetic dictionary, read Wikipedia or a book on phonetics. If you are using existing 
phonetic dictionary. Do not use case-sensitive variants like "e" and "E". Instead, all your phones must be different even in case-insensitive variation. Sphinxtrain doesn't support some special characters like '*' or '/' and supports most of others like "+" or "-" or ":" But to be safe we recommend you to use alphanumeric-only phone-set. 

Replace special characters in the phone-set, like colons or dashes or tildes, with something alphanumeric. For example, replace "a~" with "aa" to make it alphanumeric only. Nowadays, even cell phones have gigabytes of memory on board. There is no sense in trying to save space with cryptic special characters.

There is one very important thing here. For a large vocabulary database, phonetic representation is more or less
known; it's simple phones described in any book. If you don't have a phonetic book, you can just use the word's spelling and
it gives very good results:

	
	ONE O N E
	TWO T W O


For small vocabulary CMUSphinx is different from other toolkits. It's often recommended to train word-based models
for small vocabulary databases like digits. But it only makes sense if your HMMs could have variable length. 

**CMUSphinx does not support word models.** Instead, you need to use a word-dependent phone dictionary:

	
	ONE W_ONE AH_ONE N_ONE
	TWO T_TWO UH_TWO
	NINE N_NINE AY_NINE N_END_NINE


This is actually **equivalent** to word-based models and some times even gives better accuracy. **Do not use 
word-based models with CMUSphinx.**

*Phoneset file (your_db.phone) * should have one phone per line. The number of phones should match the phones used in the dictionary plus the special SIL phone for silence:

	
	AH
	AX
	DH
	IX


*Language model file (your_db.lm.DMP) * should be in ARPA format or in DMP format. Find our more about language models on [Language Model training chapter](tutoriallm).

*Filler dictionary (your_db.filler) * contains filler phones (not-covered by language model non-linguistic sounds like breath, hmm or laugh). It can contain just silences:

	
	`<s>` SIL
	`</s>` SIL
	`<sil>` SIL


Or filler phones if they are present in the db transcriptions:

	
	+um+ ++um++
	+noise+ ++noise++


The sample database for training is available at [an4 database NIST's Sphere audio (.sph) format ](http://www.speech.cs.cmu.edu/databases/an4/an4_sphere.tar.gz), you can use it in following steps. If you want to play with large example, download TEDLIUM English acoustic database. It's about 200 hours of audio recordings now.
## Compilation of the required packages

The following packages are required for training:


*  sphinxbase-5prealpha

*  sphinxtrain-5prealpha

*  pocketsphinx-5prealpha

The following external packages are also required:


*  perl, for example ActivePerl on Windows

*  python, for example ActivePython on Windows

In addition, if you download packages with a ''.gz'' suffix, you will need ''gunzip'' or the equivalent to unpack them.

Install the perl and python packages somewhere in your executable path, if they are not already there.

We recommend that you train on Linux: this way you'll be able to use all the features of sphinxtrain. 
You can also use a Windows system for training, in that case we recommend to use ActivePerl.

For download instructions, see [Download page](http://cmusphinx.sourceforge.net/wiki/download/). 
Basically you need to put everything into single root folder, unzip and untar them, and run ''configure'' and ''make'' and ''make install'' in each package folder. Put the database folder into this root folder as well. By the time you finish this, you will have a tutorial directory with the following contents

	
	  tutorial
	    an4
	    an4_sphere.tar.gz
	    sphinxtrain
	    sphinxtrain-5prealpha.tar.gz
	    pocketsphinx
	    pocketsphinx-5prealpha.tar.gz
	    sphinxbase
	    sphinxbase-5prealpha.tar.gz



You will need to install software as an administrator ''root''. After you installed the software you may need to update the system configuration so the system will be able to find the dynamic libraries. For example

	
	export PATH=/usr/local/bin:$PATH
	export LD_LIBRARY_PATH=/usr/local/lib
	export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig


If you don't want to install into system path, you may install in your home folder. In that case you can append the following option to ''autogen.sh'' script or to the ''configure'' script

	
	--prefix=/home/user/local


Obvsiously the folder can be an arbitrary folder but remember to update the environment configuration after that. If you will find that your binaries fail to load dynamic libraries, something like ''failed to open libsphinx.so.0 no such file or directory'' it means that you didn't configure the environment properly.

## Setting up the training scripts

To start the training change to the database folder and run the following commands:

**On Linux**

	
	sphinxtrain -t an4 setup


**On Windows**

	
	python ../sphinxtrain/scripts/sphinxtrain -t an4 setup


Do not forget to replace an4 with your task name.

This will copy all the required configuration files into etc subfolder of your database folder and prepare database for training, the structure after setup will be:

	
	  etc
	  wav


In process of training other data folders will be created, the database should look like this:

	
	  etc
	  feat
	  logdir
	  model_parameters
	  model_architecture
	  result
	  wav


After setup, we need to edit the configuration files in etc folder, there are many variables but to get started we need
to change only a few. First of all find the file ''etc/sphinx_train.cfg''

### Setup the format of database audio

	
	$CFG_WAVFILES_DIR = "$CFG_BASE_DIR/wav";
	$CFG_WAVFILE_EXTENSION = 'sph';
	$CFG_WAVFILE_TYPE = 'nist'; # one of nist, mswav, raw


If you recorded MSWav, change ''sph'' to ''wav'' here and ''nist'' to ''mswav''.


	
	$CFG_WAVFILES_DIR = "$CFG_BASE_DIR/wav";
	$CFG_WAVFILE_EXTENSION = 'wav';
	$CFG_WAVFILE_TYPE = 'mswav'; # one of nist, mswav, raw

### Configure path to files

See the following lines in your etc/sphinx_train.cfg file:

	
	# Variables used in main training of models
	$CFG_DICTIONARY     = "$CFG_LIST_DIR/$CFG_DB_NAME.dic";
	$CFG_RAWPHONEFILE   = "$CFG_LIST_DIR/$CFG_DB_NAME.phone";
	$CFG_FILLERDICT     = "$CFG_LIST_DIR/$CFG_DB_NAME.filler";
	$CFG_LISTOFFILES    = "$CFG_LIST_DIR/${CFG_DB_NAME}_train.fileids";
	$CFG_TRANSCRIPTFILE = "$CFG_LIST_DIR/${CFG_DB_NAME}_train.transcription"


These values would be already set if you set up the file structure like described earlier, but make sure that your files are really named
this way. 

The $CFG_LIST_DIR variable is the /etc directory in your project, and the $CFG_DB_NAME variable is the name of your project itself. 

### Configure model type and model parameters

To select acoustic model type see [acousticmodeltypes](acousticmodeltypes)

	
	$CFG_HMM_TYPE = '.cont.'; # Sphinx4, Pocketsphinx
	#$CFG_HMM_TYPE  = '.semi.'; # PocketSphinx only
	#$CFG_HMM_TYPE  = '.ptm.'; # Sphinx4, Pocketsphinx, faster model


Just uncomment what you need. For resource-efficient applications use semi-continuous models, for sphinx4 
use continuous models

	
	 $CFG_FINAL_NUM_DENSITIES = 8;


If you are training continuous models for large vocabulary and have more than 100 hours of data, put 32 here. It can be any
degree of 2: 4, 8, 16, 32, 64.

If you are training semi-continuous or PTM model, use 256 gaussians.

	
	# Number of tied states (senones) to create in decision-tree clustering
	$CFG_N_TIED_STATES = 1000;


This value is the number of senones to train in a model. The more senones model has, the more precisely it discriminates the sounds. But on the other hand if you have too many senones, model will not be generic enough to recognize unseen speech. That means that the WER will be higher on unseen data. That's why it is **important** to not 
overtrain the models. In case there are too many unseen senones, the warnings will be generated in the norm log on stage 50 below:

	
	ERROR: "gauden.c", line 1700: Variance (mgau= 948, feat= 0, density=3, 
	component=38) is less then 0. Most probably the number of senones is too
	high for such a small training database. Use smaller $CFG_N_TIED_STATES.


The approximate number of senones and number of densities for continuous model is provided in the table below:

 | Vocabulary | Hours in db | Senones | Densities | Example                             | 
 | ---------- | ----------- | ------- | --------- | -------                             | 
 | 20         | 5           | 200     | 8         | Tidigits Digits Recognition         | 
 | 100        | 20          | 2000    | 8         | RM1 Command and Control             | 
 | 5000       | 30          | 4000    | 16        | WSJ1 5k  Small Dictation            | 
 | 20000      | 80          | 4000    | 32        | WSJ1 20k  Big Dictation             | 
 | 60000      | 200         | 6000    | 16        | HUB4  Broadcast News                | 
 | 60000      | 2000        | 12000   | 64        | Fisher Rich Telephone Transcription | 

For semi-continuous and PTM models use fixed number of 256 densities. 

Of course you also need to understand that only senones present in transcription could be trained. It means that if your transcription isn't generic enough, for example it's the same single word spoken by 10000 speakers 10000 times you still have just a few senones no matter how many hours of speech did you record. In that case you just need a few senones in the model, not few thousands of them.

It might seem that diversity could improve the model but it's not the case. Diverse speech requires some artificial speech prompts and that decrease the naturalness of the speech. Artificial models don't help in real life decoding. In order to build a best database you need to try to reproduce real environment as much as possible. It's even better to collect more speech to try to optimize the database size.

It's important to remember, that optimal numbers **depends on your database**. To train model properly, you need to experiment with different values and try to select the ones which give best WER for a development set. You can experiment with number of senones, number of gaussian mixtures at least. Sometimes it's also worth to experiment with phoneset or number of estimation iterations.

### Configure sound feature parameters

The default for sound files used in Sphinx is a rate of 16 thousand samples per second (16KHz). If this is the case, the etc/feat.params file will be automatically generated with the recommended values. 

If you are using sound files with a sampling rate of 8KHz (telephone audio), you need to change some values in etc/sphinx_train.cfg. The lower sampling rate also means a change in the sound frequency ranges used and the number of filters used to recognize speech. Recommended values are:

	
	# Feature extraction parameters
	$CFG_WAVFILE_SRATE = 8000.0;
	$CFG_NUM_FILT = 31; # For wideband speech it's 40, for telephone 8khz reasonable value is 31
	$CFG_LO_FILT = 200; # For telephone 8kHz speech value is 200
	$CFG_HI_FILT = 3500; # For telephone 8kHz speech value is 3500


### Configure parallel jobs to speedup the training

If you are on multicore machine or in PBS cluster you can run training in parallel, the following options should do the trick:

	
	# Queue::POSIX for multiple CPUs on a local machine
	# Queue::PBS to use a PBS/TORQUE queue
	$CFG_QUEUE_TYPE = "Queue";


Change type to "Queue::POSIX" to run on multicore. Then change number of parallel processes to run:

	
	# How many parts to run Forward-Backward estimatinon in
	$CFG_NPART = 1;
	$DEC_CFG_NPART = 1;             #  Define how many pieces to split decode in


If you are running on 8 core machine start around 10 parts to fully load the CPU during training.


### Configure decoding parameters

Open ''etc/sphinx_train.cfg'', make sure the following is properly configured:

	
	$DEC_CFG_DICTIONARY     = "$CFG_BASE_DIR/etc/$CFG_DB_NAME.dic";
	$DEC_CFG_FILLERDICT     = "$CFG_BASE_DIR/etc/$CFG_DB_NAME.filler";
	$DEC_CFG_LISTOFFILES    = "$CFG_BASE_DIR/etc/${CFG_DB_NAME}_test.fileids";
	$DEC_CFG_TRANSCRIPTFILE = "$CFG_BASE_DIR/etc/${CFG_DB_NAME}_test.transcription";
	$DEC_CFG_RESULT_DIR     = "$CFG_BASE_DIR/result";
	
	# These variables, used by the decoder, have to be user defined, and
	# may affect the decoder output
	
	$DEC_CFG_LANGUAGEMODEL  = "$CFG_BASE_DIR/etc/${CFG_DB_NAME}.lm.DMP";


If you are training with an4 please make sure that you changed ${CFG_DB_NAME}.lm.DMP to an4.ug.lm.DMP since the name of the language model is different in an4 database.

	
	$DEC_CFG_LANGUAGEMODEL  = "$CFG_BASE_DIR/etc/an4.ug.lm.DMP";


If everything is ok, you can proceed to training.
## Training

First of all, go to the database directory:

	
	cd an4


To train, just run the following commands:


**On Linux**

	
	sphinxtrain run


**On Windows**

	
	python ../sphinxtrain/scripts/sphinxtrain run


and it will go through all the required stages. It will take a few minutes to train. On large databases, training could take a month. 

During the stages, the most important stage is the first one which checks that everything is configured correctly and your input data is consistent. **Do not ignore the errors reported on the first 00.verify_all step.**

The typical output during decoding will look like:

	
	        Baum welch starting for 2 Gaussian(s), iteration: 3 (1 of 1)
	        0% 10% 20% 30% 40% 50% 60% 70% 80% 90% 100% 
	        Normalization for iteration: 3
	        Current Overall Likelihood Per Frame = 30.6558644286942
	        Convergence Ratio = 0.633864444461992
	        Baum welch starting for 2 Gaussian(s), iteration: 4 (1 of 1)
	        0% 10% 20% 30% 40% 50% 60% 70% 80% 90% 100% 
	        Normalization for iteration: 4


These scripts process all required steps to train the model. Training is complete.

## Training Internals

This section describes what happens during the training. In the scripts directory (''./scripts_pl''), there are several directories numbered sequentially from 00 through 99. Each directory either has a directory named ''slave*.pl'' or it has a single file with extension ''.pl''. The script sequentially goes through the directories and executes either the the ''slave*.pl'' or the single ''.pl'' file, as below. 

	
	perl scripts_pl/000.comp_feat/slave_feat.pl
	perl scripts_pl/00.verify/verify_all.pl
	perl scripts_pl/10.vector_quantize/slave.VQ.pl
	perl scripts_pl/20.ci_hmm/slave_convg.pl
	perl scripts_pl/30.cd_hmm_untied/slave_convg.pl
	perl scripts_pl/40.buildtrees/slave.treebuilder.pl
	perl scripts_pl/45.prunetree/slave-state-tying.pl
	perl scripts_pl/50.cd_hmm_tied/slave_convg.pl
	perl scripts_pl/90.deleted_interpolation/deleted_interpolation.pl


Scripts launch jobs on your machine, and the jobs will take a few minutes each to run through.
 
Before you run any script, note the directory contents of your current directory. 
After you run each slave*.pl note the contents again. Several new directories will have been created. 
These directories contain files which are being generated in the course of your training. 
At this point you need not know about the contents of these directories, though some of the 
directory names may be self explanatory and you may explore them if you are curious.

One of the files that appears in your current directory is an .html file, named an4.html, depending 
on which database you are using. This file will contain a status report of jobs already executed.
Verify that the job you launched completed successfully. Only then launch the next slave*.pl in the 
specified sequence. Repeat this process until you have run the slave*.pl in all directories.

Note that in the process of going through the scripts in 00* through 90*, you will have generated 
several sets of acoustic models, each of which could be used for recognition.
Notice also that some of the steps are required only for the creation of semi-continuous models.
If you execute these steps while creating continuous models, the scripts will benignly do nothing. 

On the stage ''000.comp_feat'' the feature feles are extracted. The system does not directly work with acoustic signals. The signals are first transformed into a  sequence of feature vectors, which are used in place of the actual acoustic signals.

This script slave_feat.pl will compute, for each training utterance, a sequence of 13-dimensional vectors (feature vectors) consisting of the Mel-frequency cepstral coefficients (MFCCs). Note that the  list of wave files contains a list with the full paths to the audio files. Since the data are all located in the same directory as you are working, the paths are relative, not absolute. You may have to change this, as well as the an4_test.fileids file,  if the location of data is different. The MFCCs will be placed automatically in a directory called 'feat'.  Note that the type of features vectors you compute from the speech signals for training and recognition, outside of this tutorial, is not restricted to MFCCs.  You could use any reasonable parameterization technique instead, and compute features 
other than MFCCs. CMUSphinx can use features of any type or dimensionality. The format of the features is described on the page [ MFC Format ](mfcformat ).

Once the jobs launched from ''20.ci_hmm'' have run to completion, you will have trained the Context-Independent 
(CI) models for the sub-word units in your dictionary. 

When the jobs launched from the ''30.cd_hmm_untied'' 
directory run to completion, you will have trained the models for Context-Dependent sub-word units 
(triphones) with untied states. These are called CD-untied models and are necessary for building decision 
trees in order to tie states. 

The jobs in ''40.buildtrees'' will build decision trees for each state of 
each sub-word unit. 

The jobs in ''45.prunetree'' will prune the decision trees and tie the states. 

Following this, the jobs in ''50.cd-hmm_tied'' will train the final models for the triphones in 
your training corpus. These are called CD-tied models. The CD-tied models are trained in many 
stages. We begin with 1 Gaussian per state HMMs, following which we train 2 Gaussian per state 
HMMs and so on till the desired number of Gaussians per State have been trained. 
The jobs in 50.cd-hmm_tied will automatically train all these intermediate CD-tied models.
 
At the end of any stage you may use the models for recognition. Remember that you
may decode even while the training is in progress, provided you are certain 
that you have crossed the stage which generates the models you want to decode with. 

### Transformation Matrix Training (advanced)

Some additional scripts will be launched if you choose to run them. These additional training steps can be costly in computation, but improve recognition rate.

Transformational matrices might help the training and recognition process in some circumstances.

The following steps will run if you specify ''$CFG_LDA_MLLT = 'yes';'' in the file ''sphinx_train.cfg''.
If you specify 'no', the default, the steps will do nothing. Step ''01.lda_train'' will estimate LDA matrix and
step ''02.mllt_train'' will estimate MLLT matrix.

The perl scripts, in turn, set up and run python modules.  The end product for these steps is a file, ''feature_transform'',  in your ''model_parameters'' directory. For details see [ldamllt](ldamllt)

### MMIE Training (advanced)

Finally, one more step will run if you specify MMIE training with ''$CFG_MMIE = "yes";''.
Default is ''"no"''. This will run steps ''60.lattice_generation'', ''61.lattice_pruning'', ''62.lattice_conversion'' and ''65.mmie_train''. For details see [mmie_train](mmie_train).
## Testing

It's **critical** to test the quality of the trained database in order to select best parameters, 
understand how application performs and optimize performance. To do that, a test decoding step is needed. 
The decoding is now a last stage of the training process.

You can restart decoding with the following command:

	
	sphinxtrain -s decode run


This command will start a decoding process using the acoustic model  you trained  and the language model you 
configured in the ''etc/sphinx_train.cfg'' file. 

	
	MODULE: DECODE Decoding using models previously trained
	        Decoding 130 segments starting at 0 (part 1 of 1) 
	        0% 


When the recognition job is complete, the script computes the recognition Word Error Rate (WER) and 
Sentence Error Rate (SER). The lower those rates the better for you. For typical 10-hours task WER
should be around 10%. For a large task, it could be like 30%.

On an4 data you should get something like:

	
	        SENTENCE ERROR: 70.8% (92/130)   WORD ERROR RATE: 30.3% (233/773)


You can find exact details of the decoding, like alignment with reference transcription, speed
 and result for each file, in ''result'' folder which will be created after decoding. Look into the file
''an4.align'':

	
	p   I   T      t   s   b   u   r   g   H      (MMXG-CEN5-MMXG-B)
	p   R   EIGHTY t   s   b   u   r   g   EIGHT  (MMXG-CEN5-MMXG-B)
	Words: 10 Correct: 7 Errors: 3 Percent correct = 70.00% Error = 30.00% Accuracy = 70.00%
	Insertions: 0 Deletions: 0 Substitutions: 3
	october twenty four nineteen seventy  (MMXG-CEN8-MMXG-B)
	october twenty four nineteen seventy  (MMXG-CEN8-MMXG-B)
	Words: 5 Correct: 5 Errors: 0 Percent correct = 100.00% Error = 0.00% Accuracy = 100.00%
	Insertions: 0 Deletions: 0 Substitutions: 0
	TOTAL Words: 773 Correct: 587 Errors: 234
	TOTAL Percent correct = 75.94% Error = 30.27% Accuracy = 69.73%
	TOTAL Insertions: 48 Deletions: 15 Substitutions: 171


For description of WER see our [Basic concepts of speech](tutorialconcepts) chapter.

## Using the model

After training, the acoustic model is located in 

	
	model_parameters/`<your_db_name>`.cd_cont_`<number_of senones>`


or in 

	
	model_parameters/`<your_db_name>`.cd_semi_`<number_of senones>`


You need only that folder. The model should have the following files:

	
	mdef
	feat.params
	mixture_weights
	means
	noisedict
	transition_matrices
	variances


Depending on the type of the model you trained. To use the model in pocketsphinx, simply point to it with the -hmm option:

	
	pocketsphinx_continuous -hmm `<your_new_model_folder>` -lm `<your_lm>` -dict `<your_dict>`.


To use the trained model in sphinx4, you need to specify the path in Configuration object

	
	     configuration.setAcousticModelPath("file:model_parameters/db.cd_cont_200");


If the model is in resources you can reference it with resource: URL

	
	     configuration.setAcousticModelPath("resource:/com/example/db.cd_cont_200");


See [Sphinx4 tutorial](tutorialsphinx4) for details.

## Troubleshooting

Troubleshooting is not rocket science. For all issues you may blame yourself. You are most 
likely the reason of failure. **Carefully** read the messages in the ''logdir'' folder that contains
detailed log of actions performed for each. In addition, messages are copied to the file, ''your_project_name.html'', which you can read in a browser.

There are many well-working proven methods to solve issues. For example, try to reduce the 
training set to see in which half the problem appears.

Here are some common problems:

##### WARNING: this phone (something) appears in the dictionary (dictionary file name), but not in the phone list (phone file name). 

 

Your dictionary either contains a mistake, or you have left out a phone symbol in the phone file.  You may have to delete comment lines from your dictionary file. 

##### WARNING: This word (word) has duplicate entries in (dictionary file name). Check for duplicates.

You may have to sort your dictionary file lines to find them. Perhaps a word is defined in both upper and lower case forms.

#####  WARNING: This word: word was in the transcript file, but is not in the dictionary (transcript line) Do cases match? 

Make sure that all the words in the transcript are in the dictionary, and that they match case when they appear. Also, words in the transcript may be misspelled, run together or be a number or symbol not in the dictionary.  If the dictionary file is not perfectly sorted, some entries might be skipped in looking for words. If you have hand-edited the dictionary file, be sure that each entry is in the proper format.

You may have specified phones in the phone list that are not represented in the words in the transcript. The trainer expects to find examples of each phone at least once.

##### WARNING: CTL file, audio file name.mfc, does not exist, or is empty.

The .mfc files are the feature files converted from the input audio files on stage ''000.comp_feats''. Did you skip this step? Did you add new audio files without converting them? The training process expects a feature file to be there, and it isn't.

##### Very low recognition accuracy.

This might happen if there is a mismatch in the audio files and the parameters of training, or between the training and the testing.

##### ERROR: "backward.c", line 430: Failed to align audio to transcript: final state of the search is not reached.

Sometimes audio in your database doesn't match the transcription properly. For example transcription file has the line "Hello world" but in audio actually "Hello hello world" is pronounced. Training process usually detects that and emits this message in the logs. If there are too many such errors it most likely mean you misconfigured something, for example you had a mismatch between audio and the text caused by transcription reordering. If there are few errors, you can ignore them. You might want to edit the transcription file to put there exact word which were pronounced, in the case above you need to edit the transcription file and put "Hello hello world" on corresponding line. You might want to filter such prompts because they affect acoustic model quality. In that case you need to enable forced alignment stage in training. To do that edit sphinx_train.cfg line 

	
	$CFG_FORCEDALIGN = 'yes';


and run training again. It will execute stages 10 and 11 and will filter your database.

##### Can't open */*-1-1.match word_align.pl failed with error code 65280

This error occurs because the decoder did not run properly after training. First check if the correct executable (**psdecode_batch** if the decoding script being used is ''psdecode.pl'' as set by ''$DEC_CFG_SCRIPT'' variable in ''sphinx_train.cfg'') is present in ''PATH''. On Linux run 

	
	which pocketsphinx_batch


and see if it is located. If it is not, you need to set the ''PATH'' variable properly. Similarly on Windows, run 

	
	where pocketsphinx_batch*


If the path to decoding executable is set properly, read the log files at ''logdir/decode/'' to find out other reasons behind the error.

##### To ask for help

If you want to ask for help about training, try to provide the training folder or at least logdir. Pack the files into archive, upload to a public file sharing resource, then post the link to the resource. Remember, the more information you provide the faster you will solve the problem.
