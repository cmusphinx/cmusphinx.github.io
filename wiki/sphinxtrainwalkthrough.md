---
layout: page 
---
This document contains an overview of SphinxTrain from the perspective of 
researchers or developers wishing to implement new features or training 
methods.  The process of training is already covered by the [Robust 
Tutorial](http://www.speech.cs.cmu.edu/sphinx/tutorial.html) and the [Sphinx 
Manual](http://fife.speech.cs.cmu.edu/sphinxman/), so it will not be covered 
here except as necessary.

## Layout of SphinxTrain code

The SphinxTrain code is organized into a few static libraries which contain 
most of the "core" functionality, and a large number of tools which do 
manipulations on acoustic model files.  In many cases, the tools mostly just 
call library functions and don't themselves contain a lot of code.

### Python Modules

The Python modules are located in the 'python/sphinx' directory of the source 
tree.  There is a setup.py file in the 'python' top-level directory which can 
be used to install these modules.  However, since all modules are written 
purely in Python, you can also simply set up your sys.path variable to point to 
the source directory (provided that you have first installed 
[NumPy](http://numpy.scipy.org)):

	
	#! python
	dhuggins@slim:~/Projects/Sphinx/SphinxTrain$ python
	Python 2.5.1 (r251:54863, Oct  5 2007, 13:36:32)
	[GCC 4.1.3 20070929 (prerelease) (Ubuntu 4.1.2-16ubuntu2)] on linux2
	Type "help", "copyright", "credits" or "license" for more information.
	>>> import sys
	>>> sys.path.insert(0, "python/sphinx")
	>>> from sphinx import *
	>>>

Some automatically-generated (and incomplete) documentation on the Python 
modules can be found at <http://lima.lti.cs.cmu.edu/pydoc/SphinxTrain/>

### Header Files

Nearly all of the header files useful in SphinxTrain development are in the 
'include/s3' directory.

### Libraries

The current directory layout of the library portion of the SphinxTrain code is:

	
	src/libs/libcommon
	src/libs/libio
	src/libs/libs2io
	src/libs/libcep_feat
	src/libs/libmodinv
	src/libs/libmllr
	src/libs/libclust
	src/libs/librpcc`</code>`
	A description of these libraries follows:
	

	 * libcommon: contains utility functions (many of which replicate or 
reimplement similar functions in SphinxBase)
	 * libio: contains reading and writing functions for every sort of file 
type dealt with in SphinxTrain.
	 * libs2io: contains reading and writing functions (or actually just 
writing, I think) for Sphinx-II format model files
	 * libcep_feat: contains dynamic feature computation code.  This does 
the same thing (but with a different API) as libsphinxfeat in SphinxBase.
	 * libmodinv: contains model inventory (i.e. acoustic model) functions, 
including Gaussian density and mixture model computation.
	 * libmllr: contains MLLR adaptation functions (which ought to be in 
libmodinv, probably)
	 * libclust: contains clustering functions, primarily related to 
decision tree building for state tying
	 * librpcc: is obsolete (formerly contained one function, which reads 
the performance counter on DEC Alphas using the RPCC instruction)
	Most of the code that you are likely to want to modify is in libmodinv 
and possibly also libclust and libcep_feat.
	
	###  Tools
	Source  code for the training programs is in the 'src/programs' 
directory.  The functionality of most of these programs is detailed elsewhere.  
The ones which are the most important for the training process are 'bw' and 
'norm'.  The 'bw' program collects expected state occupation and transition 
counts using the Forward-Backward algorithm, while 'norm' performs Maximum 
Likelihood updating of acoustic model parameters based on these counts.
	
	##  Writing SphinxTrain programs in C
	###  Basic structure of a SphinxTrain tool
	Let's look at the 'norm' tool to demonstrate the typical structure of a 
SphinxTrain tool written in C.  This tool's source code is in the directory 
'src/programs/norm' inside the SphinxTrain source tree, and contains the 
following files:
	
	`<code>`
	src/programs/norm/main.c
	src/programs/norm/Makefile
	src/programs/norm/parse_cmd_ln.c
	src/programs/norm/parse_cmd_ln.h

First the file 'Makefile' contains instructions for compiling this tool.  Most 
of the build logic is contained in the file config/common_make_rules which is 
included at the end of each tool's Makefile.  The Makefile simply defines a 
number of variables which describe the source files.  In the future, we may 
switch to using GNU Autotools in which case there will be a 'Makefile.am' which 
operates similarly.  Here is the important part of the Makefile:

	
	TOP=../../..
	DIRNAME=src/programs/norm
	BUILD_DIRS =
	ALL_DIRS= $(BUILD_DIRS)
	SRCS = \
	  main.c \
	  parse_cmd_ln.c
	H = \
	  parse_cmd_ln.h
	FILES = Makefile $(SRCS) $(H)
	TARGET = norm
	ALL = $(BINDIR)/$(TARGET)
	include $(TOP)/config/common_make_rules

The variables TOP and DIRNAME are required in all Makefiles in SphinxTrain.  
BUILD_DIRS and ALL_DIRS are only needed if there are subdirectories within the 
Makefile's directory which need to be built.  The SRCS variable lists all C 
source files to be compiled while the H variable lists all the header files in 
this directory.  The FILES directory lists all files which should be included 
in a distribution package.  The TARGET variable is very important - it gives 
the name of the program which is to be built from the source files in this 
directory.  The final two lines are required in order to make everything work.

Now, let's look at parse_cmd_ln.c and parse_cmd_ln.h.  The main function of 
these source files is to defined the command-line arguments to the tool.  For 
whatever historical reasons, these two files also contain a bunch of 
boilerplate code which is duplicated in every tool, with the only changes being 
the actual definition of arguments and the help text and description of the 
tool.  Again, this may change in the near future. Finally the main.c file 
contains the actual code.  Let's walk through what this does.  First, the 
initialize() function parses the command-line:

	
	#! cplusplus
	static int
	initialize(int argc,
	           char *argv[])
	{
	    /* define, parse and (partially) validate the command line */
	    parse_cmd_ln(argc, argv);
	    return S3_SUCCESS;
	}

This sets the internal variables which are read by the various functions in 
`<s3/cmd_ln.h>` such as cmd_ln_str(), cmd_ln_float32(), and such.  
Unfortunately these access macros are not used consistently in SphinxTrain, so 
you will see a lot of code that just uses cmd_ln_access() and casts the result 
to some other type.  This is BAD and should not be done, because in SphinxBase 
the return value of cmd_ln_access() is not a simple void pointer.

The actual work of the 'norm' program is done in the normalize() function.  
There is no good reason to structure your code this way - you could just as 
easily do all this stuff in the main() function, or preferably, you could split 
this up into a number of smaller funtcions.  But for whatever reason, this is 
the way that 'norm' was written, so let's look at what normalize() does.  
First, it declares an utterly ridiculous number of variables.  This is not 
accepted best practice for programming in the 21st century.  So, we'll skip 
that part.  Most of these variables are simply used to hold values acquired 
from the command-line. One important thing to note is that the command-line 
variable -accumdir is defined in parse_cmd_ln.c as a list of strings (using 
CMD_LN_STRING_LIST) and thus is stored in an array of pointers (char 
`<nowiki>``</nowiki>`).  This is because there are typically multiple 
accumulator directories, because the Forward-Backward part of training (using 
'bw') is often run in multiple parts on a cluster of networked machines.

The most common use case for 'norm' is to generate a new set of acoustic model 
files from accumulator directories alone.  It is also possible to specify input 
mean and variance files to be copied into the new model files in the case where 
some model parameters were not observed.  After validating the command-line 
arguments, the code checks to see if these files were specified, and reads in 
the data from them:

	
	#! cplusplus
	    if (in_mean_fn != NULL) {
	        E_INFO("Selecting unseen density mean parameters from %s\n",
	               in_mean_fn);
	        if (s3gau_read(in_mean_fn,
	                       &in_mean,
	                       &n_mgau,
	                       &n_gau_stream,
	                       &n_gau_density,
	                       &veclen) != S3_SUCCESS) {
	          E_FATAL_SYSTEM("Couldn't read %s", in_mean_fn);
	        }
	        ckd_free((void *)veclen);
	        veclen = NULL;
	    }
	    if (in_var_fn != NULL) {
	        E_INFO("Selecting unseen density variance parameters from %s\n",
	               in_var_fn);
	        if (var_is_full) {
	            if (s3gau_read_full(in_var_fn,
	                           &in_fullvar,
	                           &n_mgau,
	                           &n_gau_stream,
	                           &n_gau_density,
	                           &veclen) != S3_SUCCESS) {
	                E_FATAL_SYSTEM("Couldn't read %s", in_var_fn);
	            }
	        }
	        else {
	            if (s3gau_read(in_var_fn,
	                           &in_var,
	                           &n_mgau,
	                           &n_gau_stream,
	                           &n_gau_density,
	                           &veclen) != S3_SUCCESS) {
	                E_FATAL_SYSTEM("Couldn't read %s", in_var_fn);
	            }
	        }
	        ckd_free((void *)veclen);
	        veclen = NULL;
	    }

The functions s3gau_read() and s3gau_read_full() are defined in 
`<s3/s3gau_io.h>`.

Next, the code iterates over all of the accumulator directories given to it and 
builds a set of cumulative observation counts from the files in them (some 
obsolete code has been removed):

	
	#! cplusplus
	    for (i = 0; accum_dir[i]; i++) {
	        E_INFO("Reading and accumulating counts from %s\n", 
accum_dir[i]);
	        if (out_mixw_fn) {
	            rdacc_mixw(accum_dir[i],
	                       &mixw_acc, &n_mixw, &n_stream, &n_density);
	        }
	        if (out_tmat_fn) {
	            rdacc_tmat(accum_dir[i],
	                       &tmat_acc, &n_tmat, &n_state_pm);
	        }
	        if (out_mean_fn || out_var_fn) {
	            if (var_is_full)
	                rdacc_den_full(accum_dir[i],
	                               &wt_mean,
	                               &wt_fullvar,
	                               &pass2var,
	                               &dnom,
	                               &n_mgau,
	                               &n_gau_stream,
	                               &n_gau_density,
	                               &veclen);
	            else
	                rdacc_den(accum_dir[i],
	                          &wt_mean,
	                          &wt_var,
	                          &pass2var,
	                          &dnom,
	                          &n_mgau,
	                          &n_gau_stream,
	                          &n_gau_density,
	                          &veclen);
	            if (out_mixw_fn) {
	                if (n_stream != n_gau_stream) {
	                    E_ERROR("mixw inconsistent w/ densities WRT # "
	                            "streams (%u != %u)\n",
	                            n_stream, n_gau_stream);
	                }
	                if (n_density != n_gau_density) {
	                    E_ERROR("mixw inconsistent w/ densities WRT # "
	                            "den/mix (%u != %u)\n",
	                            n_density, n_gau_density);
	                }
	            }
	            else {
	                n_stream = n_gau_stream;
	                n_density = n_gau_density;
	            }
	        }
	    }

This is accomplished by simply adding together the counts from each directory 
to produce a running total, which is done using the functions rdacc_mixw(), 
rdacc_tmat(), and rdacc_den() (or rdacc_den_full() for full covariance 
matrices).  These functions are defined in `<s3/s3acc_io.h>`.  The variables 
mixw_acc, tmat_acc, wt_mean, wt_var, and wt_fullvar are used to store the 
cumulative counts.  You may or may not remember that the Baum-Welch update 
formula for maximum-likelihood estimation of the means of a continuous-density 
HMM is:

`<code>`#! latex
\begin{displaymath}
\hat\mu_{jk} = \frac{\sum_{t=1}^T \gamma_t(j,k) \vec o_t}{\sum_{t-1}^T 
\gamma_t(j,k)} 
\end{displaymath}
`</code>`

The SphinxTrain variables `wt_mean[i][j][k]` and `dnom[i][j][k]` correspond 
exactly to the numerator (note that this is a vector) and the denominator (note 
that this is a scalar) of this equation.  Therefore to do normalization we 
basically just have to divide `wt_mean` by `dnom`.  This is actually done in 
the file `src/libs/libmodinv/gauden.c` by the function `gauden_norm_wt_mean`.  
In `norm`, it is called in the following piece of code:

`<code>`#! cplusplus
    if (wt_mean || wt_var || wt_fullvar) {
	if (out_mean_fn) {
	    E_INFO("Normalizing mean for n_mgau= %u, n_stream= %u, n_density= 
%u\n",
		   n_mgau, n_stream, n_density);

	    gauden_norm_wt_mean(in_mean, wt_mean, dnom,
				n_mgau, n_stream, n_density, veclen);
	}
	else {
	    if (wt_mean) {
		E_INFO("Ignoring means since -meanfn not specified\n");
	    }
	}

	if (out_var_fn) {
	    if (var_is_full) {
		if (wt_fullvar) {
		    E_INFO("Normalizing fullvar\n");
		    gauden_norm_wt_fullvar(in_fullvar, wt_fullvar, pass2var, 
dnom,
					   wt_mean,	/* wt_mean now just 
mean */
					   n_mgau, n_stream, n_density, veclen,
					   cmd_ln_boolean("-tiedvar"));
		}
	    }
	    else {
		if (wt_var) {
		    E_INFO("Normalizing var\n");
		    gauden_norm_wt_var(in_var, wt_var, pass2var, dnom,
				       wt_mean,	/* wt_mean now just mean */
				       n_mgau, n_stream, n_density, veclen,
				       cmd_ln_boolean("-tiedvar"));
		}
	    }
	}
	else {
	    if (wt_var || wt_fullvar) {
		E_INFO("Ignoring variances since -varfn not specified\n");
	    }
	}
    }
    else {
	E_INFO("No means or variances to normalize\n");
    }
`</code>`

For variances, the standard Baum-Welch formula is:

`<code>`#!latex
\begin{displaymath}
\hat\Sigma_{jk} = \frac{\sum_{t=1}^T \gamma_t(j,k) (\vec o - \vec\mu_{jk})(\vec 
o - \vec\mu_{jk})^T}{\sum_{t-1}^T \gamma_t(j,k)}
\end{displaymath}
`</code>`

Note that the denominator of this equation is the same as in the mean 
re-estimation formula, and therefore the SphinxTrain variable `dnom` is used in 
both re-estimations.  In the case of diagonal covariances, the outer product 
operation in the numerator reduces to a simple element-wise squaring.  
Therefore the variance can be re-estmated independently in each dimension just 
like the mean.  There is one further wrinkle to do with two possible versions 
of this formula, which will be discussed in more depth below.

Finally, we write out the newly re-estimated means and variances, which is done 
with the function `s3gau_write`, in this code:

`<code>`#! cplusplus
    if (out_mean_fn) {
	if (wt_mean) {
	    if (s3gau_write(out_mean_fn,
			    (const vector_t ***)wt_mean,
			    n_mgau,
			    n_stream,
			    n_density,
			    veclen) != S3_SUCCESS)
		return S3_ERROR;
	    
	    if (out_dcount_fn) {
		if (s3gaudnom_write(out_dcount_fn,
				    dnom,
				    n_mgau,
				    n_stream,
				    n_density) != S3_SUCCESS)
		    return S3_ERROR;
	    }
	}
	else
	    E_WARN("NO reestimated means seen, but -meanfn specified\n");
    }
    else {
	if (wt_mean) {
	    E_INFO("Reestimated means seen, but -meanfn NOT specified\n");
	}
    }
    
    if (out_var_fn) {
	if (var_is_full) {
	    if (wt_fullvar) {
		if (s3gau_write_full(out_var_fn,
				     (const vector_t ****)wt_fullvar,
				     n_mgau,
				     n_stream,
				     n_density,
				     veclen) != S3_SUCCESS)
		    return S3_ERROR;
	    }
	    else
		E_WARN("NO reestimated variances seen, but -varfn specified\n");
	}
	else {
	    if (wt_var) {
		if (s3gau_write(out_var_fn,
				(const vector_t ***)wt_var,
				n_mgau,
				n_stream,
				n_density,
				veclen) != S3_SUCCESS)
		    return S3_ERROR;
	    }
	    else
		E_WARN("NO reestimated variances seen, but -varfn specified\n");
	}
    }
    else {
	if (wt_var) {
	    E_INFO("Reestimated variances seen, but -varfn NOT specified\n");
	}
    }
`</code>`

## Writing SphinxTrain programs in Python

The Python modules included with SphinxTrain make it easy to write small 
scripts that manipulate acoustic models. They can also be used to examine 
models interactively through the Python shell. We highly recommend installing 
the [matplotlib](http://matplotlib.sourceforge.net/) and 
[ipython](http://ipython.sourceforge.net) extensions to Python, which, in 
combination, provide a convenient, MATLAB-like environment for manipulating and 
viewing numerical data.

Let's look at the Python equivalent of the code for 'norm' described above.  
We'll skip command line parsing since you can do that using the Python [getopt 
module](http://docs.python.org/lib/module-getopt.html), and just assume that 
all the accumulator directories live under the directory 'bwaccumdir' in the 
current directory.  We will also assume that the python modules are installed 
in the 'python' directory under the current directory, which is what 
SphinxTrain's training setup script will do for you by default.

First, we have to set up the environment and load the necessary modules:

	
	#! python
	#!/usr/bin/env python
	import os
	import sys
	sys.path.append('python')
	from sphinx import s3gaucnt

In this case the `s3gaucnt` and `s3gau` modules are the only ones we actually 
need.  The first module provides the `sphinx.s3gaucnt.S3GauCntFile` class and 
some helper functions which we will use to process observation count files, 
while the second one provides the `sphinx.s3gau.S3GauFile` class which we will 
use to write out the re-estimated parameter files.  Now we will use the 
`sphinx.s3gaucnt.accumdirs` function to accumulate counts from a list of 
directories: (if we were dealing with full covariance accumulators, we would 
use `sphinx.s3gaucnt.accumdirs_full`)

	
	#! python
	# Accumulate observation counts from all accumulation directories
	gauden = s3gaucnt.accumdirs([os.path.join('bwaccumdir', x) for x in 
os.listdir('bwaccumdir')])

The return value for this function is a `sphinx.s3gaucnt.S3GauCntFile` object 
which contains the merged counts for all the items in the list of directories 
passed to `accumdirs`.  We can then obtain the mean, variance, mixture weight, 
and transition matrix counts from this object, as well as the normalization 
constant ("dnom", which corresponds to the denominator of the Baum-Welch update 
formulae), using a set of accessor functions.

Let's normalize the means first, since they are very simple.  The means are 
obtained using the `getmeans` method, while the normalizer is obtained using 
`getdnom`.  What you actually get is a list of lists of 2-dimensional arrays 
(of type `numpy.ndarray`).  For more information on how to manipulate these, 
please see [the NumPy documentation](http://numpy.scipy.org/).

	
	#! python
	# Normalize means
	outmeans = []
	# For each codebook in the counts file and its associated set of 
normalizers:
	for mgau, mgau_dnom in zip(gauden.getmeans(), gauden.getdnom()):
	    # Create a list of re-estimated parameters for this codebook and 
append it to the output array
	    outmgau = []
	    outmeans.append(outmgau)
	    # For each feature stream in this codebook and its associated set 
of normalizers:
	    for feat, feat_dnom in zip(mgau, mgau_dnom):
	        # Normalize the parameters (dividing a 2-D array by a 1-D array)
	        outmgau.append(feat / feat_dnom)
	# Write out the re-estimated means
	s3gau.open("means", "wb").writeall(outmeans)

And that's it, seriously.  Note how we can use `zip` to ensure that we match up 
codebooks and features with their corresponding normalizers without ever having 
to deal with any index variables.  `itertools.izip` might be a bit more 
efficient in this case.

Now we will do the variances.  These are only slightly more complicated due to 
the fact that we have to distinguish between "two-pass" and "one-pass" variance 
accumulators.  You may recall that there are two mathematically equivalent 
forms of the maximum likelihood estimator of variance:

	
	#! latex
	\begin{displaymath}
	\sigma^2 = E[(x-\mu)^2] = E[x^2] - \mu^2
	\end{displaymath}

In the first case the sufficient statistic is `<<latex($(x-\mu)^2$)>`> while in 
the second case it is `<<latex($x^2$)>`> and does not depend on the means at 
all.  In Sphinx speak, the first case is called "two-pass" estimation, and is 
used when the `-2passvar yes` flag is specified to `bw`.  The count file 
contains a flag which describes which type of variance statistics it contains.  
This is stored in the `pass2var` attribute of the 
`sphinx.s3gaucnt.S3GauCntFile` object.  So, our variance normalization looks 
like this:

	
	#! python
	# Normalize variances
	outvars = []
	# For each codebook in the counts file and its associated set of 
normalizers:
	for mgau, mgau_dnom, mgau_mean in zip(gauden.getvars(), 
gauden.getdnom(), outmeans):
	    # Create a list of re-estimated parameters for this codebook and 
append it to the output array
	    outmgau = []
	    outvars.append(outmgau)
	    # For each feature stream in this codebook and its associated set 
of normalizers:
	    for feat, feat_dnom, feat_mean in zip(mgau, mgau_dnom, mgau_mean):
	        if gauden.pass2var:
	            # Two pass variance: statistic is (x-\mu)^2, we just have 
to normalize it
	            outmgau.append(feat / feat_dnom)
	        else:
	            # One pass variance: statistic is x^2, we have to subtract 
the squared re-estimated mean
	            outmgau.append(feat / feat_dnom - feat_mean * feat_mean)
	# Write out the re-estimated variances
	s3gau.open("variances", "wb").writeall(outvars)

And finally, to normalize the mixture weights and transition matrices, we do... 
nothing.  In fact that is what the original `norm` program does as well.  The 
reason for this is that the normalization constant for these can be computed 
quickly when the models are loaded, and there are some benefits to storing them 
in unnormalized form (for one, it allows them to be used as priors for MAP 
adaptation).

