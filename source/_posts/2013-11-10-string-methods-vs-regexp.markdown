---
layout: post
title: ""String methods vs Regexp""
date: 2013-11-10 17:48
comments: false
categories: 
- RegExp
---
There is a [notorious quip](http://regex.info/blog/2006-09-15/247)
about regular expressions by Jaime Zawinski:
> Some people, when confronted with a problem, think, "I know,
> I'll use regular expressions." Now they have two problems.

Modern-day developers have a tendency to reach for regular
expressions when tasked with processing text.

## Problem: removing white space
A classic example may be the removal of white space at the
beginning and the end of a string.

We can strip white spaces in two steps:

```ruby Naïve approach
s = '   aaa  a   '
s.gsub! /^\s*/, ''
s.gsub! /\s*$/, ''
```

This works.

Having learned lazy captures, you may try doing it in a single
swoop:

```ruby Single swoop 1
s = '   aaa  a   '
s.gsub! /^\s*(.*?)\s*$/, '\1'
```

This is not quite optimal.
The new tool in our arsenal, the lazy qualifier `*?` causes
the finite state machine to back track quite a bit if
we _do_ have trailing white spaces.

To address this problem, we can explicitly look for a non-space
character in the right place:

```ruby Single swoop 2
s = '   aaa  a   '
s.gsub! /^\s*((?:.*\S)?)\s*$/, '\1'
```

(Note that we need to wrap `.*\S` in a group and make it optional
so that we can deal with space-only strings.)

## Benchmark
Having found alternatives, we will have to benchmark:

```ruby
require 'benchmark'

n = 5_000

str = '   abc d a dfadg   '

Benchmark.bmbm do |x|
  x.report('simple') { n.times {
    s = str.dup; s.gsub! /^\s*/,''; s.gsub! /\s*$/,''
  } }
  x.report('single1') { n.times {
    s = str.dup; s.gsub!(/^\s*(.*?)\s*$/, '\1')
  } }
  x.report('single2') { n.times {
    s = str.dup; s.gsub!(/^\s*((?:.*\S)?)\s*$/, '\1')
  } }
end
```

Here are results (with 2.0.0-p247):
```text
Rehearsal -------------------------------------------
simple    0.030000   0.000000   0.030000 (  0.031588)
single1   0.030000   0.000000   0.030000 (  0.030326)
single2   0.020000   0.010000   0.030000 (  0.023397)
---------------------------------- total: 0.090000sec

              user     system      total        real
simple    0.030000   0.000000   0.030000 (  0.032306)
single1   0.020000   0.000000   0.020000 (  0.025628)
single2   0.020000   0.000000   0.020000 (  0.018592)
```

At this point, you might be satisfied and go with the last approach.

And you would be wrong.

## There is a better way
Because String has [#strip](http://www.ruby-doc.org/core-1.9.3/String.html#method-i-strip)
method that does this, we do not have to mess with
`Regexp` at all.

```text
Rehearsal -------------------------------------------
simple    0.030000   0.000000   0.030000 (  0.030317)
single1   0.020000   0.000000   0.020000 (  0.024662)
single2   0.020000   0.000000   0.020000 (  0.019647)
strip     0.000000   0.000000   0.000000 (  0.002659)
---------------------------------- total: 0.070000sec

              user     system      total        real
simple    0.020000   0.000000   0.020000 (  0.027069)
single1   0.020000   0.000000   0.020000 (  0.023070)
single2   0.020000   0.000000   0.020000 (  0.018130)
strip     0.010000   0.000000   0.010000 (  0.002551)
```

The same principle applies, to a lesser degree,
with testing the initial character with
`/^[aeiou]/`, the last character with `/[aeiou]$/`, and
simply searching for occurrence of a character sequence with
`/some string/`.

You are better off with
[`#start_with?("aeiou")`](http://www.ruby-doc.org/core-1.9.3/String.html#method-i-start_with-3F),
[`#end_with?("aeiou")`](http://www.ruby-doc.org/core-1.9.3/String.html#method-i-end_with-3F),
and
[`#include?("some string")`](http://www.ruby-doc.org/core-1.9.3/String.html#method-i-include-3F).

In all these cases, performance gains may vary from meaningful to
marginal, but the readability improvement is undeniable.

