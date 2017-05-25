This post is going to summarize a method of compressing textures that allows the data can be quickly unpacked on the GPU into a block compressed format. The general goal is similar to Crunch, though with this method, the decompression can be performed efficiently on the GPU. The information in this post is based on the paper GST: GPU-decodable Supercompressed Textures. The paper is available here: [http://gamma.cs.unc.edu/GST/](http://gamma.cs.unc.edu/GST/)

The first stage of compression generates endpoints and indices for each block in the texture. This is similar to normal block compression and an existing block compressor can be used for this purpose.

The next step generates a dictionary of index blocks from the indices in the previous step. When adding blocks to the dictionary, a block is not added if it is within a given error threshold of another block. The index into the dictionary is represented as a delta relative to the previous block. Rather than searching the entire dictionary, there is an upper bound on the number of blocks to search. This reduces the number of bits needed to encode the block’s dictionary entry.

After the dictionary of index blocks is generated, the endpoints for each block are compressed. The first step in this process is transforming the RGB endpoints to YCoCg. This improves the redundancy of neighboring pixels. The endpoint planes then have a wavelet transform applied.

The final portion of encoding involves combining the data streams from the previous steps using an entropy encoding method known as Asymmetric Numeral Systems. The data streams contain the endpoints Y, Co and Cg planes as well as the dictionary entry deltas and the index dictionary. Using ANS, this data can be decoded in parallel using SIMD instructions.

Sample code is available here: [https://github.com/GammaUNC/GST](https://github.com/GammaUNC/GST)

