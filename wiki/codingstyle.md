---
layout: page 
---
This page describes the coding style guidelines in use for the components of Sphinx that are written in the C programming language.  It does not apply to Sphinx4, or to Python and Perl components.

These guidelines are generally based on the historical best practice within the Sphinx source.  Unfortunately, much of the code has been written by different people in different environments and does not always conform.  If you want to go and correct it, that is great, as it will probably help you understand the code better (as long as you don't just run "indent" over it or whatever).

##  Language Features 

Sphinx is written in ANSI C89.  In practice, this actually just means that we try to be compatible back to GCC 2.95 and MSC 6.0.  Using GNU, MS, or C99 extensions (such as "inline") is allowed as long as they are properly conditionalized with some portable fallback mechanism.  So, by this logic, C99/C++ style comments are not allowed, nor is Win32 exception handling, nor are dynamically-sized arrays.

## Formatting

Although in some sense this is the least important aspect of coding style, it's the most annoying one when it goes wrong.  In general, we'd like everybody to use code formatting that is portable among different editors and IDEs.

## Indentation

Indentation is 4 spaces.  Not 4-space tabs, not sometimes 8-space tabs and sometimes 4 spaces.  4 spaces.  Nobody seems to be able to agree on what the proper tab width is, disk space is cheap, and compression works.  So please set your editor to use spaces and not tabs for indentation.  In Emacs, you can accomplish this with ''M-x set-variable indent-tabs-mode nil'', or by adding this to your .emacs file:

	
	(setq indent-tabs-mode nil)

However, most of the code contains a first line that looks like this:

	
	/* -*- c-basic-offset: 4; indent-tabs-mode: nil -*- */

which makes Emacs automatically use the proper settings.  If this line isn't in a file, please feel free to add it.

### Bracketing

There is a strong preference towards always using braces to enclose loop and conditional blocks.  This makes it easier to add things to them and prevents ambiguous readings of code.  The opening  brace should **always** go on the same line as the preceding conditional or loop statement.  In addition, we typically put the ''else'' or ''else if'' keyword on a separate line from the preceding close bracket.  This makes it easier to re-arrange conditions with cut and paste.  So, in summary, blocks should look like this:

	
	if (foo) {
	    /* something */
	}
	else if (bar) {
	    /* something else */
	}
	else {
	    /* something else */
	}

### Functions

Function ''declarations'' should have the return type on the same line as the function, like this:

	
	int foo(char *bar, int baz);

Function definitions should have the return type on a separate line as the function.  In addition, the opening brace for the function goes on a line by itself.  This makes it easy to find function definitions by using ''grep ^function_name''.  Therefore, functions should look like this:

	
	int
	foo(char *bar, int baz)
	{
	    return 42;
	}

## Declarations, Commenting, and Naming

This section describes conventions for declaring things, commenting said declarations, and naming the things in question.

### Variables

Variables should be declared in the innermost scope in which they are used.  For example:

	
	for (mgau = 0; mgau < n_mgau; ++mgau) {
	    int feat;
	    for (feat = 0; feat < n_feat; ++feat) {
	        int comp;
	        for (comp = 0; comp < n_comp; ++comp) {
	        }
	    }
	}

Variable names should be concise and lowercase, with underscores separating words.  Do **not** use [Hungarian notation](http://en.wikipedia.org/wiki/Hungarian_notation).  Although short names are encouraged, try to give variables meaningful names.

Boolean variables should be declared as ''int''.  For everything else, use the typedefs in `<prim_type.h>` (from SphinxBase).

### Functions

Function declarations in header files should always have [Doxygen](http://www.stack.nl/~dimitri/doxygen/) style comments before them (these are a lot like JavaDocs).  Although there is a good rationale for putting "inline" comments on function arguments, and much of the SphinxThree code does this, it is very ugly and is thus discouraged.  Please document whether a pointer argument is an input, output, or input-output argument.  For output and input-output arguments, it is a good idea to encode this in the name of the argument by prepending ''out_'', or ''inout_'' to its name.  Input pointer arguments should also always be ''const'' unless there is some good reason for them not to be.  So, for example:

	
	/**

	 * Do something with some stuff.
	 *
	 * @param foo The foo index.
	 * @param bar (input) Pointer to a bar index.
	 * @param out_baz (output) The baz variable is returned here.
	 * @param inout_quux (input-output) Contains the quux variable on input
	 *                   and the updated quux variable on return.
	 * @return 0 for success or -1 for failure.
	 */
	int foobie(int foo, const int *bar, int *out_baz, int *inout_quux);

Function names should also be lowercase, with underscores separating words.  If a function can be thought of as a "method" on some type, then the first word in the function name should indicate what sort of object it is associated with.  So, for example, if you have a type ''foo_t'', then its associated functions should look like:

	
	/**

	 * Construct and initialize a foo_t.
	 */
	foo_t *foo_init(void);
	/**

	 * Free a foo_t.
	 */
	void foo_free(foo_t *foo);
	/**

	 * Calculate bar using a foo_t.
	 *
	 * @return The bar value.
	 */
	int foo_bar(foo_t *foo);

## General Style Points

Please try to avoid declaring functions with lots of arguments.  There's no hard and fast rule for this but ten is almost certainly too many.  You should consider abstracting their values into an object of some sort, providing default values in its constructor, and then adding the ability to set values in it (either  by directly accessing its fields or with accessor functions).  Most of SphinxTrain violates this guideline in particularly heinous ways.

Please try to make functions concise.  If your function doesn't fit in about 80 lines of text, then consider refactoring out parts of it.  The same advice about abstracting out parameters and local variables into an object of some sort applies here as well.

Abstract types (i.e. typedefs of structures without publically visible definitions) are a good idea for public APIs.  For private APIs don't bother with them.

All rules can be broken if it makes things consistently faster, more memory-efficient, more portable, or easier to understand.  These goals are often in conflict with each other!

