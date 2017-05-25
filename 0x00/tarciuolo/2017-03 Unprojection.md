   <link rel = "stylesheet" type = "text/css" href = "../../style.css" />

Unprojection
============

I recently caught a bug where there was some texture jitter. It was
because of precision issues in world-space UVs on some geo far from the
origin. So where was the precision issue? We were using an inverse
matrix to go from device-dependent screen space (SV\_POSITION) to
world-space and by the time you pack that much math into the
concatenations and inverses the floats deteriorate. While most of the
observations below was already in the code, this round of inspection
simplified it so thoroughly I found it very informative as a reminder of
why projection matrices are set up the way they are and when you
remember *all* stages of the transformation that it all boils away back
to very simple concepts. I thought it was interesting enough to dedicate
this month's post to it. In retrospect, matrices are complicated for
what they're actually doing and the solution ended on preserve almost
all precision of user input values.

Also, every time we look at this code, it's like how the heck does this
work, where's all the robustness, the complexity?! So now I'll be able
to read my own bloggy notes here and be able to jump in right away on
related matters rather than rederiving it.

Let's take a look. Here is a version of the fixed-up, simplified version
of how we reconstruct world-space in our pixel shaders.

``` {.c++}
float3 ScreenToView(float4 sv_pos)
{
  // View-space depth is in w, so assign it.
  float3 vs_pos;
  vs_pos.z = sv_pos.w;

  // Linear view-space depth is in w. We will attenuate XY by depth for 
  // projection, but not for orthographic. Values are: (1,0) for perspective, 
  // (0,1) for orthographic.
  float atten = sv_pos.w * g_UseDepth.x + g_UseDepth.y;

  // SV_Position comes in, so convert those screen-space XYs to 
  // screen-space UVs right away.
  float2 screen_uvs = sv_pos.xy * g_ScreenToUVs;
  
  // I noticed that g_ScreenToUVs and g_ScreenToView.xy can be 
  // combined too!
  float2 vs_xy = screen_uvs.xy * g_ScreenToView.xy + g_ScreenToView.zw;

  // Scaling by depth for perspective, (atten is 1 for orthographic).
  vs_pos.xy = vs_pos * atten;
  
  return vs_pos;
}

float3 ScreenToWorld(float4 sv_pos)
{
  float3 vs_pos = ScreenToView(sv_pos);

  // For view to world, no simple proportions will save us, so run it 
  // through the inverse matrix.
  float3 ws_pos = mul(s_pos, (float3x3)g_ViewToWorld) + g_ViewToWorld[3].xyz;

  return ws_pos;
}

// Example usage:
float4 MyPixelShader(float4 sv_pos : SV_Position)
{
  float3 ws_pos = ScreenToWorld(sv_pos);
  
  return DoWorldSpaceLighting(ws_pos);
}
```

`g_ScreenToUVs` is `float2(1/screenResX, 1/screenResY)`. `g_UseDepth`
ends up being `float2(1,0)` for perspective projections and
`float2(0,1)` for orthographic projections - just a way of choosing to
attenuate by depth or not. We'll get to that, and the most interesting
`g_ScreenToView` values. A simple madd on X and Y gets us there, no need
to include an all-concatenated screen-to-view matrix.

So lets get started. The basics are that you concatenate some
local-to-world world-to-view and view-to-clip space matrices. Then you
also specify a viewport, so the HW tacks on a divide-by-w and the
viewport transform before the pixel shader receives its system value
position (SV\_POSITION). when it does this: \* View-space Z is kept in
w. I'll show this through the projection matrix and just say that HW
preserves it as well. \* Clip space is \[-w,w\], NDC space is \[-1,1\]
after HW does its w divide, then viewport transforms that into
screen-space, but we undo that with the transform to screen-uvs \[0,1\],
so I don't discuss it below.

