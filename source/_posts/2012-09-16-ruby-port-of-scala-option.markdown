---
layout: post
title: Ruby port of Scala's Option monad
date: 2012-09-16 10:39
comments: false
categories: ruby scala gem
---

<p>
I have been doing some work in Scala recently and have started to
find the <a href="http://www.scala-lang.org/api/current/scala/Option.html" target="_blank">Option</a> monad (<a href="http://www.haskell.org/ghc/docs/latest/html/libraries/base/Data-Maybe.html" target="_blank">Maybe</a>
in Haskell) increasingly useful in day-to-day work.
A few weeks ago I thought I might try my hand at porting Option to Ruby for my own personal use
but found I had something that might benefit others; so I give you <a href="https://github.com/rares/option" target="_blank">option</a>.
</p>

<p>
For the un-indoctrinated, Option is a container type that allows one to deal with a boxed value in
certain terms (i.e., does the value exist or is it empty?).
It is an extremely flexible way to take input and manipulate it to produce a value.
For example, for any type of value with common interface, I can easily chain together calls
that check for presence of the value, transform it and then default the value if no value is present.
</p>

{% codeblock lang:rb %}
require "option"

Option(User.find_by_email(params[:email])).
  filter { |u| u.check_password(params[:password]) }.
  inside { |u| u.record_login! }.
  get_or_else { User.anonymous }
{% endcodeblock %}
<p>
The above will:
<ol>
  <li>
  Wrap the results of the email lookup in 'Option(user)'. It will be either a 'Some(user)' if a User is found or 'None' if nil is returned.
  </li>
  <li>
  <strong>filter:</strong> Use the result of the previous call to either check the password is correct or just return the 'None'.
  If the password check is true, then the Option returns 'Some(user)' again. If not, it returns 'None'.
  </li>
  <li>
    <strong>inside:</strong> if the password check was correct, then record that the user logged in
    successfully using the 'Some(user)'. 'inside' then just returns the reference to the 'Some(user)'.
  </li>
  <li><strong>get_or_else:</strong> if any of the calls in the chain returned 'None' then when we
  reach the get_or_else call, the Option invokes the behavior in the block, in which we define as returning an anonymous User. If the Option
  is holding a 'Some(user)' we get the un-boxed 'user'.
  </li>
</ol>
These calls (as well as a handful of others) can be used interchangibly to implement
any sequence of events but in a safer and less error prone way than manually checking for nil values.
repeatedly.
</p>

<p>
As mentioned before, the code is <a href="http://www.github.com/rares/option" target="_blank">here</a> and patches/suggestions/feedback are wecome!
</p>
