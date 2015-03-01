POSIX specifies (see [`regex(7)`](http://linux.die.net/man/7/regex), or
`re_format(7)` on BSD) that ERE chooses the leftmost longest match:

> In the event that an RE could match more than one substring of a given string,
> the RE matches the one starting earliest in the string. If the RE could match
> more than one substring starting at that point, it matches the longest.

"Earliest" (i.e. "leftmost") is consistent with PCRE, but "longest" is not. To
illustrate the difference, consider the pattern `/a?(ab)?/` and the string
`"ab"`. Note that the possible matches are `"a"` and `"ab"`.

PCRE matches `"a"`:  `/a?/` matches `"a"`, and then `/(ab)?/` matches the empty
string. `grep -P`, `perl`, and `python` all exhibit this behavior:

{% highlight bash %}
$ echo ab | grep -oP 'a?(ab)?'
a
$ echo ab | perl -pe 's/a?(ab)?/c/'
cb
$ python -c 'import re; print re.search(r"a?(ab)?", "ab").group()'
a
{% endhighlight %}

ERE, on the other hand, matches `"ab"`, the longest of the possible matches.
`grep -E`, `sed -r`, and `awk` serve as examples of this:

{% highlight bash %}
$ echo ab | grep -oE 'a?(ab)?'
ab
$ echo ab | sed -r 's/a?(ab)?/c/'
c
$ awk 'BEGIN { s = "ab"; sub(/a?(ab)?/, "c", s); print s }'
c
{% endhighlight %}

Interestingly, `vim` matches `"a"` (using the pattern `/\va?(ab)?/`).
