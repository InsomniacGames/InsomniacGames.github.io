# Multi-Input Multi-Output Functions
Sometimes, we find ourselves in a situation where we have a function that needs to accept a number of inputs and produce a variable number of outputs per input. In the particular case where we have hundreds of inputs and a handful of outputs per input, a reliable approach likely to last a good while is to apply two complementary techniques: **sort by type** and **linear processing**. 

So, for some `MultiOp` function, we'll seperately allocate two regions of memory: one region for the outputs per input, and another for integers to count the outputs per input. We'll then process the outputs in a completely linear fashion with some help from the counters.

Let's illustrate this concretely with some code:
```cpp
Output*   outputs       = (Output*)  Allocate(max_outputs); // In most practical cases, max_outputs can be computed before processing begins
uint32_t* output_counts = (uint32_t*)Allocate(input_count);

MultiOp(output_counts, outputs, inputs, input_count);

// Future-self will thank you for this cursor
Output* output_cursor = outputs;

// Iterate the headers, one per input
for (uint32_t icount = 0; icount < input_count; ++icount)
{
  // Process the outputs for this input
  uint32_t output_count = output_counts[icount];
  for (uint32_t ioutput = 0; ioutput < output_count; ++ioutput)
  {
    // Do something (preferably awesome) with inputs[icount] and output_cursor[ioutput]
  }
  
  // Advance cursor
  output_cursor += output_count;
}
```

This approach has a few nice properties:

1. Outputs are processed purely linearly and contiguously (i.e. constant stride), yielding optimal performance on that front
2. Debugging most issues is relatively straightforward, especially compared to a "packetized stream" approach
3. Can be expected to perform better than a callback-based approach
  * With a C++11 lambda, this can depend on how/if a the lambda or static function gets inlined into the callsite
4. Allocations required are temporary and their lifetime is clear
5. Variations to address many circumstances (e.g. sparse output, where only a few inputs get *any* outputs) are relatively simple to make and remain performant

It also has a few downsides:

1. For various reasons often outside of our control, the memory required for `outputs` can get large
2. Related to the above, `max_outputs` can sometimes be well above what "`typical_outputs`" would be
3. Memory overwrite bugs are still a pain to debug compared to some other approaches
4. Some less-common circumstances (e.g. multiple heterogeneous output types) are less straightforward to accomodate

In most cases, though, the downsides are well worth the upsides, especially compared to the live alternatives, and it's pretty easy to implement and get right to boot!
