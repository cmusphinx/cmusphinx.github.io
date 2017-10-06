---
layout: page 
title: MFC files
---

CMUSphinx stores features in binary files with the extension .mfc. The
format of the file is explained below. You can explore the contents of
the MFC files with the `sphinx_cepview` tool available in sphinxbase.

```
$ sphinx_cepview -f goforward.mfc 
Current configuration:
[NAME]		[DEFLT]		[VALUE]
-b		0		0
-d		10		10
-describe	0		0
-e		2147483647	2147483647
-f				goforward.mfc
-header		0		0
-i		13		13

INFO: main_cepview.c(152): Displaying 10 out of 13 columns per frame
INFO: main_cepview.c(153): Total 264 frames

 26.778  -9.018  -4.308   2.861   2.228  -1.276  -4.449   0.619  10.228   5.591 
 26.250  -8.467  -4.236  -5.483   3.638   7.682   0.225   0.725  -0.704  -5.131 
 26.233  -9.644  -3.705 -12.095   0.784  -7.368   5.375   1.682  -6.990   0.190 
 27.002  -6.852  -2.884  -6.710  -3.930  -4.984  -1.931  -0.153  -1.353   1.334 
 26.276  -9.429  -3.457  -0.386  -5.470  -5.091  -7.974 -12.054 -10.212  -1.914 
 23.995 -10.228  -9.694 -10.605  -6.520  -3.021   5.137   3.212  -0.980  -8.479 
 24.809  -8.962  -8.389  -8.554  -8.594  -3.912   3.560   6.332   2.514  -4.706 
 24.199  -7.221  -1.295  -7.918  -6.166  -3.673  -0.319   2.275 -14.336 -12.722 
 24.399 -11.508  -6.442 -10.584  -2.175  11.163  13.581   4.309   3.131 -11.000 
....
```

For IO efficiency we store only static features and dynamic features are
computed on the fly. The computation of dynamic features is configured
with the `-feat` option. For example `-feat 1s_c_d_dd` means to read
the vector and compute deltas and delta-deltas and combine them with 
1-stream feature vector. There are different types like `s2_4x` which
means to  compute deltas, delta-deltas, delta-deltas of the second order
and combine them  in a special 4-stream feature vector. If you need a
specific feature  arrangement you can implement your own feature type in
sphinxbase. If you want  to use features as is use `1s_c` feature type
which means to read the vector  unmodified.

The file has the values in binary format and each value is stored as a
float i.e. 4 bytes. The first 4 bytes in the file specify the header
which is nothing but the total number of values stored. For example, if
we have N number of frames, and let’s assume each frame is represented
as a M (usually 13) dimensional vector. So, the header which is
basically the total number of  distinct values in feature matrix is N*M.
Note that it is stored as an int32. The feature vectors will be stored
according to the column number or frame  number. So, 1st frame (M values
in total) will be stored first, followed by second and so on. The
dimension is not stored in a file but assumed. So if dimension
mismatches bad things could happen.

So the layout looks like

	header (int32 length)
	features of the frame 1 (13 floats or 13 * 4 bytes)
	features of the frame 2 (13 floats or 13 * 4 bytes)
	features of the frame ... (13 floats or 13 * 4 bytes)
	features of the frame N (13 floats or 13 * 4 bytes)

If you need to configure the vector length in trainer you can use
`-veclen` option which works with almost all binaries. It's also
possible to configure this globally for the training database in
`sphinx_train.cfg` file.

If you want to implement your own feature type you need to write a tool
to dump features in the format above. You can easily do that given you
know basics of the programming. For example you can use a Python script
or Matlab/Octave.

For example, here is how one can write mfc file from Python:

```
    def writeheader(featsize):
        fh.seek(0,0)
        fh.write(pack("=i", featsize))

    def writevec(vec):
        fh.write(pack("=" + str(13) + "f", *vec))

    def writeall(arr):
	writeheader(len(arr) * 13)
        for row in arr:
            fh.writevec(row)
```

or in Matlab

```
	fid = fopen(filename, 'wb');
	fwrite(fid, featsize, 'int32');
	count = fwrite(fid, featarray(:), 'float32');
	fclose(fid);

```
