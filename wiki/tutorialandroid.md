---
layout: page 
title: Pocketsphinx on Android
---

* auto-gen TOC:
{:toc}

# Pocketsphinx on Android

## Introduction

Current tutorial describes a most recent version available on [ github]( 
http///github.com/cmusphinx/pocketsphinx-android-demo ).

## PocketSphinx Android demo

To try the demo the best thing would be to use Android Studio. You can download 
Android Studio IDE and sdk from the [ official page 
](http://developer.android.com/sdk/index.html )

### Building and running from Android Studio

You can obtain the demo in IDE, select to checkout project from VCS, select 
github and enter the project URL 
http://github.com/cmusphinx/pocketsphinx-android-demo

Once project will be set, IDE will update and download all dependencies 
automatically, you can just run the project.

After start recognizer will take some time to initialize, then it will wait for 
a keyword "oh mighty computer". Once keyword is detected, it will ask you to 
select the demo - "digits", "weather" or "phones". The digits demo recognizes 
digits from 0 to 9, the weather demo recognizes weather forecasts and "phones" 
demo demonstrates phonetic recognition.

To try it import this project into IDE and run as usual, check logcat for 
details if something doesn't work.

### Building and running from Eclipse

We do not support Eclipse project anymore, please consider SDK upgrade.

### Building and running from command-line

You can also build with gradle build system.

 1.  Clone from Github [ pocketsphinx android demo source code 
](http://github.com/cmusphinx/pocketsphinx-android-demo ).
 2.  Attach your physical device or setup a virtual device.
 3.  Create a file local.properties to point to sdk folder: `sdk.dir = 
/home/user/android/sdk`
 4.  Run `gradle installDebug` It will build and install the application on the 
device.
 5.  Manually run the application from the device application menu.

## Using pocketsphinx-android

### Referencing the library in Android project

Library is distributed as an 
[AAR](https://developer.android.com/studio/projects/android-library.html) 
archive which includes both binary so files for different architectures and 
independent java code.

In Android Studio you need to include AAR into your project. Just go to File > 
New > New module and choose Import .JAR/.AAR Package.

You can also add AAR to your project in command line as described in 
[stackoverflow 
answer](http://stackoverflow.com/questions/21882804/adding-local-aar-files-to-my
-gradle-build).

### Setting permissions

To store asset files your application should have WRITE_EXTERNAL_STORAGE 
permission. To record audio you need RECORD_AUDIO permission. Please note that 
since Android 6.0, RECORD_AUDIO is not automatically enabled, it must be 
confirmed in application settings manually.

	
	    `<uses-permission 
android:name="android.permission.WRITE_EXTERNAL_STORAGE" />`
	    `<uses-permission android:name="android.permission.RECORD_AUDIO" />`


### Including resource files

The standard way to ship resource files with your application in Android is to 
put them in `assets/` directory of your project. But in order to make them 
available for pocketsphinx files should have physical path, as long as they are 
within .apk they don't have one. Assets class from pocketsphinx-android 
provides a method to automatically copy asset files to external storage of the 
target device. `edu.cmu.pocketsphinx.Assets#syncAssets` synchronizes 
resources reading items from `assets.lst` file located on the top 
`assets/`. Before copying it matches MD5 checksums of an asset and a file on 
external storage with the same name if such exists. It only does actualy 
copying if there is incomplete information (no file on external storage, no any 
of two .md5 files)  or there is hash mismatch. PocketSphinxAndroidDemo contains 
`ant` script that generates `assets.lst` as well as `.md5` files, look 
for `assets.xml`. 

Please note that if ant build script doesn't run properly in your build 
process, assets might be out of sync. Make sure that script runs or create md5 
files and assets.lst yourself.

To integrate assets sync in your application do the following

 1.  Copy `app/asset.xml` build file from demo application into your 
application into same folder `app`.
 2.  Edit `app/build.gradle` build file to run `assets.xml`, just as in 
android demo:

	
	      ant.importBuild 'assets.xml'
	      preBuild.dependsOn(list, checksum)
	      clean.dependsOn(clean_assets)


That should do the trick

### Sample application

The classes and methods of pocketsphinx-android were designed to resemble the 
same workflow used in pocketsphinx, except that basic data structures organized 
into classes and functions working with them are turned into methods of the 
corresponding classes. So if you are familiar with pocketsphinx you should feel 
comfortable with pocketsphinx-android too.

`SpeechRecognizer` is the main class to access decoder functionality. It is 
created with the help of `SpeechRecognizerSetup` builder. 
`SpeechRecognizerBuilder` allows to configure main properties as well as 
other parameters of teh decoder. The parameters keys and values are the same as 
those are passed in command-line to pocketsphinx binaries. Read [ 
more](pocketsphinxhandhelds ) about tweaking pocketsphinx performance.

	:::java
	        recognizer = defaultSetup()
	                .setAcousticModel(new File(assetsDir, "en-us-ptm"))
	                .setDictionary(new File(assetsDir, 
"cmudict-en-us.dict"))
	                .setRawLogDir(assetsDir).setKeywordThreshold(1e-20f)
	                .getRecognizer();
	        recognizer.addListener(this);


Decoder configuration is lengthy process that contains IO operation, so it's 
recommended to run in inside async task.

Decoder supports multiple named searches which you can switch in runtime

	:::java
	        // Create keyword-activation search.
	        recognizer.addKeyphraseSearch(KWS_SEARCH, KEYPHRASE);
	        // Create grammar-based searches.
	        File menuGrammar = new File(assetsDir, "menu.gram");
	        recognizer.addGrammarSearch(MENU_SEARCH, menuGrammar);
	        // Next search for digits
	        File digitsGrammar = new File(assetsDir, "digits.gram");
	        recognizer.addGrammarSearch(DIGITS_SEARCH, digitsGrammar);
	        // Create language model search.
	        File languageModel = new File(assetsDir, "weather.dmp");
	        recognizer.addNgramSearch(FORECAST_SEARCH, languageModel);


Once you setup the decoder and add all the searches you can start recognition 
with

	:::java
	recognizer.startListening(searchName);


You will get notified on speech end event in `onEndOfSpeech` callback of the 
recognizer listener. Then you could call
`recognizer.stop` or `recognizer.cancel()`. Latter will cancel the 
recognition, former will cause the final result
be passed you in `onResult` callback.

During the recognition you will get partial results in `onPartialResult` 
callback.

You can also access other Pocketsphinx method wrapped with Java classes in 
swig, check for details Decoder, Hypothesis, Segment and NBest classes.


## Building pocketsphinx-android

Pocketsphinx is provided with prebuilt binaries and it's not easy to compile it 
on various platforms. You shouldn't build it unless you understand what you are 
doing. Use prebuilt binaries instead.

### Build dependencies

*  [ Gradle](http://gradle.org/ )
*  [ JDK](http://openjdk.java.net/ ) **>= 1.6**
*  [ SWIG](http://www.swig.org/ ) **>= 2.0**
*  [ Android SDK](http://developer.android.com/sdk/ )
*  [ Android NDK](http://developer.android.com/tools/sdk/ndk/ )

### Building steps

You need to checkout sphinxbase, pocketsphinx and pocketsphinx-android
and put them in the same folder.

	
	Root folder
	 \_pocketsphinx
	 \_sphinxbase
	 \_pocketsphinx-android


Older versions might be incompatible with the latest pocketsphinx-android,
so you need to make sure you are using latest versions. You can use
the following command to checkout from repository:

        git clone http://github.com/cmupshinx/sphinxbase
        git clone http://github.com/cmupshinx/pocketsphinx
        git clone http://github.com/cmupshinx/pocketsphinx-android

After arragement of the files you need to update the file
`local.properties` in the project root and define the following
properties:

*  `sdk.dir` - path to Android SDK
*  `ndk.dir` - path to Android NDK

For example:

	
	sdk.dir=/home/user/local/adt-bundle-linux-x86_64-20140321/sdk
	ndk.dir=/home/user/local/android-ndk-r9d


After everything is set, run `gradle build`. It will create 
pocketsphinx-android-5prealpha-debug.aar and 
pocketsphinx-android-5prealpha-release.aar in build/outputs/aar.
