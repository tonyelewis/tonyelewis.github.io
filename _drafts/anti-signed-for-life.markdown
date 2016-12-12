---
categories : jekyll update
date       :   2016-11-15 17:32:44 +0000
layout     : post
permalink  : anti-signed-for-life
title      : "Anti: Signed for Life"
---
<style type="text/css">
.notes {
	color        : grey;
	font-size    : x-small;
	padding-left : 50px;
}

table {
	border-color : #e8e8e8;
	border-style : solid;
	border-width : 1px;
	margin-bottom: 25px;
}

th {
	padding: 0 20px;
}
</style>

{::options parse_block_html="true" /}

<div class="notes">
Anti: Signed for Life

Add:
 * throwing the baby out and leaving half the bathwater
 * alignof() / sizeof() / sizeof...()

"unsigned: A Guideline for Better Code", Jon Kalb CppCon 2016

The standard library gave us power,<br>
Then new() came and replaced free(). <br>
What price now, <br>
For a shallow piece of dignity?

GCC/Clang feature requests?

MSVC options?

</div>

This is a response to Jon Kalb's Lightning Talk at CppCon 2016, [unsigned: A Guideline for Better Code](https://www.youtube.com/watch?v=wvtFGa6XJDU). It's a really interesting talk so please watch it now if you haven't already.

Jon argues that we should avoid using unsigned integer types in C++ (at least for quantities; he accepts them for bit-masks). I was particularly interested in Jon's talk because I looked into this issue a few years ago and I ended up coming to different conclusions. So I'd like to explain here why Jon's talk hasn't convinced me and why I still think you should use unsigneds.

Caveats
===

I should acknowledge upfront: as I understand it, the strong majority of C++ experts' opinion is against unsigneds. For example: Jon's someone who really knows what he's talking about, the [C++ core guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#a-nameres-signedaes102-use-signed-types-for-arithmetic) are with him, the [Google C++ style guide](https://google.github.io/styleguide/cppguide.html#Integer_Types)'s with him, etc, etc.

So I recognise the overwhelming likelihood that I'm wrong on this. But still, there are sensible responses to make to Jon's talk so I think it's worth someone making those arguments, if only so that someone else can then shoot them down and thus help us non-experts see why they're wrong. This issue seems fundamental enough that its worth the C++ community sharing clarity on.

I'm also aware that some aspects of this could slip into time-wasting, unresolvable aesthetic disputes (like whitespace-wars). Still, I think other aspects have real consequences for correctness etc and are genuinely worth discussing.


Plot spoiler
===

I'm going to make two main points:

1. The problems Jon highlights are better dealt with by compiler errors than by dropping unsigneds
1. Once we've done that, the benefits of unsigneds outweigh the remaining costs

The problem
===

Let's start with an example:

{% highlight cpp %}
void f(const unsigned int &x) {
  cout << x << "\n";
}

int main() {
  f( -1 );
}
{% endhighlight %}

Here, we've got a function {% ihighlight c++ %}f{% endihighlight %} that takes an {% ihighlight c++ %}unsigned int{% endihighlight %} and then we're calling with -1. This invokes a silent conversion of -1 to an {% ihighlight c++ %}unsigned int{% endihighlight %}, which turns it into a huge positive number, which is what then gets printed.

Another example:


{% highlight cpp %}
unsigned int one_unsigned = 1;

if ( -1 < one_unsigned ) /* ... */
{% endhighlight %}

Here, we'd expect the expression in brackets to evaluate to true because -1 *is* less than 1,  but it doesn't because the less-than comparison again invokes a silent conversion from -1 to an {% ihighlight c++ %}unsigned int{% endihighlight %}, which again, turns it into a huge positive number.

The wrong response
====

Jon's central argument is that to avoid this sort of surprising and dangerous behaviour, we should avoid unsigned types in our code.

But I think that's the wrong response. It's neither necessary nor sufficient. 

It isn't sufficient because most of our code bases will still have plenty of interactions with unsigneds. They're in the standard library, they're in plenty of other libraries and they may well be in future contributions made to any given code base.

For example, if you do this:

{% highlight cpp %}
vector<int> a = { 1, 2, 3 };
( -1 < a.size() );
{% endhighlight %}

