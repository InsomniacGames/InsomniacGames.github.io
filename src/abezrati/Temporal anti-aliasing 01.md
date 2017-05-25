Temporal anti-aliasing attempts to address artifacts related to screen resolution and high-frequency material shading by accumulating and then presenting the weighted colors of many frames.

To reduce memory cost, we only keep track of a single history buffer and progressively add weighted content to it.

We keep track of pixels previous location by using a velocity buffer. The accuracy of the velocity buffer is paramount to minimizing errors thus we use a two channels FP16 target.

Some other studios make use of a lower precision target in conjunction with some clever encoding scheme.

The weights of the current frame colors depend on many factors: For instance pixels that generate neither depth nor velocity, such as the case of particle effects, tend to be strongly biased towards most recent frames.

Our current approach to differentiate blended scene objects from the opaque elements is to use the stencil buffer. Later in the post-processing stage we load the stencil bits and factor them in the final pixel color weights.

The stencil approach is a bit heavy handed because it doesn’t take the different levels of transparency into account. A better approach might be to accurately track alpha during the blended pass.
