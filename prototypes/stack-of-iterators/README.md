Stack of iterators
------------------

This is a toy example of a `scratch::text` type providing a whole stack of ranges,
none of which is privileged; that is, `text` itself is *not* a range, but it can be
converted fluidly into many different kinds of ranges.

* `mytext.code_units()` is a range of UTF-8 code-units, i.e., `char`.
* `mytext.code_points()` is a range of Unicode codepoints, i.e., `char32_t`.
* `mytext.graphemes()` is a range of Unicode EGCs, represented as the C++ class type `grapheme`.

Each `grapheme` object likewise provides `g.code_units()` and `g.code_points()`.

The `begin()` and `end()` of each range object return `code_unit_iterator`,
`code_point_iterator`, or `grapheme_iterator` respectively. Each of these iterators
is explicitly convertible (via `static_cast`) to any "lower-level" iterator. For example,

    grapheme_iterator itg = mytext.graphemes();
    code_point_iterator itp = static_cast<code_point_iterator>(itg);

This is statically type-safe because any (valid) `grapheme_iterator` is guaranteed to be
pointing at a grapheme boundary, which is also a codepoint boundary and a code-unit boundary.
This does not imply that `grapheme_iterator` must be implemented in terms of `codepoint_iterator`;
in fact it is not.


Out of scope
------------

This toy example does *not* implement non-Unicode encodings (except via an `#ifdef` to use ASCII
instead of UTF-8).

This toy example does *not* implement customizable error-checking policies; but it does do its
error-checking via a set of hooks that I think are a plausible orthogonal set of functionalities.
One could imagine passing a template parameter `ErrorPolicy` that provided these hooks.

I would love to implement the real Unicode EGC segmentation algorithm, but ye gods it looks complicated.
The current algorithm is just "glom together anything from one random page of combining diacritics
as a proof of API concept and ignore correctness entirely."

It is suggested that `word_iterator` and `linebreak_iterator` (and `sentence_iterator`?) should also
be part of the stack.

It is suggested that at least `word_iterator` needs to be bidirectional in order to satisfy the
use-case "I got a click on this character; highlight the word that contains it." Quite possibly
*all* of these iterators will need to be bidirectional in real life.
