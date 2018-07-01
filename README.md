# intel-intrinsics

![Travis Status](https://travis-ci.org/AuburnSounds/intel-intrinsics.svg?branch=master)
This package allows you to use Intel intrinsics in D code.

## Usage

```d

version(LDC)
{
    import inteli.xmmintrin; // allows SSE1 intrinsics
    import inteli.emmintrin; // allows SSE2 intrinsics

    // distance between two points in 4D
    float distance(float[4] a, float[4] b) nothrow @nogc
    {
        __m128 va = _mm_loadu_ps(a.ptr);
        __m128 vb = _mm_loadu_ps(b.ptr);
        __m128 diffSquared = _mm_sub_ps(va, vb);
        diffSquared = _mm_mul_ps(diffSquared, diffSquared);
        __m128 sum = _mm_add_ps(diffSquared, _mm_srli_si128!8(diffSquared));
        sum = _mm_add_ps(sum, _mm_srli_si128!4(sum));
        return _mm_cvtss_f32(_mm_sqrt_ss(sum));
    }
    assert(distance([0, 2, 0, 0], [0, 0, 0, 0]) == 2);
}

```

## Why?

### Familiar syntax

Why Intel intrinsic syntax? Because it is more familiar to C++ programmers 
and there is a convenient online guide provided by Intel: 
https://software.intel.com/sites/landingpage/IntrinsicsGuide/

Without this guide it's much more difficult to write sizeable SIMD code.

### Future-proof

LDC SIMD intrinsics are a moving target (https://github.com/ldc-developers/ldc/issues/2019), 
and you need a layer over it if you want to be safe.

We maintain that layer because we need it for our products.

Because those x86 intrinsics are internally converted to IR, **they don't tie to a particular architecture**.
So you could target ARM one day and still get some speed-up.


### Portability

**For now only LDC is supported**, but in the future the same set of intrinsics will work with DMD too. 
This is intended to be the most practical SIMD solution for D. 
Including an emulation layer for DMD 32-bit which doesn't have any SIMD capability right now.


### Supported instructions set

- SSE1
- SSE2

The lack of AVX intrinsics is explained by the lack of raw speed gain with these instruction sets.