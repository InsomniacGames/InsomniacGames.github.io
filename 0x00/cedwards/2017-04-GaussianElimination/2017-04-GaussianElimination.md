<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <script type="text/javascript" async src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
    </script>
</head>

<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    TeX: { extensions: ["AMSsymbols.js"] },
    tex2jax: {
      inlineMath: [ ['$','$'], ["\\(","\\)"] ],
      processEscapes: true
    }
  });
</script>

Gaussian Elimination
====================
A system of linear equations can be solved using augmented matrices and a process called Gaussian
Elimination.  If you have a linear system of equations, an *augmented matrix* is created by taking
each equation and placing the coeffcients and result into a single row in a matrix.  

For example, if you have this system of linear equations:
`$$4x + 14y + 12z = 26$$`
`$$-6x - 17y - 11z = -55$$`
`$$x + 3y + 2z = 8$$`

The augmented matrix would be:
`$$
  \left[\begin{array}{rrr|r}
     4  &  14  &  12  &  26 \\
    -6  & -17  & -11  & -55 \\
     1  &   3  &   2  &   8
  \end{array}\right]
$$`

The goal is to do a series of operations on the augmented matrix that convert it into *reduced row
echelon form*.  In this form, the diagonals of the matrix are $1$ and zeroes are found below the
diagonal.
`$$
  \left[\begin{array}{rrr|r}
  1 & a_1 & a_2 & b_0 \\
  0 & 1   & a_5 & b_1 \\
  0 & 0   & 1   & b_2
  \end{array}\right]
$$`

It should be clear that once you have the matrix in this form, you have solved for one of your
unknowns.  The last row contains the value for $z$.  The second row contains a formula in terms
of $y$ and $z$ so you can plug in the value for $z$ and solve for $y$.  Finally, your first row
is an equation that contains three unknowns, but you can now solve for $x$ by plugging in the
values of $y$ and $z$.


The way you convert an arbitrary augmented matrix into reduced row echelon form is by performing
a series of row operations that do not change the relationship between the equations.  These 
elementary operations are:
1. Swap one row with another
2. Scale (multiply) all elements of the row by a value
3. Replace a row by adding it to a scaled version of another row

Using the system of linear equations above, we start with an augmented matrix.
`$$
  \left[\begin{array}{rrr|r}
     4  &  14  &  12  &  26 \\
    -6  & -17  & -11  & -55 \\
     1  &   3  &   2  &   8
  \end{array}\right]
$$`

To get a $1$ into the first column of the first row, we swap the first and last rows.
`$$
  \left[\begin{array}{rrr|r}
     1  &   3  &   2  &   8 \\
    -6  & -17  & -11  & -55 \\
     4  &  14  &  12  &  26 
  \end{array}\right]
$$`

Now we want to get zeroes in the first column of the remaing rows.  This is accomplished by scaling
the first row and adding it to each of the others.  For starters, we scale the first row by $6$ so 
that it becomes `$\left[\begin{array}{rrr|r}6 & 18 & 12 & 48 \end{array}\right]$`.  When 
added to the second row and replaced, we get:
`$$
  \left[\begin{array}{rrr|r}
     1  &   3  &   2  &   8 \\
     0  &   1  &   1  &  -7 \\
     4  &  14  &  12  &  26 
  \end{array}\right]
$$`

Similarly, we scale the first row by $-4$ and add it to the last row to get a zero into the first
column of the last row:
`$$
  \left[\begin{array}{rrr|r}
     1  &   3  &   2  &   8 \\
     0  &   1  &   1  &  -7 \\
     0  &   2  &   4  &  -6 
  \end{array}\right]
$$`

Now we move on to the second row, second column.  There is already a $1$ in this slot, so we don't
need to adjust this row at all.  To get a zero in the second column of the third row, we need to 
scale the second row by $-2$ and add it to the last row:
`$$
  \left[\begin{array}{rrr|r}
     1  &   3  &   2  &   8 \\
     0  &   1  &   1  &  -7 \\
     0  &   0  &   2  &   8 
  \end{array}\right]
$$`

Finally, the last row needs to have a $1$ in the third column.  This can be accomplished by scaling
the whole row by $1/2$:
`$$
  \left[\begin{array}{rrr|r}
     1  &   3  &   2  &   8 \\
     0  &   1  &   1  &  -7 \\
     0  &   0  &   1  &   4 
  \end{array}\right]
$$`

The augmented matrix is in reduced row echelon form, and we have solved one of our unkowns.
`$$ z = 4 $$`

We also have a new equation for $y$ in terms of a single unknown, $z$.

`$$ 
  \begin{array}{rcl}
      y + z & = & -7 \\
          y & = & -7 - z \\
            & = & -7 - 4 \\ 
            & = & -11
  \end{array}
$$`


We have a new equation for $x$ as well, and since we know $y$ and $z$, we can solve for the last
unknown:

`$$ 
  \begin{array}{rcl}
      x + 3y + 2z & = & 8 \\
                x & = & 8 - 3y - 2z \\
                  & = & 8 - 3(-11) - 2(4) \\ 
                  & = & 33
  \end{array}
$$`

Note that it's possible for there to be no solution to a system of linear equations.  This happens
when one of the columns on the left side of the augmented matrix contains all zeroes.

Further Reading
---------------

Van Verth, James M., Lars M. Bishop. "Linear Transformations and Matrices." 
*Essential Mathematics for Games and Interactive Applications.* 3rd ed.
Boca Raton: CRC, 2016. Web. ISBN 978-1482250923.