...you still get the same surprising result because {% ihighlight c++ %}size(){% endihighlight %} returns an unsigned type. And you'll encounter these unsigneds are all over the standard library and even in the language itself (eg {% ihighlight c++ %}alignof(){% endihighlight %}, {% ihighlight c++ %}sizeof(){% endihighlight %} and {% ihighlight c++ %}sizeof...(){% endihighlight %}).

You may have your own opinions about whether you like that fact. But it is a fact and it's a fact I can't see changing any time soon.

So if you want to purge the {% ihighlight c++ %}unsigned{% endihighlight %} keyword from your code, be my guest. But let's acknowledge that that won't actually solve the problem, it'll only sweep some manifestations of it under the carpet.

And I'll soon argue that unsigned types offer real benefits that this approach loses.

So this approach is throwing out the baby and not even getting rid of all the bathwater.


The different response
====

Rather than avoiding unsigneds in the hope of reducing the chance of accidentally making the sort of slip above, let's just tell the compiler to forbid that slip. Professional Perl developers [expect](https://perlmaven.com/always-use-strict-and-use-warnings) each other to always `use strict` and `use warnings`. I propose that professional C++ developers should already expect each other to `-Werror` (treat warnings as errors) and should now also expect `-Wsign-compare -Wsign-conversion` too.

With these flags, GCC and Clang reject all the above examples with sensible error messages. Job done.

Even if you decide that, like Jon, you're going to avoid unsigneds, it still makes sense to let these checks protect you wherever your code interacts with other unsigneds. If you're completely sure your code never will, then turning on the checks will cost you nothing.

Of course, turning on compiler checks often incurs time costs for fixing the code it rejects. (And FWIW, I would like to see the compilers offer an option that warns about implicit signed -> unsigned conversions but not about unsigned -> signed). But I think the signal to noise ratio is easily enough to justify the checks for anything more serious than knocking up a fun project on the weekend.

For {% ihighlight c++ %}( -1 < unsigned_one ){% endihighlight %} :

