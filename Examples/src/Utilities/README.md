#Various Utilities#

In this directory you have utilities thay may be included and used in
other examples. So, you need to install them before moving to the other
examples.

You need to do

    make 
    make install

to install them. `make` will produce both a dynamic and a static library, called `libpacs.so` and `libpacs.a`, respectively. `make install` installs the header files in `PACS_ROOT/include` and the libraries into `PACS_ROOT/lib`. Use `make DEBUG=no` if you want the library code to be optimised.

After you have installed the libraries you can run

    make test

which produces tests for most of the utilities. All tests start with `test_`. You may have a look at the source to see what they do.

*Remember to install the utilities, they are used by other examples!.*

**Note**: some utilities are in a nested namespace of the namespace `apsc`. Check it out looking at the code or at the examples.

List of the utilities:

* `chrono`  An utility to take times, built on the chrono utilities of the standard library.

* `CloningUtilities` Tools for clonable classes (Prototye design pattern). It contains some type traits to test if a class T containes the (usually virtual) method

```
std::unique_prt<B> clone() const;
```

that returns the pointer to a copy of the object wrapped into a
unique_prt.  B can be either T or a base class of T. It also contains
an interesting class, called `PointerWrapper`, which implements an owning pointer with deep copy
semantic! The "pointed" class should be clonable. It means that you can use it to implement composition of a polymorphic object and have autometically the copy operators in the composing class!

* `cxxversion` To test with which version of C++ you are compiling your code

* `extendedAssert`  Asserts with a message. It extends assert macro so that you can insert a message. There are also swithches that can be activated with the `-DXXX` compiler option to change the behaviour of some of them.

