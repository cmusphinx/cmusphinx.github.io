---
layout: page 
---
# Installing SciPy for Sphinxtrain

This page details the things you need to install on Windows, Linux, and Mac OS 
X in order to do Python development with SphinxTrain.

Note that Python is **not** currently **required** to use SphinxTrain, unless 
you wish to use [LDA and MLLT feature transforms](/wiki/LDAMLLT).  However,
prototyping and research is currently being done primarily using Python.

First, install Python.  For Windows and Mac OS X, the latest release can be 
found at <http://www.python.org/download/>.  On Linux, it should be installed
with your distribution.

Next, install NumPy and SciPy.  For Windows, there are binary packages at 
<http://scipy.org/Download>.  On Mac OS X, you may be able to find it via
[Fink](http://fink.sf.net) or [DarwinPorts](http://darwinports.com/).  On 
Linux, it should be available in most recent distributions.  On Ubuntu and 
Debian, for instance, you can just run:

	
	apt-get install python-scipy


If you don't have a binary package to install, then you can compile it from 
source following the instructions on <http://scipy.org/Download>.

You can also optionally install [iPython](http://ipython.scipy.org/) and 
[matplotlib](http://matplotlib.sourceforge.net/) for enhanced interactive use 
and MATLAB-like graphics.

Now, if you wish to install the SphinxTrain Python modules system-wide, go to 
the `python` directory inside your SphinxTrain installation and run:

	
	python setup.py build
	sudo python setup.py install


