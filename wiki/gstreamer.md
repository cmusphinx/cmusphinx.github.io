---
layout: page 
---
#  Using PocketSphinx with GStreamer and Python 

PocketSphinx supports for the [GStreamer](http://gstreamer.freedesktop.org/) streaming media framework.  What this means is that the PocketSphinx decoder can be treated as an element in a media processing pipeline, specifically, one which filters audio into text.

This is useful because it simplifies a lot of the work involved in writing interactive speech applications.  You don't need to handle any of the difficulties of dealing with audio input.  Also, perhaps more importantly, it means that speech input can be integrated into a GUI application using [GTK+](http://www.gtk.org/) without agonizing pain.

## Overview

This document will walk you through the steps of building a simple demo using GTK+, GStreamer, and Python.  The only thing this program will do is recognize speech and display it in a text area, but it should give you the tools you need to do more interesting things.

Before starting, you should have an up-to-date version of pocketsphinx from the subversion repository, as well as GStreamer, GTK+, and the Python.

Pocketsphinx will automatically build the GStreamer plugin, as part of the normal build process, so long as the appropriate GStreamer 1.0 development files (header files, etc) are present and identified on your development platform. Pocketsphinx tests for them via the pkg-config infrastructure (which allows libraries to place .pc package-configuration files on a system, so that other software has a unified interface to query for library location, version info, compiler/linker flags, etc.). If these .pc files are not identified during execution of the configure script (typically called automatically by autogen.sh), then their absence will be noted in the script's output log. However, the script will proceed to configure pocketsphinx without them (the GStreamer plugin, after all, is an optional component of pocketsphinx).

In particular, pocketsphinx requires that the following three .pc files be available in order to configure the platform to build the GStreamer plugin :

	
	gstreamer-1.0.pc
	gstreamer-base-1.0.pc
	gstreamer-plugins-base-1.0.pc


These files can be stored in different locations based on O/S preferences. For example, in Debian [Jessie], the first two are provided in the libgstreamer1.0-dev package and the last one is provided in the libgstreamer-plugins-base1.0-dev package. You can check that GStreamer is properly installed with

	
	pkg-config --modversion gstreamer-plugins-base-1.0


You also should see the following line in configuration script output:

	
	checking for GStreamer... yes


After installation you should make sure that GStreamer is able to find the pocketsphinx plugin.  If you have installed pocketsphinx in ''/usr/local'', then you may have to set the following environment variable:

	
	export GST_PLUGIN_PATH=/usr/local/lib/gstreamer-1.0


If you have installed pocketsphinx under a different prefix, you will also need to set the ''LD_LIBRARY_PATH'' variable.  If your installation prefix is ''$psprefix'', then you would want to set these variables.

	
	export LD_LIBRARY_PATH=$psprefix/lib
	export GST_PLUGIN_PATH=$psprefix/lib/gstreamer-1.0


To verify that GStreamer can find the plugin, run ''gst-inspect-1.0 pocketsphinx''.  You should get a large amount of output, ending with something like this:

	
	  decoder             : The underlying decoder
	                        flags: readable
	                        Boxed pointer of type "PSDecoder"


If you instead see something like this:

	
	No such element or plugin 'pocketsphinx'


and all above environmental variables are properly set, it's quite possible that you haven't properly compiled the GStreamer plugin as part of pocketsphinx build, you can check the ''config.log'' in pocketsphinx and build log for details.
## Background Reading

Before you look at this, it would be a good idea to look at:


*  The [PyGTK Tutorial](http://pygtk.org/pygtk2tutorial/index.html)

*  The [GStreamer Application Developer's Guide](http://gstreamer.freedesktop.org/data/doc/gstreamer/head/manual/html/index.html)

*  The [GStreamer Python Tutorial](http://pygstdocs.berlios.de/pygst-tutorial/index.html)

Of course, we also assume that you know [Python](http://www.python.org/).

##  Skeleton of a simple GUI program 

Our simple demo program will just consist of a window, a text box, and a button which the user can push to start and stop speech recognition.

We will start by creating a Python class representing our demo application:

	
	class DemoApp(object):
	    """GStreamer/PocketSphinx Demo Application"""
	    def __init__(self):
	        """Initialize a DemoApp object"""
	        self.init_gui()
	        self.init_gst()
	
	    def init_gui(self):
	        """Initialize the GUI components"""
	
	    def init_gst(self):
	        """Initialize the speech components"""
	
	app = DemoApp()
	gtk.main()


Now let's fill in the ''init_gui'' method.  We are going to create a window with a ''gtk.VBox'' in it, holding a ''gtk.TextView'' (with associated ''gtk.TextBuffer'') and a ''gtk.ToggleButton''.

	
	    def init_gui(self):
	        """Initialize the GUI components"""
	        self.window = gtk.Window()
	        self.window.connect("delete-event", gtk.main_quit)
	        self.window.set_default_size(400,200)
	        self.window.set_border_width(10)
	        vbox = gtk.VBox()
	        self.textbuf = gtk.TextBuffer()
	        self.text = gtk.TextView(buffer=self.textbuf)
	        self.text.set_wrap_mode(gtk.WRAP_WORD)
	        vbox.pack_start(self.text)
	        self.button = gtk.ToggleButton("Speak")
	        self.button.connect('clicked', self.button_clicked)
	        vbox.pack_start(self.button, False, False, 5)
	        self.window.add(vbox)
	        self.window.show_all()


This gives us a nice text box and a button that does nothing.  Now, we want to introduce GStreamer and PocketSphinx to our program, so that clicking the button makes it recognize speech and insert it into the textbox at the current location.

## Adding GStreamer to the program

To use GStreamer, we need to add a few more lines of imports to the top of the program:

	
	gi.require_version('Gst', '1.0')
	from gi.repository import GObject, Gst
	GObject.threads_init()
	Gst.init(None)


Now, we will fill in the ''gst.init'' method, which initializes a GStreamer pipeline that will do speech recognition for us.

## Creating the pipeline

For simplicity we are creating it using the ''gst.parse_launch'' function, which reads a  textual description and creates a pipeline from it.  If you are not running GNOME, you may need to change ''gconfaudiosrc'' to a different source element, such as ''alsasrc'' or ''osssrc''.

	
	        self.pipeline = gst.parse_launch('autoaudiosrc ! audioconvert ! audioresample '
	                                         + '! pocketsphinx name=asr ! fakesink')


This pipeline consists of an audio source, followed by conversion and resampling (the ''pocketsphinx'' element currently requires 16kHz, 16-bit PCM audio), followed by recognition.  The ''fakesink'' element discards the output of the speech recognition element (more on this below). There used to be vader element before to do voice activity detection, now voice activity is detected inside decoder for best accuracy.

## Running the pipeline

We are going to set up our program so that the pipeline starts out being paused, and is set to play (i.e. start recognition) when the user presses the "Speak" button.  Once a final recognition result is retrieved we will put it back in paused mode and wait for another button press.  If the user presses the button in the middle of recognition, it will halt speech recognition immediately.  We will implement this using the ''button_clicked'' method which was connected to the button's signal in ''init_gui'':

	
	    def button_clicked(self, button):
	        """Handle button presses."""
	        if button.get_active():
	            button.set_label("Stop")
	            self.pipeline.set_state(gst.State.PLAYING)
	        else:
	            button.set_label("Speak")
	            self.pipeline.set_state(gst.State.PAUSED)


## The 'pocketsphinx' element

The ''pocketsphinx'' element functions as a filter - it takes audio data as input and produces text as output.  This makes sense in the GStreamer framework, where data flows from a source to a sink, and is potentially useful for captioning or other multimedia applications.  However, due to the fact that automatic speech recognition by nature operates on arbitrarily large chunks of data ("utterances"), and the recognition result for an utterance can change as more data becomes available, this simple streaming data flow is not suitable for most ASR applications.

In practice, we want to treat the speech recognizer more like an input device in GUI programming, which emits a stream of messages that can be subscribed to and interpreted by a controller object.  Fortunately, GStreamer allows us to do (almost) exactly this, using the same mechanism used for GTK+ widgets.  So, we are going to connect methods in ''DemoApp'' to the messages emitted by the ''pocketsphinx'' element:

	
	        bus = self.pipeline.get_bus()
	        bus.add_signal_watch()
	        bus.connect('message::element', self.element_message)
	        
	        
	    def element_message(self, bus, msg):
	        """Receive element messages from the bus."""
	        msgtype = msg.get_structure().get_name()
	        if msgtype != 'pocketsphinx':
	            return
	
	        if msg.get_structure().get_value('final'):
	                self.final_result(msg.get_structure().get_value('hypothesis'), msg.get_structure().get_value('confidence'))
	            self.pipeline.set_state(gst.State.PAUSED)
	            self.button.set_active(False)
	        elif msg.get_structure().get_value('hypothesis'):
	            self.partial_result(msg.get_structure().get_value('hypothesis'))


Now, the methods ''partial_result'' and ''final_result'' will be called whenever a partial or complete utterance is decoded.

## Updating the text buffer

Finally, we need to implement the methods which update the text buffer with the current partial and final recognition result.  There isn't much of interest here with relation to GStreamer and PocketSphinx.

	
	    def partial_result(self, hyp, uttid):
	        """Delete any previous selection, insert text and select it."""
	        # All this stuff appears as one single action
	        self.textbuf.begin_user_action()
	        self.textbuf.delete_selection(True, self.text.get_editable())
	        self.textbuf.insert_at_cursor(hyp)
	        ins = self.textbuf.get_insert()
	        iter = self.textbuf.get_iter_at_mark(ins)
	        iter.backward_chars(len(hyp))
	        self.textbuf.move_mark(ins, iter)
	        self.textbuf.end_user_action()
	
	    def final_result(self, hyp, uttid):
	        """Insert the final result."""
	        # All this stuff appears as one single action
	        self.textbuf.begin_user_action()
	        self.textbuf.delete_selection(True, self.text.get_editable())
	        self.textbuf.insert_at_cursor(hyp)
	        self.textbuf.end_user_action()


## Configuring the decoder and improving accuracy

Sometimes you might want to configure the decoder to improve accuracy, improve speed of decoding or run some specific mode.
You can configure many of pocketsphinx options with gstreamer properties since we mapped gstreamer properties to pocketsphinx configuration. Not every property is supported, for example, you won't be able to set ''-topn'' and similar exotic configurations. But major configuration options are supported.

You can get a list of properties supported with ''gst-inspect-1.0 pocketsphinx''.

For example, to use your improved language model with GStreamer, you just have to set the ''lm'' and ''dict'' properties on the ''pocketsphinx'' element.  So, if your language model is in /home/user/mylanguagemodel.lm and the associated dictionary is /home/user/mylanguagemodel.dic, you would add these lines to the ''init_gst'' method.

	
	        asr = self.pipeline.get_by_name('asr'); # We previously assigned pocketsphinx element a name asr
	        asr.set_property('lm', '/home/user/mylanguagemodel.lm')
	        asr.set_property('dict', '/home/user/mylanguagemodel.dic')


You can also set properties directly in initialization pipeline:

	
	        self.pipeline = gst.parse_launch('autoaudiosrc ! audioconvert ! audioresample '
	                                        + '! pocketsphinx name=asr beam=1e-20 ! fakesink')


Properties are applied when pipeline starts, so you can reconfigure several times without any problem. On the other hand, you won't see the effect of the property set until you start the pipeline.

## Code Listing

You can [download the Python code for this example](http://svn.code.sf.net/p/cmusphinx/code/trunk/pocketsphinx/src/gst-plugin/livedemo.py).  A complete code listing follows:

	
	from gi import pygtkcompat
	import gi
	
	gi.require_version('Gst', '1.0')
	from gi.repository import GObject, Gst
	GObject.threads_init()
	Gst.init(None)
	    
	gst = Gst
	    
	print("Using pygtkcompat and Gst from gi")
	
	pygtkcompat.enable() 
	pygtkcompat.enable_gtk(version='3.0')
	
	import gtk
	
	class DemoApp(object):
	    """GStreamer/PocketSphinx Demo Application"""
	    def __init__(self):
	        """Initialize a DemoApp object"""
	        self.init_gui()
	        self.init_gst()
	
	    def init_gui(self):
	        """Initialize the GUI components"""
	        self.window = gtk.Window()
	        self.window.connect("delete-event", gtk.main_quit)
	        self.window.set_default_size(400,200)
	        self.window.set_border_width(10)
	        vbox = gtk.VBox()
	        self.textbuf = gtk.TextBuffer()
	        self.text = gtk.TextView(buffer=self.textbuf)
	        self.text.set_wrap_mode(gtk.WRAP_WORD)
	        vbox.pack_start(self.text)
	        self.button = gtk.ToggleButton("Speak")
	        self.button.connect('clicked', self.button_clicked)
	        vbox.pack_start(self.button, False, False, 5)
	        self.window.add(vbox)
	        self.window.show_all()
	
	    def init_gst(self):
	        """Initialize the speech components"""
	        self.pipeline = gst.parse_launch('autoaudiosrc ! audioconvert ! audioresample '
	                                         + '! pocketsphinx ! fakesink')
	        bus = self.pipeline.get_bus()
	        bus.add_signal_watch()
	        bus.connect('message::element', self.element_message)
	
	        self.pipeline.set_state(gst.State.PAUSED)
	
	
	
	
	    def element_message(self, bus, msg):
	        """Receive element messages from the bus."""
	        msgtype = msg.get_structure().get_name()
	        if msgtype != 'pocketsphinx':
	            return
	
	        if msg.get_structure().get_value('final'):
	                self.final_result(msg.get_structure().get_value('hypothesis'), msg.get_structure().get_value('confidence'))
	            self.pipeline.set_state(gst.State.PAUSED)
	            self.button.set_active(False)
	        elif msg.get_structure().get_value('hypothesis'):
	            self.partial_result(msg.get_structure().get_value('hypothesis')
	
	    def partial_result(self, hyp):
	        """Delete any previous selection, insert text and select it."""
	        # All this stuff appears as one single action
	        self.textbuf.begin_user_action()
	        self.textbuf.delete_selection(True, self.text.get_editable())
	        self.textbuf.insert_at_cursor(hyp)
	        ins = self.textbuf.get_insert()
	        iter = self.textbuf.get_iter_at_mark(ins)
	        iter.backward_chars(len(hyp))
	        self.textbuf.move_mark(ins, iter)
	        self.textbuf.end_user_action()
	
	    def final_result(self, hyp, confidence):
	        """Insert the final result."""
	        # All this stuff appears as one single action
	        self.textbuf.begin_user_action()
	        self.textbuf.delete_selection(True, self.text.get_editable())
	        self.textbuf.insert_at_cursor(hyp)
	        self.textbuf.end_user_action()
	
	    def button_clicked(self, button):
	        """Handle button presses."""
	        if button.get_active():
	            button.set_label("Stop")
	            self.pipeline.set_state(gst.State.PLAYING)
	        else:
	            button.set_label("Speak")
	            self.pipeline.set_state(gst.State.PAUSED)
	
	app = DemoApp()
	gtk.main()

