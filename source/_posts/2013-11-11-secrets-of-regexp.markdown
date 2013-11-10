Earlier this year, I gave a talk with this title at
[Christchurch Ruby](http://christchurch.ruby.org.nz/).

Later, I had an opportunity to refine that talk.
This post walks you through the ideas presented.

## Once upon a time…
Every developer nowadays should know what "regular expressions"
(henceforth regex) are.
But not a lot of them know where regexes came from.

In order to explain the genesis of regex, we need to step back and
review formal language theory.

## Just enough formal language theory
In a branch of mathematical logic called formal language theory, there
is no semantics.
Just forms.

A languages _L_ consists of words, which consists of letters in an alphabet _Σ_.
Devoid of semantics, a language can be finite (in terms of the number of words
belonging to it), but finite languages are not very interesting.

Then, one can consider describing a language by succinctly providing a
test to determine whether or not a string of letters in _Σ_ is a valid
word in _L_.
This is basically what an 'expression' is.

While constructing an expression, you can denote closure under
concatenation; a string of letters is a word in _L_ if that string is
'repeated' at least once.
This 

{% codeblock %}
foo!
{% endcodeblock %}
