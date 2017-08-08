---
layout: page 
title: Pocketsphinx Tutorial
---

* auto-gen TOC:
{:toc}

# Building application with pocketsphinx

## Installation

Pocketsphinx is a library that depends on another library called SphinxBase 
which provides common functionality 
across all CMUSphinx projects. To install Pocketsphinx, you need to install 
both Pocketsphinx and Sphinxbase. It's possible to use Pocketsphinx both in 
Linux, Windows, on MacOS, iPhone and Android.

First of all, download the released packages pocketsphinx and sphinxbase from 
project downloads, checkout them from subversion or github. For more details 
see [ download page](/wiki/download ).

Unpack them into same directory. On Windows, you will need to rename 
'sphinxbase-X.Y' (where X.Y is the SphinxBase version number) to simply
'sphinxbase' to satisfy project pocketsphinx configuration.

**THIS TUTORIAL DESCRIBES POCKETSPHINX 5PREALPHA, IT IS NOT GOING TO WORK ON 
OLDER VERSIONS
**

### Unix-like Installation

To build pocketsphinx in a unix-like environment (such as Linux, Solaris, 
FreeBSD etc) you need to make sure you have the following dependencies 
installed: gcc, automake, autoconf, libtool, bison, swig at least version 2.0, 
python development package, pulseaudio development package. If you want to 
build without dependencies you can use proper configure options like 
--without-swig-python but for beginner it is recommended to install all 
dependencies.

You need to download both sphinxbase and pocketsphinx packages and unpack them. 
Please note that you can not use sphinxbase and pocketsphinx of different 
version, please make sure that versions are in sync. After unpack you should 
see the following two main folders:

     sphinxbase-X.X
     pocketsphinx-X.x

On step one, build and install SphinxBase. Change current directory to 
`sphinxbase` folder. If you downloaded directly from the repository, you need 
to do this at least once to generate the `configure` file:

     % ./autogen.sh

if you downloaded the release version, or ran `autogen.sh` at least once, 
then compile and install:

     % ./configure
     % make
     % make install

The last step might require root permissions so it might be `sudo make 
install`. If you want to use fixed-point arithmetic, you must configure 
SphinxBase with the --enable-fixed option. You can also set installation prefix 
with `--prefix`. You can also configure with or without SWIG python support.

The sphinxbase will be installed in `/usr/local/` folder by default. Not 
every system loads libraries from this folder automatically. To load them you 
need to configure the path to look for shared libaries. It can be done either 
in the file `/etc/ld.so.conf` or with exporting environment variables:

     export LD_LIBRARY_PATH=/usr/local/lib
     export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig
     
