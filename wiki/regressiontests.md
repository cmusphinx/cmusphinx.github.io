---
layout: page 
title: Performance Regression Tests
---

This page describes the available regression tests, how to set them up if you 
have the data, and the length of time to run each test. The page also describes 
the current weekly schedule of regression tests for Sphinx 4. The schedule was 
based upon the times of the tests and the time constraints in which the tests 
must be run.

These tests were designed to verify performance regression. They run 
automatically in machines located at Carnegie Mellon. The tests use data 
released by the [LDC](http://www.ldc.upenn.edu/). The advantage is that the 
data are well known in the speech community. The disadvantage is that the data 
are licensed, and not everyone has access to them.

There are plans to create unit regression tests that could be run by developers 
just before checking in code. These would run quickly, providing a fast test 
that things did not break. They would use open source data also, so anyone 
could run the tests.

## Overview

The regression test main script does a fresh download of the code from the 
Sphinx-4 repository (currently, a svn repository at http://sourceforge.net). 
The script runs tests, and raw result numbers are stored at a cvs repository at 
sourceforge.net. The script also creates HTML reports (//cf.// tests running on 
[filbert](http://cmusphinx.github.io/regression_report_filbert.html)) and 
sends email reports to the cmusphinx-results mailing list. Check the main 
[mailing list](http://sourceforge.net/mail/?group_id=1904) page for the archive 
or to subscribe/unsubscribe.

## Installing the Tests


### Required software

The tests run automatically as a cron job. Therefore, the system that runs the 
tests needs to have the following easily available (//e.g.// in the system 
path):


*  cron

*  svn

*  cvs

*  rsync

*  bash

*  awk

*  javac, version 1.6 at least

### Storing results

The test results are stored in files kept in a CVS repository at 
sourceforge.net. They are kept in CVS rather than SVN to avoid sending a 
"commit" message every time the regression test scripts update the results. 
There are steps that have to be done manually.

First, get the CVS data.

	
	env CVS_RSH=ssh cvs -z3 
-d:ext:USERNAME@cmusphinx.cvs.sourceforge.net:/cvsroot/cmusphinx checkout 
regressionResults


About once a year, clean the main `regression.log` file from old results. For 
example, for year 2010, you do the following.

`<file bash cleanup.sh>`
grep '|2010-' regression.log > regression.2010.log
grep -v '|2010-' regression.log > regression.temp
cat regression.header regression.system regression.temp > regression.log
rm regression.temp
cvs add regression.2010.log
cvs co -m "update files" 
`</file>`

If the machine you are using for tests is not already in the `regression.log` 
file, you will have to update both `regression.log` and `regression.system` 
(or only the latter if it is time for the annual cleanup). You will have to add 
a line contaning, in this order, as detailed in `regression.header`:

 1.  the string "system" literally
 2.  machine name
 3.  number of CPUS
 4.  cacheSize (in kbytes)
 5.  clock (MHz)
 6.  memory (Mbytes)
 7.  architecture
 8.  OS

For example, this line was added for the machine `filbert`:

	
	system|filbert|8|4096|2660|15904|x86_64|Linux|


In Linux, the information about CPU speed, memory, etc can be found in either 
`/proc/meminfo` or `/proc/cpuinfo`.

### Data

The tests assume that the data (audio, acoustic models) used are available 
under `/lab`, and the environment variable `$SF_ROOT` points to the root of 
a working copy of the sphinx4 code. 

At CMU, the data are available from the `robust` account at `~robust/lab`. 
Create the link:
    ln -s ~robust/lab /lab

### Final steps

Create the variable `SF_ROOT` pointing to the working copy of the repository. 
If the Sphinx-4 working copy is located at `~/SourceForge`, add this to your 
`~/.profile` file, or create it if it does not exist:
    export SF_ROOT=${HOME}/SourceForge

With these in place, install the crontab below. Beware that cron uses `bash` 
regardless of your choice of shell.
    crontab regression_crontab

`<file bash regression_crontab>`
MAILTO=cmusphinx-results@lists.sourceforge.net
50 18 * * * (. $HOME/.profile ; cd $SF_ROOT/sphinx4/tests/regression; svn -q up 
.; ./regressionTest nightly batch)
35 23 * * 0 (. $HOME/.profile ; cd $SF_ROOT/sphinx4/tests/regression; 
./regressionTest sunday batch)
35 23 * * 1 (. $HOME/.profile ; cd $SF_ROOT/sphinx4/tests/regression; 
./regressionTest monday batch)
35 23 * * 2 (. $HOME/.profile ; cd $SF_ROOT/sphinx4/tests/regression; 
./regressionTest tuesday batch)
35 23 * * 3 (. $HOME/.profile ; cd $SF_ROOT/sphinx4/tests/regression; 
./regressionTest wednesday batch)
35 23 * * 4 (. $HOME/.profile ; cd $SF_ROOT/sphinx4/tests/regression; 
./regressionTest thursday batch)
35 23 * * 5 (. $HOME/.profile ; cd $SF_ROOT/sphinx4/tests/regression; 
./regressionTest friday batch)
35 23 * * 6 (. $HOME/.profile ; cd $SF_ROOT/sphinx4/tests/regression; 
./regressionTest saturday batch)
50 23 * * 0 (. $HOME/.profile ; cd $SF_ROOT/sphinx4/tests/regression; 
./regressionTest async0 batch)
50 23 * * 1 (. $HOME/.profile ; cd $SF_ROOT/sphinx4/tests/regression; 
./regressionTest async1 batch)
50 23 * * 2 (. $HOME/.profile ; cd $SF_ROOT/sphinx4/tests/regression; 
./regressionTest async2 batch)
50 23 * * 3 (. $HOME/.profile ; cd $SF_ROOT/sphinx4/tests/regression; 
./regressionTest async3 batch)
50 23 * * 4 (. $HOME/.profile ; cd $SF_ROOT/sphinx4/tests/regression; 
./regressionTest async4 batch)
50 23 * * 5 (. $HOME/.profile ; cd $SF_ROOT/sphinx4/tests/regression; 
./regressionTest async5 batch)
50 23 * * 6 (. $HOME/.profile ; cd $SF_ROOT/sphinx4/tests/regression; 
./regressionTest async6 batch)
30 17 * * * (. $HOME/.profile ; cd $SF_ROOT/sphinx4/scripts; svn -q up .; 
./updateS4Javadocs.sh)
05 02 * * * (. $HOME/.profile ; cd $SF_ROOT/web; svn -q up .; 
$SF_ROOT/web/script/nightlyBuild.sh)
05 06 * * * (. $HOME/.profile ; cd $SF_ROOT/web; svn -q up .; 
$SF_ROOT/web/script/update_sf.sh)
00 03 * * * (. $HOME/.profile ; cd $SF_ROOT/web; svn -q up .; 
$SF_ROOT/web/script/sfbackup.sh)
`</file>`

# Regression Test Times

 

This chart shows the available tests and the approximate time to run each test.


 |                  | **Word List** | **flat unigram** | **unigram** | 
**bigram** | **trigram** | **flat unigram fst** | **unigram fst** | **bigram 
fst** | **trigram fst** | **Acoustic Model** | 
 |                  | ------------- | ---------------- | ----------- | 
---------- | ----------- | -------------------- | --------------- | 
-------------- | --------------- | ------------------ | 
 | **ti46**         | 0:10          | 0:15             |             |          
  |             | 0:10                 |                 |                |     
            | tidigits           | 
 | **tidigits**     | 1:00          | 1:00             |             |          
  |             | 1:00                 |                 |                |     
            | tidigits           | 
 | **an4_words**    | 0:20          | 0:20             | 0:20        | 0:20     
  | 0:20        | 0:20                 | 0:20            | 0:20           | 
0:20            | wsj                | 
 | **an4_spelling** | 0:20          | 0:20             | 0:20        | 0:20     
  | 0:20        | 0:20                 | 0:20            | 0:20           | 
0:20            | wsj                | 
 | **an4_full**     | 1:30          | 1:30             | 1:30        | 1:30     
  | 1:30        | 1:30                 | 2:00            | 2:00           | 
3:00            | wsj                | 
 | **rm1**          | 22:00         | 22:00            | 1:30        | 01:30    
  | 2:30        | 22:00                | 25:00           | 25:00          | 
25:00           | rm1                | 
 | **hub4**         |               |                  |             |          
  | 10:00       |                      |                 |                |     
            | wsj                | 




 

## Some test notes

	* **trigram** tests are going away in favor of **trigram_fst** tests
	* Each test has a 'quick' version that takes 1/5 as long as the full 
test
	* There is a flaw in the fst/SimpleLinguist implementation that yields 
very large heaps for the  rm1_bigram_fst and rm1_trigram_fst tests. Once this 
flaw if corrected, the rm1 fst tests can be incorporated into the weekly 
schedule


## Test naming conventions

Full test names are built by concatenating the test name and the language 
model.  Some examples are:

	* an4_words_wordlist
	* rm1_flat_unigram_quick
	* an4_spelling_trigram_fst
	* tidigits_wordlist_quick

Note that this is a minor modification to the current naming scheme. 
Previously, some tests had no language model listed (an4, ti46). The 
regression.log will be updated to reflect this change for all old tests.

## Test schedule

Tests are run every night, on multiple machines and operating systems.  Tests 
start no earlier than 8PM eastern time, and should run no later than 6AM the 
following morning.  This allows for 10 hours of test time per machine per day. 
Saturday and Sunday tests can run between the hours of 6AM and 8PM as well.

# Standard Test

There is a 'standard test' set which is run every night on all machines.  It 
consists of the following test:

 | Test                                                                         
                                                            | Approximate time 
| 
 | ----                                                                         
                                                            | ---------------- 
| 
 | ti46_wordlist ti46_flat_unigram ti46_flat_unigram_fst                        
                                                            | 00:01            
| 
 | tidigits_wordlist_quick tidigits_flat_unigram_quick 
tidigits_flat_unigram_fst_quick tidigits_jsgf tidigits_wordlist_quick_dynamic   
     | 00:06            | 
 | an4_words_wordlist an4_words_unigram an4_words_bigram an4_words_trigram 
an4_words_unigram_fst an4_words_bigram_fst an4_words_trigram_fst | 0:25         
    | 
 | rm1_bigram_quick rm1_trigram_quick                                           
                                                            | 0:05             
| 
 | wsj5k_trigram                                                                
                                                            | 0:10             
| 
 | tidigits_wordlist_live_quick an4_words_bigram_live                           
                                                            |                  
| 
 | tidigits_rejection_quick an4_words_rejection                                 
                                                            | 0:20             
| 
 | Total Time                                                                   
                                                            | Approx 1:40      
| 


 ====== Weekly test schedule ======

By day:

 | Day of the week                                                              
                                                                                
                                                                                
                | Tests | Test Time |
 | ---------------                                                              
                                                                                
                                                                                
                | ----- | -----------
 | Sunday | tidigits_wordlist tidigits_flat_unigram tidigits_flat_unigram_fst 
wsj20k_trigram |  0:40 |                                                        
                                                                                
                 
 | Monday | tidigits_wordlist tidigits_flat_unigram tidigits_flat_unigram_fst | 
 0:20 |                                                                         
                                                                                
               
 | Tuesday | an4_spelling_wordlist an4_spelling_flat_unigram 
an4_spelling_unigram an4_spelling_bigram an4_spelling_flat_unigram_fst 
an4_spelling_unigram_fst an4_spelling_bigram_fst an4_spelling_trigram_fst 
an4_full_wordlist an4_full_flat_unigram |  0:45 |
 | Wednesday | an4_full_unigram an4_full_bigram an4_full_flat_unigram_fst |  
1:10 |                                                                          
                                                                                
                  
 | Thursday | an4_full_unigram_fst an4_full_bigram_fst an4_full_trigram_fst |  
0:45 |                                                                          
                                                                                
                
 | Friday | rm1_flat_unigram_quick rm1_unigram_quick rm1_unigram_fst_quick 
rm1_flat_unigram_fst_quick rm1_bigram_fst_quick |  0:10 |                       
                                                                                
                    
 | Saturday | an4_words_flat_unigram an4_words_flat_unigram_fst hub4_trigram |  
0:10 |                                                                          
                                                                                
               
 | async0 | rm1_flat_unigram |  0:25 |                                          
                                                                                
                                                                                
               
 | async1 | rm1_unigram |  0:25 |                                               
                                                                                
                                                                                
               
 | async2 | rm1_flat_unigram_fst | |                                            
                                                                                
                                                                                
               
 | async3 | rm1_unigram_fst | |                                                 
                                                                                
                                                                                
               
 | async4 | rm1_bigram |  0:25 |                                                
                                                                                
                                                                                
               
 | async5 | rm1_trigram |  0:15 |                                               
                                                                                
                                                                                
               
 | async6 | rm1_bigram_fst | |                                                  
                                                                                
                                                                                
               

By test:

 |                                                                              
                                                                                
                                                                                
                                                                                
                                                                                
                                  | Word List | flat unigram | unigram | bigram 
| flat unigram fst | unigram fst | bigram fst | trigram fst | Acoustic Model |
 |                                                                              
                                                                                
                                                                                
                                                                                
                                                                                
                                  | --------- | ------------ | ------- | ------ 
| ---------------- | ----------- | ---------- | ----------- | ----------------
 | ti46                                                |  0:01 **Nightly**      
       |  0:01 **Nightly**       |                              |               
               |  0:01 **Nightly**                     |                        
               |                                 |                              
                                 |  tidigits             |                      
                                 
 | tidigits                              |  0:05 **Mo**                |  0:05 
**Mo**    |                              |                              |  0:08 
**Mo**                  |                                       |               
                  |                                                             
  |  tidigits             |                                                     
                                  
 | tidigits_quick                |  0:01 **Nightly** | 0:01 **Nightly**         
|                               |                              |       0:01 
**Nightly**                                                |                    
                   |                                 |                          
                                     |  tidigits             |                  
                                     
 | an4_words                            |  0:04 **Nightly**      |  0:05 
**Nightly**                                   |       0:04 **Nightly**          
                |     0:04 **Nightly**                         |      0:05 
**Nightly**                                                |        0:04 
**Nightly**                               |       0:04 **Nightly**        |  
0:04 **Nightly** |  wsj                               |
 | an4_spelling                         |  0:04 **Tu**         |        0:04 
**Tu**     |  0:04 **Tu**  |  0:04 **Tu** |  0:01 **Tu**            |  0:04 
**Tu**          |  0:05 **Tu**        |  0:06 **Tu**                            
      |  wsj                           |                                        
                                                                                
                                        
 | an4_full                              |  0:15 **Tu**                |        
0:04 **Tu**     |      0:25 **We**  |   0:25 **We**    |         0:22 **We**    
 |  0:19 **Th**          |  0:26 **Th**          |  0:30 **Th**                 
                       |  wsj                           |                       
                                                                                
                                 
 | rm1_quick                            |                                       
 |  0:06  **Fr**        |  0:05 **Fr**  |  0:04 **Sa**  |              0:05 
**Fr**               |      0:05 **Fr**      |  0:05 **Sa**        |            
                                                     |  rm1                     
            |                                                                   
                                     


        * *Note*: Once the RM1 tests have been optimized to run in a reasonable 
amount of time, they will be added to the set of standard tests.


## Test Machines

 | Name    | CPUs | Cache (KB) | Clock Speed (MHz) | Memory (MB) | Architecture 
| OS    | 
 | ----    | ---- | ---------- | ----------------- | ----------- | ------------ 
| --    | 
 | filbert | 8    | 4096       | 2660              | 15904       | x86_64       
| Linux | 

## Historical Test Machines

 | Name      | CPUs | Cache (KB) | Clock Speed (MHz) | Memory (MB) | 
Architecture   | OS          | 
 | ----      | ---- | ---------- | ----------------- | ----------- | 
------------   | --          | 
 | argus     | 2    | 4096       | 360               | 512         | sparcv9    
    | solaris     | 
 | boteco    | 1    | ?          | 700               | 256         | pentium-3  
    | MS-Win2000  | 
 | debris    | 8    | 8 * 8096   | 750               | 32768       | 
UltraSPARC-III | solaris-5.9 | 
 | george    | 1    | 2048       | 2200              | 900         | pentium-4  
    | Linux       | 
 | glottis   | 2    | 8182       | 1015              | 2048        | 
UltraSPARC-III | solaris-5.9 | 
 | mangueira | 2    | 2560       | 750               | 1024        | blade1000  
    | solaris     | 
 | mickey    | 1    | 1800       | 1700              | 900         | pentium-4  
    | Linux       | 
 | mute      | 1    | 2048       | 296               | 128         | sparcv9    
    | solaris     | 
 | pharynx   | 1    | ?          | 450               | 256         | pentium-3  
    | Linux       | 
 | sunlabs   | 8    | 4096       | 336               | 4096        | E3500      
    | solaris     | 

