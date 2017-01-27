Floating Point
==============

The IEEE 754 floating-point standard defines a single-precision (32-bit) number as being 
represented by:

|Sign |Exponent|Mantissa|
|-----|:-------|:-------|
|1 bit|8 bits  |23 bits |

To convert a number to this format, it is put into normalized binary scientific notation.
Normalized means that there is a one in the first position:

(+/-)1.Mantissa x 2^e^

In the standard, e has a bias of 127, so you calculate the actual exponent to be the value of 
these 8 bits minus 127.  At first blush, the range of the exponent would appear to be:

[0 - 127, 255 - 127] = [-127, 128]

But hold that thought.

Because the format stipulates that there is a one in the first position, there is no need 
to store it in the representation.  The one is a *hidden bit* that is assumed to be 
there.  But this means you wouldn't be able to represent zero, at least not without some 
more explanation.  So, there are some special case values reserved in the format, giving
two representations for zero, two for infinity, and few flavors of *Not a Number*:

|Sign   |Exponent|Mantissa               |=    |
|:-----:|:------:|:---------------------:|:---:|
| 0     |00000000|00000000000000000000000|+0   |
| 1     |00000000|00000000000000000000000|-0   |
| 0     |11111111|00000000000000000000000|Inf  |
| 1     |11111111|00000000000000000000000|-Inf |
| 0     |11111111|(not all zero)         |NaN  |

These reserved values use up an entry at either end of the exponent range, so the range is
actually [-126, 127].

Hole at Zero
------------

Examine the smallest possible value after zero and the next smallest value using what
has been described so far:

|Sign   |Exponent|Mantissa               |Scientific            |=                |Distance|
|:-----:|:------:|:---------------------:|:--------------------:|:---------------:|:------:|
| 0     |00000000|00000000000000000000000|-                     |+0               |-       |
| 0     |00000001|00000000000000000000000|2^0^ x 2^-126^        |2^-126^          |2^-126^ |
| 0     |00000001|00000000000000000000001|(2^0^+2^23^) x 2^-126^|2^-126^+2^-149^  |2^-149^ |
|       |        |                       |                      |                 |        |

Because of the *hidden bit* (2^0^ = 1), we have lost precision between zero and the next smallest 
representable number.  You can see this by looking at the distance between each of the numbers in 
the sequence.  As shown above, the distance between zero and the next smallest number is larger 
than the distance between the smallest non-zero number and the next one.  This gap is called the 
*hole at zero*.  It means that there is less accuracy in the numeric representations near zero.


Denormals
---------

The IEEE standard fills in the *hole at zero* by reserving another special case.  It allows for
storing some values that are not normalized.  These are called *denormals*.  Since these numbers
are not normalized, the *hidden bit* is assumed to be zero for these numbers instead of one.

*Denormals* are indicated by values would appear to have an exponent of 127, but they now have a
special meaning.  Normally, an exponent of all zero bits would be caluclated as:

e = 0 - 127 = -127.  

But the special meaning in the standard stipulates that the exponent is actually assigned to -126.  

|Sign   |Exponent|Mantissa               |=            |
|:-----:|:------:|:---------------------:|:-----------:|
| 0     |00000000|(not all zero)         |+ denormal   |
| 1     |00000000|(not all zero)         |- denormal   |

By forcing the exponent to be calculated as -126 and the *hidden bit* to be zero, this allows the 
standard to represent values near zero with increased accuracy.

Further Reading
---------------
Goldberg, David. "What Every Computer Scientist Should Know About Floating-Point Arithmetic." 
*Computing Surveys*
March 1991: 171-264. validlab.com/goldberg/paper.ps. Web. 23 Jan. 2017.

Van Verth, James M., Lars M. Bishop. "Representing Real Numbers." 
*Essential Mathematics for Games and Interactive Applications.* 3rd ed.
Boca Raton: CRC, 2016. Web. ISBN 978-1482250923.