* `Factory`  A generic object factory. Inspired by a code by [Andrei Alexandrescu](https://en.wikipedia.org/wiki/Andrei_Alexandrescu). 

* `GetPot`  GetPot command parser<http://getpot.sourceforge.net/>. I have simplified the version available on the given link, so you have just to so `#include <GetPot>` of `#include "GetPot"`, and have it available.

* `gnuplot-iostream` A stream to open gnuplot from within a program. Useful for simple visualizations within your code. You need[gnuplot](http://www.gnuplot.info/) installed in your system (it is available as debian package).

* `hashCombine.hpp` Provides the function object `hash_combine` that may be used to combine the hash key of object of different types, provided the latter have `std::hash` defined. It can be used to creat the hash key of an user-defined class by combining that of non-static members of the class, in order to achieve better uniformity. The usage is explained in the file.

* `is_complex.hpp` A header file containing a type trait to interrogate is a type is a `std::complex<T>`

* `is_eigen.hpp` A header file containing a type trait to interrogate is a type is a `Eigen::Matrix`


* `JoinVectors.hpp` Just an example on how to imitate the `join` phyton command. You can use it to iterate jointly on a set of vectors. 

* `Proxy.hpp` It is not a proxy (bad naming, sorry). It is an utility that may be used to register objects in an object Factory automatically.

* `readCSV` A class to read csv files. Useful if you have data in a speadsheet and you want to load it into a C++ code. There are better tools than this one around. But this is relativley simple and handy

* `scientific_precision` A function that sets the precision of a stream to the maximum value for a floating point. It contains also stream manipulators for the same purpose.

* `setUtilities` Three utilities to simplify operations on sets (union/difference/intersection)
represented by an ordered container. They are built on top of the analogous
utilities of the Standard Library, but with a simpler interface. 

* `StatisticsComputations` Some tools to compute basic statistics of a sample.

* `string_utility` Some extra utilities for strings: trimming (eliminate useless blanks) and lower-upper conversion. We have recetly added utilities for reading a whole text file in a buffer (it is faster, though potentially memory consuming, and an utility that computed the Levenshtein edit distance between two strings).

* `toString` Converts anything for which there is the `<<` streming operator to a string. A use of `std::stringstream`.

* `overloaded` A facility, called `overloaded` that implements the overloaded design pattern that may be used to visit a std::variant.


** Note ** `Factory.hpp` and `Proxy.hpp` are in fact links to the same file in the folder `GenericFactory`. If the files are not present for some reason you may safely copy in `Utility/` the files in `GenericFactory/`.


## An explanation of `hash_combine`.##
The functor `apsc::hash_combine` provided by  `hashCombine.hpp` is given in two formats, one used *fold expressions* introduced in C++17, the other requires just c++11 to work. Since they are short but complicated it is worthwile giving some explanation.
We recall that a good hash function should satisfy as far as possible the uniformity property: the probability distribution of the hash keys (conditioned to the distribution of the possible arguments) should be uniform. Since we do not normally know the distribution of the arguments, it is normally assumed that it is uniform as well.
This is not so easy to achieve and a badly unbalanced hash function may make your unordered container very inefficient! That's why a lot of "tricks" are used to increase entropy and avoid "clustering".

Here the c+=17 version of my `hash_comine` (taken from the web, I thank the unknown author):

	template <typename T, typename... Rest>
	void hash_combine(std::size_t& seed, const T& v, const Rest&... rest)
	{
	   std::hash<T> hasher;
        seed ^= hasher(v) + 0x9e3779b97f4a7c15 + (seed << 6) + (seed >> 2);
        (hash_combine(seed,rest), ...);
	}

Note that **`seed` is passed by reference**, and indeed it will finally contain the hash key! See the documentation in `hashCombine.hpp` or `test_hash_combine.cpp` for the way to use this utility for the construction of the hash function for your class.

We use some unusual operators: `operator^()` is a binary operator that performs a bit-wise exclusive or (xor). That is if we have two integers, `a` and `b`, whose binary representation is `a=0b1001` and `b=0b1100`, we have `a^b=0b0101` (The `0b` indicates that what follows is a binary literal). You can compare with the result of the two other binary bit-wise logical operators: `a|b=0b1101` (bit-wise or) and `a&b=0b1010` (bit-wise and). In this context, bit-wise xor is used to introduce a bit of entropy. 
Then, we add the "magic number" `0x9e3779b9` (`0x` indicates that is is in hexadecimal format). It is the integral part of the Golden Ratio's fractional part `0.61803398875…`  multiplied by `2^64`. Adding it has a scattering effect, often referred to as "Golden Ratio Hashing", or "Fibonacci Hashing" and was popularised by Donald Knuth (The Art of Computer Programming: Volume 3: Sorting and Searching). If you are interested, in number theoretical terms it is related to the Steinhaus Conjecture. 

After doing that, we have the `<<` and `>>` operators. These are the left and right *bit-shift* operators that take an number to shift and an unsigned integer indicating the number of shifts. To understand how it works let assume that the variable `a=0b1001` and `b` defined above is just a 4-bit integer (to make things simpler). We have
`(a<< 1)=0b0010`, `(a<< 2)=0b0100`, and `(a>>1)=0100`, `(a>>2)=0010`. I hope it is clear. Again, all this fuss is to scatter the digits around (in fact the bits).

Finally, I am using here a fold expression in `(hash_combine(seed,rest), ...);` to expand the variadic template. It means that, for intance,
`hash_combine(0,a,b,c)` expands in

	std::hash<T> hasher;
	seed ^= hasher(a) + 0x9e3779b97f4a7c15 + (seed << 6) + (seed >> 2);
	hash_combine(seed,b);
	hash_combine(seed,c);

thanks to the magic of a fold expression. The pre-c++17 version achieves fold expression with a dirty tick that I avoid explaining (after all c++17 is now well extablished).

## What you can learn from these examples##

The files in this directory illustrate the advantage of having little general utilities that can be integrated in different codes.
Some utilities are simple to understand, like `Chrono` or `StatisticsComputations`, others make use of more sophisticated generic programming 
techniques, like `Factory`, or template metaprogramming, like `is_complex`, `is_eigen`, `joinVectors` and `CloningUtilities`. 
Finally, some, like `gnuplot_iostream` and `GetPot` are just copies of tools available open source, copied here for simplicity.

In `CloningUtilities` and `joinVectors` you have classes that define a dereferencing operator (`*`) and an access via pointer operator
(`->`). Something you do not find very often.

In `hashCombine.hpp` the use of some bit-wise operators and fold expression.


