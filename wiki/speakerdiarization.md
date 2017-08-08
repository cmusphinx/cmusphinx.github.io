---
layout: page 
---
## Segmentation and Diarization using LIUM tools

LIUM has released a free system for speaker diarization and segmentation, which 
integrates well with Sphinx.  This tool is essential if you are trying to do 
recognition on long audio files such as lectures or radio or TV shows, which 
may also potentially contain multiple speakers.

Segmentation means to split the audio into manageable, distinct chunks of 
homogeneous audio - e.g. speech, silence, music.  Diarization specifically 
means identifying the unique speakers in an audio file.  The LIUM tool does 
both of these - in fact it is not very useful to do one without the other.

### Getting Started

First, download the LIUM_spkDiarization .jar file from 
http://liumtools.univ-lemans.fr//index.php?option=com_content&task=blogcategory&
id=32&Itemid=60.  Running it without arguments will print out a summary of its 
options, which looks like this:

```
dhuggins@lima-2:~/Projects/PyCon/freelt$ ../tools/LIUM_SpkDiarization-3.1.jar 
info[info] 	 ====================================================== 
info[program] 	 name = Diarization
info[info] 	 ------------------------------------------------------ 
info[show] 	 [options] show
info[ParameterFeature-Input] 	 --fInputMask 	 Features input mask = %s.mfcc
info[ParameterFeature-Input] 	 --fInputDesc 	 Features info (type[,s:e:ds:de:dds:dde,dim,c:r:wSize:method]) = audio2sphinx,1:1:0:0:0:0,13,0:0:0:0
info[ParameterFeature-Input] 	 	 type [sphinx,spro4,gztxt,audio2sphinx] = audio2sphinx (4)
info[ParameterFeature-Input] 	 	 static [0=not present,1=present ,3=to be removed] = 1
info[ParameterFeature-Input] 	 	 energy [0,1,3] = 1
info[ParameterFeature-Input] 	 	 delta [0,1,2=computed on the fly,3] = 0
info[ParameterFeature-Input] 	 	 delta energy [0,1,2=computed on the fly,3] = 0
info[ParameterFeature-Input] 	 	 delta delta [0,1,2,3] = 0
info[ParameterFeature-Input] 	 	 delta delta energy [0,1,2,3] = 0
info[ParameterFeature-Input] 	 	 file dim = 13
info[ParameterFeature-Input] 	 	 normalization, center [0,1] = 0
info[ParameterFeature-Input] 	 	 normalization, reduce [0,1] = 0
info[ParameterFeature-Input] 	 	 normalization, window size = 0
info[ParameterFeature-Input] 	 	 normalization, method [0 (segment), 1 (cluster), 2 (sliding), 3 (warping)] =0
info[info] 	 ------------------------------------------------------ 
info[ParameterSegmentationFile-Input] 	 --sInputMask 	 Output segmentation mask = %s.in.seg
info[ParameterSegmentationFile-Input] 	 --sInputFormat 	 Output segmentation format = seg,ISO-8859-1
info[ParameterSegmentationFile-Output] 	 --sOutputMask 	 Output segmentation mask = %s.out.seg
info[ParameterSegmentationFile-Output] 	 --sOutputFormat 	 Output segmentation format = seg,ISO-8859-1
info[info] 	 ------------------------------------------------------ 
info[ParameterSystem] 	 --system = current
info[ParameterSystem] 	 --doCEClustering = false
info[ParameterSystem] 	 --saveAllStep = false
info[ParameterSystem] 	 --loadInputSegmentation = false
info[info] 	 ------------------------------------------------------ 
```

The LIUM segmentation tool can take a variety of file types as input.  By 
default, it is assumed that the input is audio - this can be a WAV file, 
probably also MP3 and others.  If you already have MFCCs and wish to use them 
instead, then pass %%--fInputDesc sphinx%% to it.  The input segmentation 
arguments (%%--sInputMask%% and %%--sInputFormat%%) are not required - if they 
are not given the tool will start with the entire file.

The default output format is the one used by LIUM in their evaluations.  
However, the tool can easily output the control files used by Sphinx3 and 
PocketSphinx (FIXME: and Sphinx4 too?), by passing %%--sOutputFormat ctl%%.  In 
theory it also supports Transcriber XML, but I haven't figured out how to make 
that work yet.

Here's a little script that will turn `ctl` format output files into label 
files for Audacity:

	
	#!/usr/bin/perl -w
	use strict;
	
	while (`<>`) {
	    chomp;
	    my ($show, $sf, $ef, $uttid) = split;
	    $sf /= 100;
	    $ef /= 100;
	    print "$sf\t$ef\t$uttid\n";
	}


And, here's one that will turn the output `hyp` files from PocketSphinx into 
label files, for transcription fixing:

	
	#!/usr/bin/perl -w
	use strict;
	
	while (`<>`) {
	    chomp;
	    my ($text, $uttid, $score) = /^(.*)\((\S+)(?:\s+(-?\d+))?\)$/;
	    my ($time, $start, $end) = split /-/, $uttid;
	    print "$start\t$end\t$text\n";
	}