For more details on linker configuration see [Shared Libraries 
HOWTO](http://tldp.org/HOWTO/Program-Library-HOWTO/shared-libraries.html).

Then change to pocketsphinx folder and perform the same steps

    % ./configure
    % make
    % make install

To test installation, run `'pocketsphinx_continuous -inmic yes`' and check 
that it recognizes words you are saying to the microphone.

If you get an error such as: `error while loading shared libraries: 
libpocketsphinx.so.3`, you may want to check your linker configuration with 
LD_LIBRARY_PATH environment variable described above.

### Windows

In MS Windows (TM), under MS Visual Studio 2010 (or newer - we test with Visual 
C++ 2010 Express):

*  load sphinxbase.sln located in sphinxbase directory
*  compile all the projects in SphinxBase (from `sphinxbase.sln`)
*  load `pocketsphinx.sln` in pocketsphinx directory
*  compile all the projects in PocketSphinx

MS Visual Studio will build the executables and libraries under 
`.\bin\Release` or `.\bin\Debug` (depending on the target you choose on MS 
Visual Studio). To run `pocketsphinx_continuous.exe`, don't forget to copy 
sphinxbase.dll to the bin folder. Otherwise the executable will fail to find 
this library. Unlike on Linux, the path to the model is not preconfigured in 
Windows, so you have to specify pocketsphinx_continuous where to find the model 
with -hmm, -lm and -dict options. Change to pocketsphinx folder and run

       bin\Release\Win32\pocketsphinx_continuous.exe -inmic yes -hmm 
model\en-us\en-us -lm model\en-us\en-us.lm.bin -dict 
model\en-us\cmudict-en-us.dict

to recognize from microphone. To recognize from file run

        bin\Release\Win32\pocketsphinx_continuous.exe -infile 
test\data\goforward.raw -hmm model\en-us\en-us -lm model\en-us\en-us.lm.bin 
-dict model\en-us\cmudict-en-us.dict



## Pocketsphinx API Core Ideas

Pocketsphinx API is designed to ease the use of speech recognizer functionality 
in your applications

 1.  It is much more likely to remain stable both in terms of source and binary 
compatibility, due to the use of abstract types.
 2.  It is fully re-entrant, so there is no problem having multiple decoders in 
the same process.
 3.  It has enabled a drastic reduction in code footprint and a modest but 
significant reduction in memory consumption.

Reference documentation for the new API is available at 
<https://cmusphinx.github.io/doc/pocketsphinx/>

## Basic Usage (hello world)

There are few key things you need to know on how to use the API:

 1.  Command-line parsing is done externally (in `<cmd_ln.h>`)
 2.  Everything takes a `ps_decoder_t *` as the first argument.

To illustrate the new API, we will step through a simple "hello world" example. 
 This example is somewhat specific to Unix in the locations of files and the 
compilation process.  We will create a C source file called `hello_ps.c`.  To 
compile it (on Unix), use this command:

	
	gcc -o hello_ps hello_ps.c \
	    -DMODELDIR=\"`pkg-config --variable=modeldir pocketsphinx`\" \
	    `pkg-config --cflags --libs pocketsphinx sphinxbase`


Please note that compilation errors here mean that you didn't carefully read 
the tutorial and didn't follow the installation guide above. For example 
pocketsphinx needs to be properly installed to be available through pkg-config 
system. To check that pocketsphinx is installed properly, just run `pkg-config 
--cflags --libs pocketsphinx sphinxbase` from the command line and see that 
output looks like

	
	-I/usr/local/include -I/usr/local/include/sphinxbase 
-I/usr/local/include/pocketsphinx  
	-L/usr/local/lib -lpocketsphinx -lsphinxbase -lsphinxad


### Initialization

The first thing we need to do is to create a configuration object, which for 
historical reasons is called `cmd_ln_t`.  Along with the general boilerplate 
for our C program, we will do it like this:

	
	#include `<pocketsphinx.h>`
	
	int
	main(int argc, char *argv[])
	{
	        ps_decoder_t *ps = NULL;
	        cmd_ln_t *config = NULL;
	
	        config = cmd_ln_init(NULL, ps_args(), TRUE,
	                 "-hmm", MODELDIR "/en-us/en-us",
	                 "-lm", MODELDIR "/en-us/en-us.lm.bin",
	                 "-dict", MODELDIR "/en-us/cmudict-en-us.dict",
	                 NULL);
	
	        return 0;
	}


The `cmd_ln_init()` function takes a variable number of null-terminated 
string arguments, followed by NULL.  The first argument is any previous 
`cmd_ln_t *` which is to be updated.  The second argument is an array of 
argument definitions - the standard set can be obtained by calling 
`ps_args()`.  The third argument is a flag telling the argument parser to be 
"strict" - if this is `TRUE`, then duplicate arguments or unknown arguments 
will cause parsing to fail.

The `MODELDIR` macro is defined on the GCC command-line by using 
`pkg-config` to obtain the `modeldir` variable from PocketSphinx 
configuration.  On Windows, you can simply add a preprocessor definition to the 
code, such as this:

	
	#define MODELDIR "c:/sphinx/model"


(replace this with wherever your models are installed).  Now, to initialize the 
decoder, use ps_init:

	
	        ps = ps_init(config);


### Decoding a file stream

Because live audio input is somewhat platform-specific, we will confine 
ourselves to decoding audio files.  The "turtle" language model recognizes a 
very simple "robot control" language, which recognizes phrases such as "go 
forward ten meters".  In fact, there is an audio file helpfully included in the 
PocketSphinx source code which contains this very sentence.  You can find it in 
`test/data/goforward.raw`.  Copy it to the current directory.  If you want to 
create your own version of it, it needs to be a single-channel (monaural), 
little-endian, unheadered 16-bit signed PCM audio file sampled at 16000 Hz.

Main pocketsphinx use case is to read audio data in blocks of memory from 
somewhere and feed them to the decoder. To do that we first open the file and 
start decoding of the utterance using `ps_start_utt()`:

	
	        rv = ps_start_utt(ps);


We will then read 512 samples at a time from the file, and feed them to the 
decoder using `ps_process_raw()`:

	
	        int16 buf[512];
	        while (!feof(fh)) {
	            size_t nsamp;
	            nsamp = fread(buf, 2, 512, fh);
	            ps_process_raw(ps, buf, nsamp, FALSE, FALSE);
	        }

Then we will need to mark the end of the utterance using `ps_end_utt()`:

	
	        rv = ps_end_utt(ps);


Then we retrieve the hypothesis to get recognition result

	
	        hyp = ps_get_hyp(ps, &score);
	        printf("Recognized: %s\n", hyp);


We can also retrieve the hypothesis during recognition, it will return partial 
result.

### Cleaning up

To clean up, simply call `ps_free()` on the object that was returned by 
`ps_init()`.  Free the configuration object with cmd_ln_free_r.

### Code listing

	
	#include `<pocketsphinx.h>`
	
	int
	main(int argc, char *argv[])
	{
	    ps_decoder_t *ps;
	    cmd_ln_t *config;
	    FILE *fh;
	    char const *hyp, *uttid;
	    int16 buf[512];
	    int rv;
	    int32 score;
	
	    config = cmd_ln_init(NULL, ps_args(), TRUE,
	                 "-hmm", MODELDIR "/en-us/en-us",
	                 "-lm", MODELDIR "/en-us/en-us.lm.bin",
	                 "-dict", MODELDIR "/en-us/cmudict-en-us.dict",
	                 NULL);
	    if (config == NULL) {
	        fprintf(stderr, "Failed to create config object, see log for 
details\n");
	        return -1;
	    }
	    
	    ps = ps_init(config);
	    if (ps == NULL) {
	        fprintf(stderr, "Failed to create recognizer, see log for 
details\n");
	        return -1;
	    }
	
	    fh = fopen("goforward.raw", "rb");
	    if (fh == NULL) {
	        fprintf(stderr, "Unable to open input file goforward.raw\n");
	        return -1;
	    }
	
	    rv = ps_start_utt(ps);
	    
	    while (!feof(fh)) {
	        size_t nsamp;
	        nsamp = fread(buf, 2, 512, fh);
	        rv = ps_process_raw(ps, buf, nsamp, FALSE, FALSE);
	    }
	    
	    rv = ps_end_utt(ps);
	    hyp = ps_get_hyp(ps, &score);
	    printf("Recognized: %s\n", hyp);
	
	    fclose(fh);
	    ps_free(ps);
	    cmd_ln_free_r(config);
	    
	    return 0;
	}
	


## Advanced Usage

For more complicated uses of the API please check the API reference.

 1.  For word segmentations, the API provides an iterator object which is used 
to, well, iterate over the sequence of words.  This iterator object is an 
abstract type, with some accessors provided to obtain timepoints, scores, and 
(most interestingly) posterior probabilities for each word.
 2.  Confidence of the whole utterance can be accessed with ps_get_prob method.
 3.  You can access lattice if needed
 4.  You can configure multiple searches and switch between them in runtime.

###  Searches 

Developer can configure several "search" objects with different grammars and 
language models and switch them in runtime to provide interactive experience 
for the user.

There are different possible search modes:
   - keyword - efficiently looks for keyphrase and ignores other speech. allows 
to configure detection threshold.
   - grammar - recognizes speech according to JSGF grammar. Unlike keyphrase 
grammar search doesn't ignore words which are not in grammar but tries to 
recognize them.
   - ngram/lm - recognizes natural speech with a language model.
   - allphone - recognizes phonemes with a phonetic language model.

Each search has a name and can be referenced by a name, names are 
application-specific. The function ps_set_search allows to activate the search 
previously added by a name. 

To add the search one needs to point to the grammar/language model describing 
the search. The location of the grammar is specific to the application. If only 
a simple recognition is required it is sufficient to add a single search or 
just configure the required mode with configuration options.

The exact design of a searches depends on your application. For example, you 
might want to listen for activation keyword first and once keyword is 
recognized switch to ngram search to recognize actual command. Once you 
recognized the command you can switch to grammar search to recognize the 
confirmation and then switch back to keyword listening mode to wait for another 
command.