Keeping in mind these two transforms, let's get into the projection
matrix. [Here's a D3D-style
one](https://msdn.microsoft.com/en-us/library/windows/desktop/bb205353(v=vs.85).aspx).

``` {.c++}
[  (2*n)/(r-l)       0            0       0  ]
[       0       (2*n)/(t-b)       0       0  ]
[  (l+r)/(l-r)  (t+b)/(b-t)      f/(f-n)  1  ]
[       0            0       (n*f)/(n-f)  0  ]
```

So for a view-space position XvYvZv, its clip position is:

``` {.c++}
Xc = Xv * (2*n)/(r-l)   +   Zv * ((l+r)/(l-r))
Yc = Yv * (2*n)/(t-b)   +   Zv * ((t+b)/(b-t))
// Zc = ignore this, we'll use Wc aka Zv
Wc = Zv * 1 // And here it is.
```

We can see we got Z as-is in W, and X and Y are related to a scale and
bias. First let me work only on X since Y just substitutes top and
bottom for left and right. Let's add in the rest of the transformation
from clip to screen-space (sv\_pos):

``` {.c++}
Xs = ( Xc/Wc  * 1/2 + 1/2 ) * ViewportScale
```

I know, that's not quite the order of operations, nor do I show
center-pixel stuff, but we see from the original pixel shader that we're
going to take out resolution scale anyway and leave center-pixel offsets
be.

Back to keeping it ever-simple! To get from NDC to UVs, this is what we
want:

``` {.c++}
U = ( Xc/Wc  * 1/2 + 1/2 )
```

Well let's get to solving the inverse of this... so substitute Xc and
Wc:

``` {.c++}
U = ( (Xv * (2*n)/(r-l)   +   Zv * ((l+r)/(l-r))) / Zv )  *  1/2  +  1/2

(U * 2 - 1) = (Xv * (2*n)/(r-l)   +   Zv * ((l+r)/(l-r))) / Zv  // Move NDC to screen UV 

(U * 2 - 1) = Xv/Zv * (2*n)/(r-l)   +   ((l+r)/(l-r))  // Apply division by Zv to terms

(U * 2 - 1) - ((l+r)/(l-r)) = Xv/Zv * (2*n)/(r-l) // Move the skew ratio over

((U * 2 - 1) - ((l+r)/(l-r))) * (r-l)/(2*n) = Xv/Zv // Multiply other side by inverse to move over the fov ratio

((U * 2 - 1) - ((l+r)/(l-r))) * (r-l)/(2*n) = Xv/Zv
```

An aside for multiplying out the left side of the subtraction:

``` {.c++}
(U * 2 - 1) * (r-l)/(2*n)

(U * 2 * ((r-l)/(2*n))) - ((r-l)/(2*n))

(U * ((r-l)/n)) - ((r-l)/(2*n))
```

An aside for multiplying out the right side of the subtraction:

``` {.c++}
(l+r)/(l-r) * (r-l)/(2*n)

(l+r) * 1/(l-r) * (l-r) * -1/(2*n) // flip the sign to swap r-l for l-r

(l+r) * -1/(2*n)

-(l+r)/(2*n)
```

Now back together:

``` {.c++}
Xv/Zv = (U * ((r-l)/n)) - ((r-l)/(2*n))  -  (-(l+r)/(2*n)) // pieces in position

Xv/Zv = (U * ((r-l)/n)) - ((r-l)/(2*n))  +  ((l+r)/(2*n)) = Xv/Zv  // reduce signs

Xv/Zv = (U * ((r-l)/n)) - (r+r)/(2*n)  // combine bias components and cancel l's

Xv/Zv = (U * ((r-l)/n)) - (2r)/(2*n)  // further reduce

Xv/Zv = (U * ((r-l)/n)) - r/n  // further reduce

Xv = ((U * ((r-l)/n)) - r/n) * Zv  // and of course...
```

So we've got Xv in terms of U times some value, minus some other value:
our madd! You can see we're still multiplying by view-space Z in the
pixel shader, so only include the scale and bias:

``` {.c++}
float4 g_ScreenToView( (r-l) / n, ?, -r / n, ? );
```

Do it all again for Y:

``` {.c++}
Yc = Yv * (2*n)/(t-b)   +   Zv * ((t+b)/(b-t)) // original from above

V = ( Yc/Wc  * 1/2 + 1/2 ) // screen-uvs review

V = ( (Yv * (2*n)/(t-b)   +   Zv * ((t+b)/(b-t))) / Zv )  *  1/2  +  1/2 // with the substitutions

(V * 2 - 1) = (Yv * (2*n)/(t-b)   +   Zv * ((t+b)/(b-t))) / Zv  // Move NDC to screen UV 

(V * 2 - 1) = Yv/Zv * (2*n)/(t-b)   +   ((t+b)/(b-t))  // Apply division by Zv to terms

(V * 2 - 1) - ((t+b)/(b-t)) = Yv/Zv * (2*n)/(t-b) // Move the skew ratio over

((V * 2 - 1) - ((t+b)/(b-t))) * (t-b)/(2*n) = Yv/Zv // Multiply other side by inverse to move over the fov ratio

((V * 2 - 1) - ((t+b)/(b-t))) * (t-b)/(2*n) = Yv/Zv

((V * 2 - 1) * (t-b)/(2*n))  -  (((t+b)/(b-t)) * (t-b)/(2*n)) = Yv/Zv

(V * 2 * (t-b)/(2*n)) - (t-b)/(2*n) - (((t+b)/(b-t)) * (t-b)/(2*n)) = Yv/Zv

(V * (t-b)/n) - (t-b)/(2*n) - (((t+b)/(b-t)) * -(b-t)/(2*n)) = Yv/Zv

(V * (t-b)/n) - (t-b)/(2*n) - -(t+b)/(2*n) = Yv/Zv

(V * (t-b)/n) - (t-b)/(2*n) + (t+b)/(2*n) = Yv/Zv

(V * (t-b)/n) - (t-b+t+b)/(2*n) = Yv/Zv

(V * (t-b)/n) - (2*t)/(2*n) = Yv/Zv

Yv/Zv = (V * (t-b)/n) - t/n

Yv = ((V * (t-b)/n) - t/n) * Zv
```

So finish packing the float4:

``` {.c++}
float4 g_ScreenToView( (r-l) / n, (t-b) / n, -r / n, -t / n );
```

Look at that! The float4 is composed of nothing but input parameters, so
no real modification is done to the data by the engine per-sae. Passing
the buck a bit, but compared to using the inverse clip matrix with more
math bolted in for getting out of screen-space, it's a lot simpler and
more precision-preserving.

Proprietary Notes
-----------------

I started with a D3D-style matrix because that's easily referenced and
most folks tend to think of {l,r,t,b} as distances to the edges of the
screen: the near plane.

Insomniac's projection actually removes that pesky divide-by-n since the
near clip isn't always exactly the screen anyway, so if you're being
arbitrary, why not choose a better arbitrary value, like 1?

``` {.c++}
// Insomniac projection:
// Note - n has not been pre-multiplied into l, r, t and b.  So these values 
//        represent the left, right, top, and bottom edges of the frustum at 
//        1 meter, not at n.
[      2/(r-l)       0            0       0  ]
[       0           2/(t-b)       0       0  ]
[  (l+r)/(l-r)  (t+b)/(b-t)      f/(f-n)  1  ]
[       0            0       (n*f)/(n-f)  0  ]
```

And that gets rid of n. This is all carefully set up for VR. The more
user-friendly horizontal FoV and/or vertical FoV are ratios anyway: the
same angle at 1 as at n. See [D3D's
listing](https://msdn.microsoft.com/en-us/library/windows/desktop/bb205350(v=vs.85).aspx)
how the values are not related to the near plane.

After all is said and done, we're really talking about undoing the
frustum's widening at distance into the boxy space of the screen. For
that we only need scaling by width and height, and offset by an origin.

NOTE: I may've got an axis or two flipped by showing our pixel shader
and reducing the D3D math because we also keep view-space similar to
screen-space in that we have +Y *down*.

``` {.c++}
// Note - View (rotated 180 degrees around x axis from world coords):
//
//          Z
//         /
//        /___X
//        |
//        |
//        Y
//
//
// - Post divide-by-w clip space (y is inverted):
//
//                ______(1, 1, 1)
//              /      /|
//             /_____ / |
//            |      |  |
//            |      |  |
//            |      | /
//            |______|/
//  (-1, -1, 0)
//
//
```

So our g\_ScreenToView internally is consequently:

``` {.c++}
float4 g_ScreenToView = float4( l - r, b - t, l, t );
```

Scale by width, height, offset by top-left corner for origin. So simple!

<p>-- Tony Arciuolo (Senior Engine Programmer)</p>