|                     | GCC 6.2  | Clang 3.8 | ICC17 (on [godbolt](https://godbolt.org/)) |
|:------------------- |:--------:|:---------:|:------------------:|
| `-Wsign-compare`    | &#10004; | &#10004;  |                    |
| `-Wsign-conversion` | &#10004; |           | &#10004;           |
| `-Wall`             | &#10004; |           |                    |
| `-Wconversion`      |          |           | &#10004;           |

And for {% ihighlight c++ %}f( -1 );{% endihighlight %} :

|                     | GCC 6.2  | Clang 3.8 | ICC17 (on [godbolt](https://godbolt.org/)) |
|:------------------- |:--------:|:---------:|:------------------:|
| `-Wsign-compare`    |          |           |                    |
| `-Wsign-conversion` | &#10004; | &#10004;  | &#10004;           |
| `-Wall`             |          |           |                    |
| `-Wconversion`      |          | &#10004;  | &#10004;           |

Note that `-Wcast-qual` and `-pedantic` are no use for any of this.

Now that I've argued compiler checks are a better solution to the original problem than abandoning unsigneds, I'd like to make a further separate argument in defence of unsigneds.

Jon identified one central problem with them and that's been addressed. And that changes the picture a lot. For example, Jon uses this example of a function signature in his talk:


IT'S NOW GOING TO MAKE YOUR LIFE MUCH EASIER TO USE THE NATURAL TYPES.


And unsigned types still have three really big advantages in their favour:
1. they can't represent negative numbers;
1. compilers know that and
1. so do C++ programmers.

It's really helpful to use unsigneds to tell the compiler when you know that variables should always be positive because then it can reliably detect any violations for you at compile-time. This automatically protects you from lots of unambiguous bugs (like mistakenly passing -1 to a function taking an {% ihighlight c++ %}unsigned int{% endihighlight %}). It can also highlight points in your code where your unsigned variables interact with signed data, points which often merit you thinking about often require you think about that inter and it's a good idea to think about that and decide how your code should handle it.

If you instead put those always-positive variables in signed types, you're hiding that information from the compiler so now you're responsible for writing any checks and they'll all be postponed until runtime. For example, a runtime check for every function argument that takes an always-positive number as an int that the input really is positive.


{% highlight cpp %}

int foo::recalc_num_bars(const int &num_bars) {
  if ( num_bars < 0 ) {
    do_something_about_it();
  }

  /* ... */

  if ( return_value < 0 ) {
    do_something_about_that();
  }
  return return_value;
}

{% endhighlight %}

It's also really helpful to use unsigneds to tell readers (and clients) of your code when you know variables should always be positive because it's an incredibly concise expression that comes with a compiler-backed guarantee.

Of course, if you put your always-positive variables in signed types, then you can always write comments to document that for every such variable. <br>
Or you might ofen forget. <br>
Or the reader might not be able to find your comments. <br>
Or you might let them get out of sync.
Or, even if not, the reader might wonder whether they are out of sync. <br>
Or the reader might wonder if any of the other ints are just undocumented always-positive ints. <br>
Or the reader might get needlessly confused by having to keep switching between different variables' comments. <br>
Or the reader might wonder if you're possibly using negatives values as undocumented special values. <br>
Or the reader might wonder whether you've remembered to check the value really is positive, and then re-checked later on, and then rechecked later on again, etc. <br>
Or the reader might wonder whether your function might accidentally return a negative and what they should do about that. <br>
Or they might wonder about happens if a negative value does get into your code.

And all of these issues can seem perfectly obvious when you're writing the code but then can make the code so much less clear for future readers.



So let's look at some objections:


Objection 1. Performance
========================

It's possible that sticking to ints makes code run a bit faster, but I'm guessing that's only a substantial effect in a tiny, tiny proportion of code that's hyper performance critical.


Objection 2. Typing
===================

{% ihighlight c++ %}unsigned int{% endihighlight %} does require more key presses than `int`. So create a type alias. I use `uint`:

{% highlight cpp %}

using uint = unsigned int;

{% endhighlight %}

If that's too long, you can use unt.

Actually, I mostly use {% ihighlight c++ %}size_t{% endihighlight %} because then I don't get narrowing conversions from standard library return types so I can use compiler warnings for narrowing conversions without too much noise.

{% highlight cpp %}

constexpr std::size_t operator "" _z (unsigned long long n
                                      ) {
	return static_cast<size_t>( n );
}
constexpr auto x = 5_z;

{% endhighlight %}

Objection 3. Difficulty
=======================

Whenever you choose to use an always-positive invariant to a variable, you do need to take a bit more care to ensure it doesn't go below 0, because that violates your invariant and is an error. That's true whether you implement that variable in a signed or an unsigned but it is true that an unsigned type is more likely to expose the error in a bug.

These problems are typically going to come when you're decrementing or subtracting an unsigned so let's look at those.

Decrementing an unsigned past 0 almost always comes when you're reverse looping down to zero. This upwards for loop is fine:

{% highlight cpp %}

for (size_t x = 0_z; x < 10_z; ++x) /* ... */

{% endhighlight %}

...but this downwards for loop is an infinite loop:

{% highlight cpp %}
for (size_t x = 9_z; x >= 0_z; --x) /* ... */
{% endhighlight %}

because after 0, the x will be decremented past zero, which will make it a huge positive number, which will then pass the test and keep the loop going.

Fortunately, I don't generally have this problem because I code these sorts of loops using a range-for over a suitable range from one of the range libraries. For example, using range-v3 or Boost Range, the upwards loop becomes:

{% highlight cpp %}
for (const size_t &x : iota( 0_z, 10_z ) ) /* ... */

for (const size_t &x : irange( 0_z, 10_z ) ) /* ... */
{% endhighlight %}

...which I prefer. I find it more readable and expressive. I find it easier to avoid mistakes. I like getting a const loop variable which can flag any mistaken attempts to alter it. I've also found it to be comparably fast in simple benchmarks I've run.

{% highlight cpp %}
for (const size_t &x : iota( 0_z, 10_z ) | reverse ) /* ... */

for (const size_t &x : irange( 0_z, 10_z ) | reversed ) /* ... */
{% endhighlight %}

That expresses what I want cleanly and doesn't try to make the unsigned negative.

Subtraction is more tricky. Have a look at this example

{% highlight cpp %}

unsigned int a = 4;
unsigned int b = 7;
( a - b );

{% endhighlight %}

Since the subtraction in the bracketed expression is being done with {% ihighlight c++ %}unsigned int{% endihighlight %}s, the negative answer of -3 comes out as a huge positive integer.

Again, it's quite likely that if you were thinking of these variables as always-positive values, this is still an error in your code if you use signed types. And you only have to use the result in a signed context to get a compiler error to highlight the problem.

And of course, as discussed above, avoiding unsigneds in your code doesn't actually eradicate them from your code. For example:

{% highlight cpp %}
vector<int> a = { 1, 2, 3, 4 };
vector<int> b = { 1, 2, 3, 4, 5, 6, 7 };
( a.size() - b.size() );

{% endhighlight %}

...doesn't explicitly instantiate any unsigned types but suffers the same problem. So *all* C++ programmers need to learn to be alert to this danger.

These issues aren't quite

Still, I think this is the biggest reason to avoid unsigneds. So, as far as I can see, the situation is that unsigneds are really valuable because they're enormously helpful to both compilers and humans looking at our code, allowing both to do their job better. We've been able to fix the worst problems, which Jon highlighted. The only remaining substantial problem is the danger of running past zero when substracting or decrementing. Again, dropping unsigneds feels like the wrong solution because it's throwing out all the lovely benefits of unsigneds with the subtracting/decrementing bathwater. Instead, wouldn't it be better 

Still I would like to see a better strategy for this problem. My thought is:
 * ask the compilers to add a warning for subtraction of unsigneds with a message that suggests alternatives
 * encourage widespread use of that warning, along with -Werror (which wouldn't cost unsigned-rejecters anything)
 * standardise small helper functions that do the things you're likely to want to do in this situation, say:
   * std::abs_diff   ( 7u, 4u ) ==  3u
   * std::abs_diff   ( 4u, 7u ) ==  3u
   * std::signed_diff( 7u, 4u ) ==  3
   * std::signed_diff( 4u, 7u ) == -3
   * std::dim        ( 7u, 4u ) ==  3u
   * std::dim        ( 4u, 7u ) ==  0u
   * std::subtract   ( 7u, 4u ) ==  3u
   * std::subtract   ( 4u, 7u ) ==  4294967293u






So let me summarise:

I agree with Jon that the signed -> unsigned implicit conversions are nasty

I disagree that avoiding unsigneds is an adequate guideline because most of us do still encounter them

Rather, I think that regardless of feelings about unsigned, we should tell our compilers to reject the dangerous implicit conversions

That done, I think unsigneds then offer real benefits in how much they tell the compiler and readers of our code

My one big hesitation is the ( 1u - 2u ) == 4294967295u issue. Again, that danger exists even for coders that avoid unsigned types.

Suggestion:
I like the idea: compilers offering a warning about unsigned subtraction and 



So what do you think? Please discuss and let me know.







/






Advanced:
-isystem


// g++                                              -std=c++11                eg.cpp -o eg.gcc_naive_bin   && ./eg.gcc_naive_bin
// clang++                                          -std=c++11 -stdlib=libc++ eg.cpp -o eg.clang_naive_bin && ./eg.clang_naive_bin
// g++     -Werror -Wsign-compare                   -std=c++11                eg.cpp -o eg.gcc_bin         && ./eg.gcc_bin
// clang++ -Werror -Wsign-compare                   -std=c++11 -stdlib=libc++ eg.cpp -o eg.clang_bin       && ./eg.clang_bin
// g++     -Werror -Wsign-compare -Wsign-conversion -std=c++11                eg.cpp -o eg.gcc_bin         && ./eg.gcc_bin
// clang++ -Werror -Wsign-compare -Wsign-conversion -std=c++11 -stdlib=libc++ eg.cpp -o eg.clang_bin       && ./eg.clang_bin
// g++     -Werror -Wsign-compare -Wconversion      -std=c++11                eg.cpp -o eg.gcc_bin         && ./eg.gcc_bin
// clang++ -Werror -Wsign-compare -Wconversion      -std=c++11 -stdlib=libc++ eg.cpp -o eg.clang_bin       && ./eg.clang_bin

// -Wcast-qual
// -Wextra



	FWIW, I'd like the compilers to offer a setting that rejects implicit signed -> unsigned conversions (32-bit eg: -1 -> 4,294,967,295) but permits implicit unsigned to signed (32-bit eg: 4,294,967,295 -> -1) because latter case is only a problem when you're using the upper half of the unsigned type's range, in which case you should probably be using a bigger type anyway (as Jon says in his talk).



 (Jon Kalb Bjarne Stroustrup)

thanks to Jon, James, Jason(?)
