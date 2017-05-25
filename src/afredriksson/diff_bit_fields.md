When you're isolating bit fields and need both the "before and after", that is, both the isolated bits and the remaining data with a hole full of zeroes where the bits were, a common approach is this:

    uint32_t value = ...;
    uint32_t a = value & kFieldMask;     // Isolate 'A' bits
    uint32_t b = value & ~kFieldMask;    // Isolate the other bits

There is an alternate way to express the same thing that can sometimes come in handy:

    uint32_t value = ...;
    uint32_t a = value & kFieldMask;     // Isolate 'A' bits
    uint32_t b = value ^ a;              // Isolate the other bits
  
This exploits the fact that the xor of `value` with `a` will just flip any 1-bits we already isolated in the first logical and.

Mostly this is useful as an optimization for code size, because you avoid storing both `kFieldMask` and `~kFieldMask` in the instruction stream. However, the two-mask version probably executes faster on modern CPUs because there is more inherent instruction-level parallelism in that version.