LIBTWIDDLE
==========
[![Build Status](https://travis-ci.org/fsaintjacques/libtwiddle.svg?branch=develop)](https://travis-ci.org/fsaintjacques/libtwiddle)

libtwiddle is a small library that implements bit data structures:

  * bitmaps
  * bloom filters
  * hyperloglog

USAGE
=====

bitmap
------

```C
#include <assert.h>
#include <twiddle/bitmap/bitmap.h>

int main(int argc, char* argv[]) {
  const uint64_t nbits = 1024;
  struct tw_bitmap* bitmap = tw_bitmap_new(nbits);

  assert(bitmap);

  tw_bitmap_set(bitmap, 512);
  assert(tw_bitmap_test_and_clear(bitmap, 512));
  assert(!tw_bitmap_test(bitmap, 512));

  tw_bitmap_set(bitmap, 768);
  assert(tw_bitmap_find_first_bit(bitmap) == 768);

  return 0;
}
```

bitmap-rle
------

```C
#include <assert.h>
#include <twiddle/bitmap/bitmap_rle.h>

int main(int argc, char* argv[]) {
  /** allocate a bitmap containing 2 billions bits */
  const uint64_t nbits = 1UL << 31;
  struct tw_bitmap_rle* bitmap = tw_bitmap_rle_new(nbits);

  assert(bitmap);

  /** fill 1 billion bits */
  const uint64_t start = 0UL;
  const uint64_t end = 1UL << 30;
  tw_bitmap_rle_set_range(bitmap, start, end);

  /**
   * bitmap_rle DOES NOT support setting bits in non sorted order, e.g.
   * calling the following will raise an assert() call (or undefined behaviour
   * if compiled with NDEBUG):
   *
   * tw_bitmap_rle_set(bitmap, start - 1);
   */

  assert(tw_bitmap_rle_test(bitmap, start));
  assert(tw_bitmap_rle_test(bitmap, end));
  assert(tw_bitmap_rle_find_first_bit(bitmap)  == start);
  assert(tw_bitmap_rle_find_first_zero(bitmap) == end + 1);

  return 0;
}
```

bloomfilter
-----------

```C
#include <assert.h>
#include <string.h>

#include <twiddle/bloomfilter/bloomfilter.h>

int main(int argc, char *argv[]) {
  const uint64_t nbits = 1024;
  const uint16_t k = 7;
  struct tw_bloomfilter *bf = tw_bloomfilter_new(nbits, k);
  assert(bf);

  char *values[] = {"herp", "derp", "ferp", "merp"};

  for (int i = 0; i < ((sizeof(values) / sizeof(values[0]))); ++i) {
    tw_bloomfilter_set(bf, strlen(values[i]), values[i]);
    assert(tw_bloomfilter_test(bf, strlen(values[i]), values[i]));
  }

  assert(!tw_bloomfilter_test(bf, sizeof("nope"), "nope"));

  return 0;
}
```

hyperloglog
-----------

```C
#include <assert.h>
#include <stdio.h>

#include <twiddle/hyperloglog/hyperloglog.h>

int main(int argc, char *argv[]) {
  const uint8_t precision = 16;
  struct tw_hyperloglog *hll = tw_hyperloglog_new(precision);
  assert(hll);

  const uint32_t n_elems = 10 * (1 << precision);
  for (int i = 0; i < n_elems ; ++i) {
    tw_hyperloglog_add(hll, sizeof(i), (char *) &i);
  }

  printf("estimated count: %f, real count: %d\n",
         tw_hyperloglog_count(hll),
         n_elems);

  return 0;
}
```

INSTALL
=======

libtwiddle uses cmake as its build manager.

Prerequisite libraries
----------------------

To build libtwiddle, you need the following libraries installed on
your system:

  * pkg-config
  * check (http://check.sourceforge.net)

Building from source
--------------------

In most cases, you should be able to build the source code using the following:

    $ mkdir build
    $ cd build
    $ cmake .. -DCMAKE_INSTALL_PREFIX=$PREFIX
    $ make
    $ make test
    $ make install
