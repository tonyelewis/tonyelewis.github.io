---
layout: post
title:  "clone_ptr"
date:   2016-11-14 17:32:44 +0000
categories: jekyll update
---

 * [link](http://stackoverflow.com/questions/4069865/clone-ptr-implementation-error)
 * [link](http://www.codeproject.com/Articles/11399/Clone-Smart-Pointer-clone-ptr)
 * [link](https://www.codeproject.com/script/Membership/LogOn.aspx?rp=http%3a%2f%2fwww.codeproject.com%2fArticles%2f11399%2fClone-Smart-Pointer-clone-ptr&download=true)
 * [link](http://codereview.stackexchange.com/questions/92593/a-clone-ptrt-that-does-not-require-t-to-have-a-clone-method)
 * [link](http://www.boost.org/doc/libs/1_48_0/doc/html/move/implementing_movable_classes.html)
 * [link](http://www.codeguru.com/cpp/cpp/algorithms/general/article.php/c10407/Clone-Smart-Pointer-cloneptr.htm)

I want to argue for better C++ support for a clean, safe and easy way to get normal copy-semantics for polymorphic base classes (probably requiring some dynamic copy-construction ("clone()") boilerplate code for the polymorphic classes). In particular, I suggest a std::clone_ptr<T>, which is like std::unique_ptr<T> except that it requires T be cloneable and exploits that to provide normal copy semantics.

My Questions
============

I feel like this would C++ much friendlier for implementing nice ABC-based designs. Am I missing something? Is this a stupid idea? Or do you think it might be worth publicising in the community?

The Problem
===========

To motivate... your new class uses one of several policies so you build a nice ABC-based hierarchy to implement the policies decision and then you add a std::unique_ptr to the ABC as a data member in your class (ie the strategy design pattern) :

{% highlight cpp %}
class abc_policy                            { /* ... */ };
class concrete_policy_a : public abc_policy { /* ... */ };
class concrete_policy_b : public abc_policy { /* ... */ };

class my_class {
private:
  std::unique_ptr<abc_policy> the_policy;

  /* ... */
};
{% endhighlight %}

...and now you find your compiler is stopping you doing reasonable things because you've broken copy-semantics for my_class (and other classes that contain it). AFAIA, this leaves you with few good options:

 1. accept that you'll have to find some workaround for each error the compiler throws at you for trying to copy my_class
 1. decide that C++ makes this sort of abstracting/separating design too painful and squash all the code back into my_class
 1. add dynamic copy-construction (`clone()`) boilerplate code for the abc_policy hierarchy and then write your own special methods for my_class to use it

Your day goes badly quickly, though you feel like you've only followed simple, standard OO practice.

Steps to a Solution
===================

In my code, I've:
 1. standardised my clone boilerplate
 1. created a cloneable concept and
 1. (most importantly) created a clone_ptr<T> that's just like std::unique_ptr<T> except that it requires T be cloneable and exploits that to provide normal copy semantics

The Three Smart Pointers

What should a smart pointer do when you write code that tries to copy (not move) the smart pointer

 * `std::unique_ptr<T>` - prevent it at compile-time
 * `std::shared_ptr<T>` - create another reference-counted pointer to the same object
 * `std::clone_ptr<T>` - copy the object using the 

 to

 Though you're willing to add clone() boilerplate code to your policy classes, you still find that
 (a) you're on your own there with no support from the language/library to achieve  and (AFAIA) having to do horrible workarounds, possibly mixing memory management code in with my_class logic.


------

&nbsp;

&nbsp;

&nbsp;

Now you've broken copy-semantics for my_class (and for other class that contains it). I suspect this issue comes up quite a lot and is one of the common reasons why .

&nbsp;

I feel that C++ has a big hole around supporting dynamic copy-construction.

&nbsp;

Let's say you try a good C++ design 

&nbsp;

You've learnt about OO, design patterns and C++ so you 

&nbsp;

unique_ptr<T>

&nbsp;

shared_ptr<T>

&nbsp;

clone_ptr<T>

&nbsp;

I'll ensure my classes have necessary boiler plate to do dynamic copy-construction

&nbsp;

&nbsp;

&nbsp;

<!-- You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/ -->
