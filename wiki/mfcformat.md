---
layout: page 
title: MFC files
---

CMUSphinx stores features in files with the extension .mfc. Please note that 
during training and decoding
we usually use static features like mel-cepstrum and dynamic features like 
mel-cepstrum deltas or delta-deltas. On the filesystem we store only static 
features and dynamic features are computed on the fly. The computatoin is 
configured with the `-feat` option. For example `-feat 1s_c_d_dd` means to 
read the vector and compute deltas and delta-deltas and combine them with 
1-stream feature vector. There are different types like 's2_4x' which means to 
compute deltas, delta-deltas, delta-deltas of the second order and combine them 
in a special 4-stream feature vector. If you need a specific feature 
arrangement you can implement your own feature type in sphinxbase. If you want 
to use features as is use `1s_c` feature type which means to read the vector 
unmodified.

The file stores the values in binary format and each value is stored as a float 
i.e. 4 bytes. The first 4 bytes in the file specify the header which is nothing 
but the total number of values stored. For example, if we have N number of 
frames, and let’s assume each frame is represented as a M (usually 13) 
dimensional vector. So, the header which is basically the total number of 
distinct values in feature matrix is N*M. Note that it is stored as an int32. 
The feature vectors will be stored according to the column number or frame 
number. So, 1st frame (M values in total) will be stored first, followed by 
second and so on.

It looks like

	
	header (int32 length)
	features of the frame 1 (13 floats or 13 * 4 bytes)
	features of the frame 2 (13 floats or 13 * 4 bytes)
	features of the frame ... (13 floats or 13 * 4 bytes)
	features of the frame N (13 floats or 13 * 4 bytes)


If you need to configure the vector length in trainer you can use -veclen 
option which works with almost all binaries. It's also possible to configure 
this globally for the training database in sphinx_train.cfg file.

If you want to implement your own feature type you need to write a tool to dump 
features in the format above. You can easily do that given you know basics of 
the programming.

