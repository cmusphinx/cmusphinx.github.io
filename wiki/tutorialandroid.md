---
layout: page
title: PocketSphinx on Android
---

---
* auto-gen TOC:
{:toc}
---

> **Caution!** This tutorial has not been tested for quite some
  time. It is possible that it no longer works with the latest
  versions of Android and the developer tools.

This tutorial describes a demo app that is available on [GitHub](https://github.com/cmusphinx/pocketsphinx-android-demo).

## PocketSphinx Android demo

In order to run the demo app we recommend to use Android Studio. You can
download the Android Studio IDE and sdk from the [official
download page](http://developer.android.com/sdk/).

### Building and running from Android Studio

In order to obtain the demo in the IDE, please select to *checkout a project
from VCS*, select *GitHub* and enter the project URL: <https://github.com/cmusphinx/pocketsphinx-android-demo>.

Once the project is set up, your IDE will update and download all dependencies
automatically. You should now be able to run the project.

After starting the app the recognizer will take some time to initialize. After
initialization it will wait for the keyword "oh mighty computer". Once this
keyword is detected, it will ask you to select the demo – "digits", "weather"
or "phones". The "digits" demo recognizes digits from 0 to 9, the "weather" demo
recognizes weather forecasts and the "phones" demo demonstrates phonetic
recognition.

To try a certain demo, import this project into the IDE and run it as usual.
In case of errors please check the logcat for further details.

### Building and running from Eclipse

We do not support the Eclipse project anymore, please consider an SDK upgrade.

### Building and running from the command line

You can also build the project with the gradle build system.

1. Clone the repo from Github:  
   `git clone https://github.com/cmusphinx/pocketsphinx-android-demo.git`.
2. Attach your physical device or setup a virtual device.
3. Create a file `local.properties` to point to sdk folder:  
   `sdk.dir = /home/user/android/sdk`.
4. Run `gradle installDebug`. It will build and install the application on the device.
5. Manually run the application from the device application menu.

## Using pocketsphinx-android

### Referencing the library in an Android project

The library is distributed as an Adroid Archive ([AAR](https://developer.android.com/studio/projects/android-library.html))
which includes both the binary so files for different architectures and
independent Java code.

In Android Studio you need to include the AAR into your project. Just go to
*File > New > New module* and choose *Import .JAR/.AAR Package*.

You can also add the AAR to your project using the command line as described in
[this stackoverflow post](http://stackoverflow.com/questions/21882804/adding-local-aar-files-to-my
-gradle-build).

Once the AAR is imported as module into the project, make sure it is listed as
a dependency of a main module in `app/build.gradle`:

```
dependencies {
    compile project(':aars')
}
```

### Setting permissions

In order to store asset files your application must have *WRITE_EXTERNAL_STORAGE*
permission. To record audio you need *RECORD_AUDIO* permission. Please note that
since Android 6.0, *RECORD_AUDIO* is not automatically enabled, but must be
confirmed in the application settings manually.

```
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
```

### Including resource files

The standard way to ship resource files with your application in Android
is to put them in the `assets/` directory of your project.
In order to make them available for pocketsphinx, these files should have
physical path. However, as long as they are in *.apk* they don't have such a
physical path.

The `Assets` class from pocketsphinx-android provides a method to automatically
copy asset files to the external storage of the  target device.
`edu.cmu.pocketsphinx.Assets#syncAssets` synchronizes  resources by reading
items from the `assets.lst` file which is located on the top `assets/`.
Before copying it matches the MD5 checksums of an asset and a file on external
storage with the same name in case it exists.

It only copies the files if there is incomplete information (no file on the
external storage, .md5 files are not available) or if there is a hash mismatch.
The PocketSphinxAndroidDemo contains an `ant` script that generates `assets.lst`
as well as the `.md5` files, look  for `assets.xml`.

Please note that the assets might be out of sync if the `ant` build script
didn't run properly in your build process. So, you should make sure that the
script runs or to create the md5 files and `assets.lst` yourself.

To integrate the assets sync into your application do the following:

1. Copy the `models/assets.xml` build file from the demo application into your
   application into the same folder `app`.
2. Edit the `app/build.gradle` build file to run `assets.xml`, just as in the
   Android demo:

```
ant.importBuild 'assets.xml'
preBuild.dependsOn(list, checksum)
clean.dependsOn(clean_assets)
```

That should do the trick. You can now verify that the `assets.lst` file was
created and that the md5 files are updated.

### Sample application

The classes and methods of pocketsphinx-android were designed to resemble the
same workflow used in pocketsphinx, except that basic data structures are
turned into classes and functions that work with these structures are turned
into methods of the corresponding classes. So, if you are familiar with
pocketsphinx you should feel comfortable with pocketsphinx-android, too.

The `SpeechRecognizer` is the main class to access decoder functionality. It
is created with the help of a `SpeechRecognizerSetup` builder.
A `SpeechRecognizerBuilder` allows to configure the main properties as well as
other parameters of the decoder. The parameters’s keys and values are the
same as those that are passed to the pocketsphinx binaries on the command-line.
Read more about tweaking the performance of pocketsphinx
[here](/wiki/pocketsphinxhandhelds).

```java
recognizer = defaultSetup()
	    .setAcousticModel(new File(assetsDir, "en-us-ptm"))
	    .setDictionary(new File(assetsDir, "cmudict-en-us.dict"))
	    .getRecognizer();
recognizer.addListener(this);
```

The decoder configuration is a lengthy process that contains IO operations, so
it’s recommended to run it inside an async task.

A decoder supports multiple named searches which you can switch in runtime:

```java
// Create keyword-activation search
recognizer.addKeyphraseSearch(KWS_SEARCH, KEYPHRASE);

// Create grammar-based searches
File menuGrammar = new File(assetsDir, "menu.gram");
recognizer.addGrammarSearch(MENU_SEARCH, menuGrammar);

// Next search for digits
File digitsGrammar = new File(assetsDir, "digits.gram");
recognizer.addGrammarSearch(DIGITS_SEARCH, digitsGrammar);

// Create language model search
File languageModel = new File(assetsDir, "weather.dmp");
recognizer.addNgramSearch(FORECAST_SEARCH, languageModel);
```

Once you set up the decoder and added all the searches you can start recognition
with:

```java
recognizer.startListening(searchName);
```

You will get notified on the speech end event in the `onEndOfSpeech` callback of
the recognizer listener. Then you could call `recognizer.stop()` or
`recognizer.cancel()`. Latter will cancel the recognition, former will cause the
final result to be passed in the `onResult` callback.

During the recognition you will receive partial results in the `onPartialResult`
callback.

You can also access other Pocketsphinx methods that are wrapped in Java classes in
swig. For details check for the `Decoder`, `Hypothesis`, `Segment` and `NBest`
classes.


## Building pocketsphinx-android

Pocketsphinx is provided with prebuilt binaries and it’s challenging to compile
it on various platforms. Unless you fully understand what you are doing, you
should rather not build it yourself. We recommend to use prebuilt binaries
instead.

### Build dependencies

  *  [Gradle](http://gradle.org/)
  *  [JDK](http://openjdk.java.net/) >= 1.6
  *  [SWIG](http://www.swig.org/) >= 2.0
  *  [Android SDK](http://developer.android.com/sdk/)
  *  [Android NDK](http://developer.android.com/tools/sdk/ndk/)

### Building steps

You need to checkout sphinxbase, pocketsphinx and pocketsphinx-android
and put them in the same directory.

```
root-directory
├─ pocketsphinx
├─ sphinxbase
└─ pocketsphinx-android
```

Older versions might be incompatible with the latest pocketsphinx-android,
so you need to make sure you are using the latest versions. You can use the
following commands to checkout the repositories:

```bash
git clone https://github.com/cmusphinx/sphinxbase
git clone https://github.com/cmusphinx/pocketsphinx
git clone https://github.com/cmusphinx/pocketsphinx-android
```

After arranging the directories you need to update the file `local.properties`
in the project root and define the following properties:

  * `sdk.dir` – the path to the Android SDK
  * `ndk.dir` – the path to the Android NDK

For example:

```bash
sdk.dir=/home/user/local/adt-bundle-linux-x86_64-20140321/sdk
ndk.dir=/home/user/local/android-ndk-r9d
```

After everything is set, run `gradle build`. This will create
`pocketsphinx-android-5prealpha-debug.aar` and
`pocketsphinx-android-5prealpha-release.aar` in `build/outputs/aar`.

<span class="post-bottom-nav">
  [Building an application with pocketsphinx](/wiki/tutorialpocketsphinx)
  [Building a dictionary](/wiki/tutorialdict)
</span>
