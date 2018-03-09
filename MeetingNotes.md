# Meeting Notes

The next meeting is scheduled for Wednesday, March 28th, from 2:30-4:00pm EST.

- [March 7th, 2018](#march-7th-2018)
- [February 21st, 2018](#february-21st-2018)
- [January 31st, 2018](#january-31st-2018)
- [January 10th, 2018](#january-10th-2018)
- [December 13th, 2017](#december-13th-2017)
- [November 30th, 2017](#november-30th-2017)
- [October 25th, 2017](#october-25th-2017)
- [October 11th, 2017](#october-11th-2017)
- [September 27th, 2017](#september-27th-2017)
- [September 13th, 2017](#september-13th-2017)
- [August 30th, 2017](#august-30th-2017)
- [August 16th, 2017](#august-16th-2017)

# March 7th, 2018

## Draft agenda:
- Discussion of JeanHeyd's and Martinho's prior Unicode related efforts.
  JeanHeyd had previously experimented with basic_text and basic_text_view
  classes and Martinho is the author of Ogonek (https://github.com/libogonek/ogonek).

## Meeting summary:
- Attendees:
  - Tom Honermann
  - Peter Bindels
  - JeanHyde Meneide
  - Corentin Jabot
  - Mark Zeren
  - R. Martinho Fernandez
- First up was a round of introductions in honor of our new attendees.
- JeanHyde presented his prior work on `text_view` and `text` types.
  - [Slides](presentations/a%20rudimentary%20unicode%20abstraction%20-%20ThePhD%20-%202018.03.07.pptx)
  - Tom loved that JeanHyde's presentation could almost be used as-is for
    his own `text_view` implementation!
  - Notable similarities to Tom's `text_view`:
    - Naming.
    - Both are based on range concepts.
    - Both use low level encode/decode interfaces wrapped by iterator classes.
    - JeanHyde's `text` closely matches what Tom has planned to provide
      for approximately forever, but still doesn't.
    - Default encodings are deduced based on code unit type (`char`,
      `char16_t`, etc...)
    - Error policies are used to allow different error handling.  By default,
      exceptions are thrown, but a policy is provided to substitute a
      replacement character.
    - We have both struggled with being unable to differentiate `"text"` and
      `u8"text"`.
  - Notable differences from Tom's `text_view`:
    - UTF-8 is assumed as the default encoding for `char` based strings; Tom's
      implementation assumes the implementation defined character encoding.
    - The active error policy can be overridden on a per-operation basis
      (for at least some operations).  This ability never occurred to Tom.
    - Relational operators are provided; Tom's implementation doesn't (yet?)
      provide them.
  - Design choices we discussed:
    - Implicit converting constructors for `text` that transcode from other
      encodings.
      - Potentially undesirable due to hidden performance costs.
      - Compatible encodings (e.g., ASCII as a subset of UTF-8) allowed
        an optimization to use memcpy rather than transcoding.
        - JeanHyde remarked that this had a tendency to produce broken UTF-8
          when invalid ASCII sequences were provided.
        - Tom noted that broken UTF-8 seemed reasonable in that case since
          a precondition was violated.
        - Mark noted that UB could be good for release builds, but for debug
          builds, validation could be desirable.
    - Normalization forms.  Choose one?  Provide as views?
    - Relational operators.  [Editor's note: I think JeanHyde mentioned that
      these operators implemented canonical equivalence, but I don't recall for
      sure.]
- Martinho discussed his work on Ogonek.
  - https://github.com/libogonek/ogonek
  - Like all efforts we've discussed so far, much fun was had experimenting
    with error handling approaches.
  - Layering of segmentation iterators over code point iterators over code unit
    iterators resulted in large iterators.  In some cases, a single iterator might
    store 16 iterators within it.
    - When discussing this, Tom mentioned our discussions of layering in Albuquerque
      and the conclusion we reached that higher level iterators should store code
      unit iterators and, if intermediate representations were needed, views of the
      appropriate abstraction could be created on demand.
  - Support for normalization and segmentation (grapheme/word iterators) is provided.
  - Grapheme cluster segmentation supports locale tailoring.
  - Implementation of segmentation uses an elegant approach of operator overloading
    to enable specifying the segmentation rules almost verbatim from the Unicode
    standard; this helped to avoid implementation errors.
  - Flags are used instead of enumerations for break conditions; this enables
    tracking multiple break states concurrently.
  - The intent to optimize dispatch for, e.g., transcoding operations, has been
    present from the beginning (as has been the case for all of our collective
    efforts) but (like all of our collective efforts so far) has not yet been
    implemented.
- Further dicussion followed Martinho's presentation:
  - Dutch "ij" is a digraph consisting of two code points that form a single
    grapheme cluster.  When capitalizing, both letters are capitalized to "IJ".
  - If you don't normalize, comparison operators are bad!
    - JeanHyde's presentation made this point as well:
      "Ў" (u+040E) != "y˘" (u+0079 u+02D8)
  - Trivia: Vietnamese keyboards default to producing scan codes matching Unicode
    NFD; most keywords produce NFC.
  
## Assignments:
- Tom: Get the use cases doc sufficiently up to date to solicit help.


# February 21st, 2018

## Draft agenda:
- Topics for the Jacksonville evening session.  Some topic ideas:
  - "Common" text operations.
  - Text is immutable (get a rope)
  - Normalization as an implementation detail:
    - When a particular normalization form is needed, use a view.
    - If a contiguous code unit sequence is needed, copy from a view.
  - Backward compatibility for char8_t.
  - Transcoding at the borders:
    - Command line issues.
    - Environment variable issues.

## Meeting summary:
- Attendees:
  - Tom Honermann
  - Zach Laine
  - Mark Zeren
  - Peter Bindels
- Mark raised concerns about our use of Slack and the fact that our discussions
  there keep disappearing due to the 10000 message limit currently imposed on the
  Cpplang workspace.  This results in continued rehashing of prior discussions as
  new people join our channel.
  - It was suggested that we could setup a separate workspace just for our group;
    this would at least give us our own 10000 message limit.
  - A suggestion was made to use Google Groups or similar.  Tom noted that the
    benefits of immediate discussion that takes place in a chat facility are
    difficult to reproduce in an email list.  But there was general agreement that
    we have no reason not to setup a Google Group.  Tom agreed to do so.  The new
    group is available at https://groups.google.com/forum/#!forum/std-text-wg.
- Next up was discussion of an agenda for our planned evening session in Jacksonville.
  We started off with the items from the draft agenda.
  - "Common" text operations:
    - Tom noted that this overlaps with our previous intent to draft a use cases
      document (the one that Tom has completely failed to follow up on but still
      intends to do).  The idea would be to help identify gaps and biases in our
      own experience.  The discussion might help to address the code points vs
      grapheme concerns and help us answer whether 1) there should be a default
      mode of operation and, 2) if so, should it be code point or grapheme based.
    - Zach observed that the Unicode standard already documents, at least at a low
      level, all of the operations users might want to do.  For example, collation,
      case mapping, etc...  As long as the provided interfaces expose what Unicode
      documents, then additional abstractions can be built on top as desired.
    - We then segued into a discussion of locales; should interfaces eschew, permit
      or require locale selection via explicit parameters?
      - Peter observed that having to forward locale options around throughout an
        entire application is undesirable and problematic.
      - We discussed a few alternatives such as global or thread local locale state
        but no conclusions were reached.
      - Zach noted that collation tailoring effectively prohibits a static set of
        locales; e.g., potential locales can't be statically enumerated.
      - We briefly discussed locales as part of the type system, but concluded that
        locales are fundamentally run-time bound.
  - Normalization as an implementation detail:
    - Tom repeated a perspective that, when a particular normalization form is
      needed, use a view adapter and, if a contiguous code unit sequence is needed,
      copy from a view.
  - Text immutability:
    - Tom asked about use cases for mutating text in place as opposed to using a
      rope or other segmented data structure to overlay edits.
    - Peter observed that ropes are great for such purposes when dealing with large
      text, but that in-place mutation works well for smallish strings.
    - Tom noted that a benefit of edits-via-rope is that iterators into the underlying
      text are not invalidated.
    - It was acknowledged that the ability to append is common and necessary.  There
      is a case to be made for string builders.
    - Mark described a desire for a non-allocating rope.  The use case is to
      conceptually concatenate some number of strings without necessarily allocating
      storage for the combined result.  Peter noted that his rope implementation
      supports this:
      - https://github.com/dascandy/s2/blob/master/include/s2/detail/rope.h#L26
  - Transcoding at the borders:
    - We discussed the feasibility of mandating a specific internal encoding and
      requiring transcoding on I/O.
    - Tom expressed an intent to write a paper addressing command lines and environment
      variables.
  - Status updates:
    - Tom suggested that we could present status updates on our respective projects
      (on Zach's text, Tom's text_view, char8_t).
    - As part of discussing char8_t, we could specifically discuss backward compatibility.
- Discussion then moved on to current status updates:
  - From Zach:
    - The Unicode line break algorithm is really complicated.
    - Currently working on bidirectional algorithm support.
    - Next up is line breaking, case mapping, finishing up collation tailoring.
    - Zach described some interesting properties of Unicode collation; proper collation
      requires comparing full strings due to multiple weight levels.  Basically,
      a collation algorithm can not terminate early.
    - We discussed the idea of shareable test infrastructure for some of the work
      Zach has been doing.
  - From Tom:
    - The char8_t implementation is now complete:
      - https://github.com/tahonermann/gcc/tree/char8_t

## Assignments:
- Tom: Get the use cases doc sufficiently up to date to solicit help.
- Tom: Finalize evening session in Jacksonville.


# January 31st, 2018

## Draft agenda:
- Status updates.
- Goals for Jacksonville.
- Normalization as part of the type system?
- Grapheme clusters and value semantics?

## Meeting summary:
Editor's note: This meeting summary was written long after this meeting
occurred and I find my memory of the discussion more fuzzy than normal.
I'll endeavor to ensure meeting summaries are recorded more promptly in
the future.  Zach and Florin are encouraged to update this summary with
their recollections.

- Attendees:
  - Tom Honermann
  - Zach Laine
  - Florin Trofin
- Zach reported his continued progress on collation tailoring.
- Florin commented on use of ranges in text processing.  Tom and Zach
  were happy to see him reaching similar conclusions that we had each
  reached while working on our respective projects.  (Editor's note:
  I wish I could remember more of this discussion).
  - [Florin's notes] Florin suggested to use a stream-oriented approach
    wrt normalization: If we keep the internal encoding in FCC and the
    clients need a read-only view for the text but with a different
    normalization encoding (FCD for example) - by having this stream
    oriented interface we avoid memory copies (you can do the conversion
    on the fly, lazily). ICU doesn't do this (they use a pointer/size
    buffer-oriented approach) which is problematic, both in terms of
    interface (doesn't play nicely with containers) and also
    performance - eagerly applies operations on the whole buffer.
- The remainder of our time was predominantly spent discussing
  normalization.
  - As mentioned in earlier meetings, Zach is using the FCC normalization
    form in his design.
  - Zach noted that NFC has become a defacto preference and is recommended
    by the W3C (https://www.w3.org/International/questions/qa-html-css-normalization)
    - Editor's comment: I find it interesting that the reason given for
      preferring a particular normalization form is to prevent unintended
      mismatches of identifiers between HTML and CSS as a result of failure
      to normalize identifier comparisons.
  - We discussed the idea of an internal normalization form and the idea of
    normalizing at application borders (as is typical for transcoding).  This
    lead to discussion of normalization iterators and views.
  - We discussed the appeal of providing text in particular normalization forms
    via view adapters.  This has the advantage of allowing implementors freedom to
    use whatever normalization form (if any) they deem appropriate and avoids
    having to specify or prefer any particular form.  A disadvantage is that,
    when text is needed in a particular normalization form in contiguous memory,
    users would need to copy to contiguous memory (e.g., `std::string`, `std::vector`,
    etc...).
  - We discussed the normalization of compile-time string literals.  No
    normalization form is currently prescribed for Unicode string literals, so
    text in literals may be in any normalization form or none at all (or not be
    well-formed at all).  This implies that, for text related containers that
    enforce a particular normalization form, conversions will be required even
    from string literals.
  - Zach noted that text may be well-formed for both the FCC and NFC forms.

## Assignments:
- Tom: Get the use cases doc sufficiently up to date to solicit help.
- Tom: Find a sponsor for an evening session in Jacksonville.


# January 10th, 2018

## Draft agenda:
- Peter Bindels to present his work on std2 string and character encoding
  support. See https://github.com/dascandy/s2/tree/master/include/s2
- Catch up on work over the winter break.  In particular, it sounds like
  Zach has made some good progress on grapheme cluster and collation support!

## Meeting summary:
- Attendees:
  - Tom Honermann
  - Zach Laine
  - Peter Bindels
  - Mark Zeren
- Zach provided an update on his text library:
  - `text` now maintains its content in the FCC normalization form
    (see http://www.unicode.org/notes/tn5).  This is a kind of FCD 
    form; see TN5 previously, or
    http://www.unicode.org/L2/L2001/01371-FCD.htm
  - A new set of `is_normalized_nf{c,kc,d,kc}()` free functions allows
    querying the normalization state of a sequence of code points.
  - Tailored normalization is implemented for languages and locales.
  - Support for most of the Unicode algorithms is now present.
    - Except for optional compression.
- Peter provided an overview of his library and the thoughts that have
  been motivating it.
  - Peter's ideas and motivations felt very familiar to both Zach and
    Tom.
  - Peter's work currently allows implicit conversion between different
    encodings.  We discussed the potential performance pitfalls of such
    implicit conversions.
- Further discussion revisited prior one-encoding-to-rule-them-all
  conversations.
  - We remain divided on this, but all agreed that Zach's work and the
    feedback produced from Boost review will be valuable.
  - Peter noted that avoiding transcoding by using UTF-16 (on Windows)
    as the internal encoding can actually *decrease* performance due to
    increased pressure on L1/L2 cache and less ability to take advantage
    of short string optimizations.
- We discussed the impact of encoding on command line arguments and
  environment variables.
  - It was noted that command lines and environment variable values can't
    effectively have an associated encoding because they may contain file
    names and, at least on POSIX, file names do not have to adhere to any
    particular encoding.
  - It was noted that recent versions of Windows actually enforces
    well-formedness of UTF-16 file names.  Tom (at least) was under the
    impression that Windows did not enforce this and treated file names
    as sequences of 16-bit code units that were not required to be
    well-formed UTF-16.
  - Tom noted a discussion with Gaby that occurred on the c++std-ext
    reflector under the title "typedef name for P0781R0" that discussed
    handling of command line arguments and environment variable values.
    Gaby suggested modeling a new interface on the filesystem design.
    For example, an interface that provides `u8string()`, `u16string()`,
    `u32string()`, etc... member functions to request values in a
    particular encoding.  Otherwise, the data would be provided in an
    implementation defined encoding.
- We spent a little time discussing potential graphem cluster operations,
  but are still spinning our wheels a bit.  The question was raised
  regarding whether value semantics are desirable for grapheme clusters.
  It was noted that, at a minimum, there must be convenient interfaces
  for comparing graphme clusters to character and, possibly, string
  literals.
- We discussed the role Unicode normalization should play in encoded
  string/text types.  We've so far mostly assumed that normalization
  would not be baked into the type system.  However, it may be desirable
  to use the type system to enforce a (internal) normalization form.
  Taken to the extreme, we could make normalization indivisible from
  encoding form.  e.g., `utf-8-nfd`, `utf-8-nfc`, etc...
- We discussed goals for the next meeting in Jacksonville.  Zach suggested
  the idea of hosting another evening session.  Tom to arrange.
- Tom mentioned the libunistring C library as another potential source
  of interface inspiration with regard to useful string/grapheme
  operations.
  - https://www.gnu.org/software/libunistring/manual/libunistring.html

## Assignments:
- Everyone: Think about what operations are desired on grapheme clusters.
- Tom: Get the use cases doc sufficiently up to date to solicit help.
- Tom: Arrange for another evening session in Jacksonville.


# December 13th, 2017

## Draft agenda:
- Report feedback on P0169: regex with Unicode character types.
- Discuss char8_t and backward compatibility approaches.

## Meeting summary:
- Attendees:
  - Tom Honermann
  - Zach Laine
  - Mark Zeren
- We discussed Zach's progress adding support for grapheme clusters:
  - text, text_view, and rope now deal in grapheme clusters, new string, string_view, and
    unencoded_rope classes provide code unit based functionality.  Each of these new classes
    has an implicit UTF-8 association.
  - The grapheme cluster iterators are bidirectional; there had previously been concerns
    that bidirectional grapheme cluster iterators could not meet complexity requirements,
    but that turned out not to be the case, at least not if support is limited to
    "Stream-Safe Text Formats" (http://www.unicode.org/reports/tr15/#Stream_Safe_Text_Format).
  - The prior checks on mutation to ensure well formed code unit sequences have been removed
    as operating on grapheme cluster boundaries ensures mutation at acceptable points.
- We discussed potential operations on graphemes:
  - Comparison being code unit based (fast, handles equality but not equivalence).
  - Comparison handling normalization (slow, handles equivalence and equality).
  - Relational operators (correct collation depends on locale).
- We discussed P0169 - regex with Unicode character types
  - It appears that std::regex interfaces lack correct support for variable length encodings.
    For example, std::regex_traits::translate() and std::regex_traits::translate_nocase()
    are passed and return a single code unit; that isn't sufficient to identify a Unicode
    code point for UTF-16 (and UTF-8).
  - Mark suggested a viable option of only adding support for char32_t specializations and
    requiring transcoding from UTF-16.  This would work around the std::regex issues with
    variable length encoding.
  - Ultimately, we decided not to pursue this proposal at this time in favor of focusing on
    more foundational work.
  - Tom noted that we should follow up with any potential std::regex replacement proposals
    to ensure encoding is handled properly.  In particular, this might include:
    - https://github.com/hanickadot/compile-time-regular-expressions
- We discussed Unicode normalization and how it fits into our considerations.
  - Mark suggested the possibility of baking normalization into the type system (implicitly
    or explicitly) similarly to what we've been discussing for encodings.
  - Tom suggested the idea that an application could adopt a particular normalization form
    to go with its internal encoding.  The application would then normalize as needed at
    application boundaries (including string literals, command line arguments, environment
    variables/values, file I/O, networking, etc...)
- Tom briefly provided an update on char8_t related work:
  - Still working on completing library support for libstdc++.
  - Github's code search facilities enabled identifying projects to use in determining
    backward compatibility requirements.
- Tom informed the group that Peter Bindels will join our next meeting and will present
  what he has been working on (https://github.com/dascandy/s2).

## Assignments:
- Everyone: Think about what operations are desired on grapheme clusters.
- Tom: Get the use cases doc sufficiently up to date to solicit help.


# November 30th, 2017

## Draft agenda:
- Review Albuquerque.
- Look at P0169: regex with Unicode character types
  - Jeffrey asked if we could review this and present it in Jacksonville.
    Otherwise, Titus will probably reject it for lack of a champion.

## Meeting summary:
- Attendees:
  - Tom Honermann
  - Florin Trofin
  - Mark Zeren
  - Zach Laine
- The meeting started with recap of various discussions from the Albuquerque
  meeting.  In particular, Mark summarized the discussion of iterator/range
  layering and emphasized the desire to provide abstractions (such as extended
  grapheme clusters) that map directly so code unit ranges (as opposed to
  indirectly via code point ranges).
- Tom reported progress on the char8_t proposal:
  - New effort was inspired by perceived encouragement for char8_t at the
    Albuquerque Unicode evening session.
  - Support for the proposed core language changes has been implemented in a
    fork of gcc available in the 'char8_t' branch of https://github.com/tahonermann/gcc.
  - Support for the proposed library changes has begun for libstdc++.
  - We discussed backward compatibility features that may be necessary.  Tom
    favors breaking compatibility in the standard and providing migration
    support via compiler options that implement backward compatibility features
    such as implicit conversions from 'char8_t' pointers/references to 'char'
    and non-strict type aliasing.
  - Plan is to have the char8_t proposal ready for presentation to EWG and
    LEWG in Jacksonville with a complete implementation available in gcc and,
    hopefully, Clang.
- Tom briefly mentioned the possibility of renaming his 'text_view' to
  'code_point_view' and proposing that in Jacksonville, but then immediately
  questioned the point in doing so.  If the intent is to encourage users to work
  primarily at the extended grapheme level, then providing code point utilities
  may be counterproductive.
- Zach reported progress on his text library:
  - Support for multiple error handling strategies is now available.  Support
    for throwing exceptions or substituting replacement characters is builtin;
    other strategies can be implemented by specifying an error handler functor
    type for the converting iterators.
  - Zach is starting to look into support for grapheme clusters.
- We again discussed the merits, or lack there of, of allocators.
- Tom requested that everyone review P0169 (regex with Unicode character types)
  for the next meeting to collect everyone's thoughts and discuss whether any
  of us is interested and/or willing to present it in Jacksonville.

## Assignments:
- Everyone: Review P0169.
- Tom: Get the use cases doc sufficiently up to date to solicit help.
- Tom: Test (Tom's) text_view view/iterators over (Zach's) text/rope containers.


# October 25th, 2017

## Draft agenda:
- Last meeting before Albuquerque.
- Straw polls to gauge our collective perspectives on various topics.
- Goals and scheduling for the next meeting.

## Meeting summary:
- Cancelled due to competing obligations.


# October 11th, 2017

## Draft agenda:
- Last meeting before Albuquerque mailing deadline.
- Review and discuss use cases and example solutions.
- Goals and scheduling for the next meeting.

## Meeting summary:
- Attendees:
  - Tom Honermann
  - Beman Dawes
  - Florin Trofin
  - Mark Zeren
- The meeting started off with introductions for Florin, our new recruit.
- We discussed proposals scope.  Florin cautioned against large sweeping proposals and
  there was general agreement.  Tom stated that small proposals are good, but that we need
  to keep the big picture in mind.  Beman noted that small proposals that set the foundation
  for other proposals that build on them are desirable.  There was general agreement.
- Tom relayed some feedback provided by Sean Parent regarding the text_view proposal.
  - Sean is not a fan of the error policy approach and observed that error handling
    isn't always needed.  He advocated a separate interface for validation (with error
    handling), and for transcoding (without error handling; well-formed input is a
    pre-condition).  The advantage being to exploit UB for performance benefit.
  - Sean observed that a simple transcode function could be implemented on top of
    text_view:
    ```
    template <class InputEncoding, class OutputEncoding, class InputIterator, class OutputIterator>
    OutputIterator transcode(InputIterator f, InputIterator l, OutputIterator out);
    ```
    `std::copy()` suffices to do this today, but only between encodings that use
    the same character set since text_view does not yet have support for transcoding
    code points between character sets.
- Florin brought up Adobe's open source library hosted at:
    https://github.com/stlab/adobe_source_libraries
  and we discussed its `copy_utf()` function:
    https://github.com/stlab/adobe_source_libraries/blob/master/adobe/unicode.hpp#L499-L525
  Tom noted that `std::copy()` suffices for this with text_view for UTF encodings and,
  once character set transcoding is in place, for any encodings (though policies for how to
  transcode code points that map 1-0 and 1-N may be an issue deserving of a specific interface).
- Back to Sean's observations of when error checking is needed and when it is unwanted, we
  discussed whether it made sense to specify building blocks that could be used for various
  purposes (ala text_view and its error policies) or whether it made more sense to specify
  interfaces for particular scenarios (validation, transcoding, enumeration) that have implicit
  error policies and that can exploit narrow contracts.  We concluded that separate interfaces
  should be specified such that they *can* be implemented in terms of building blocks, but
  that are not required to be.  This would allow an implementation to, for example, make
  direct use of SIMD instructions during transcoding without having to worry about error
  checking.  Enumeration remains a case where different error policies may be desirable.
- We briefly discussed again code point iteration vs (extended) graphme cluster iteration.
  Tom is coming to the conclusion that text_view is misnamed; it should be code_point_view
  with text_view reserved for enumeration of (extended) grapheme clusters.

## Assignments:
- Tom: Get the use cases doc sufficiently up to date to solicit help.
- Tom: Test (Tom's) text_view view/iterators over (Zach's) text/rope containers.


# September 27th, 2017

## Draft agenda:
- Review and discuss use cases and example solutions.
- Goals and scheduling for the next meeting. 

## Meeting summary:
- Attendees:
  - Tom Honermann
  - Mark Zeren
- Tom made a little progress editing the UseCases.html document
  (http://htmlpreview.github.io/?https://github.com/tahonermann/std-text-wg/blob/master/UseCases.html)
  though the changes have not yet been pushed.
- We reviewed the in-progress changes; an added introduction and graph depicting abstractly the feature
  landscape for text/Unicode proposals.  These changes are intended to help guide thinking about how
  various proposals fit together and where they fit in to a componentized view of text processing.
- Mark noted that the appearance of "grapheme cluster sequence" in the componentized view suggests a
  Unicode centric view that may not be desirable.  Tom agreed and clarified that that wasn't the intent;
  but that some terminology is needed to bridge between legacy encodings and Unicode encodings.  Perhaps
  "grapheme sequence" or similar would be a better choice.
- Mark noted that the document makes scant references to locales.  Tom agreed that updates are needed to
  better describe how locales fit in.
- Mark noted that interoperability with C is not mentioned either.  Tom agreed this should be addressed
  as well.
- Mark also provided some thoughts that are useful to guide our designs; essentially a number of
  naturally occurring tensions that necessitate balancing trade offs:
  - The tension between keeping things simple while enabling the full range of Unicode and legacy
    encoding complexity.  This impacts techability and on-boarding.  Complexity arises in distinct
    ways.  For example, ICU is a large complicated library.  `std::string` is simple by comparison
    but using it is complicated due to its limited features: providing only a code unit view and no
    explicitly associated encoding.
  - The tension between desires for vocabulary types vs (standard library) building blocks.  Most
    users probably want simple vocabulary types while experts need building blocks.  This raises the
    question of whether some vocabulary types can, or should, be constructed from building blocks.
    A current example is Zach's `text` class; should it be a template parameterized on an encoding,
    potentially with an alias for a UTF-8 specialization or should it be a standalone class?
  - The tension between behaviors that are selected at run-time vs compile-time.  For example, locale
    settings determine the encoding used for narrow/wide strings at run-time, but the encoding of
    Unicode strings is known at compile-time.  Another example appears in the functions exposed by
    the ctype header; character properties may be locale dependent in general, but not for Unicode.
- Tom posed the question of what should the semantics of a `text` vocabulary type be.  For example,
  should enumerating its contents produce code units, code points, or graphme clusters (as Swift does)?
  Arguably, for most uses, grapheme cluster enumeration is probably what is most useful; this is
  the approach that most enables users to not have to think about encoding issues as it prevents
  accidental slicing of grapheme clusters.  However, this also exposes complexity in terms of what
  it would mean for two elements to be considered equal.  Do their code point sequences have to
  match exactly?  What if their code point sequences are equivalent under Unicode normalization
  rules?

## Assignments:
- Tom: Get the use cases doc sufficiently up to date to solicit help.
- Everyone: contribute example solutions for use cases.
- Everyone: read the Swift 4 string manifesto and string changes documents.  Perhaps review the
  Swift string source code.
  - https://github.com/apple/swift-evolution/blob/master/proposals/0163-string-revision-1.md
  - https://github.com/apple/swift/blob/master/docs/StringManifesto.md
  - https://github.com/apple/swift/blob/master/stdlib/public/core/String.swift
- Tom: Test (Tom's) text_view view/iterators over (Zach's) text/rope containers.


# September 13th, 2017

## Draft agenda:
- Identify and discuss use cases that we would like to address and how our existing collection
  of proposals maps to those use cases.
- Goals for the group
  - For Albuquerque.
  - For beyond Albuquerque.
- General vision for C++ and Unicode support.
  - How do we merge what we're all working on into something cohesive and complementary?
  - What are the major gaps that remain?
- Goals and scheduling for the next meeting. 

## Meeting summary:
- Attendees:
  - Tom Honermann
  - Zach Laine
  - Michael Spencer
- We reviewed the UseCases.html document
  (http://htmlpreview.github.io/?https://github.com/tahonermann/std-text-wg/blob/master/UseCases.html)
  for its suitability in recording use cases applicable to our goals and as a presentation device for
  comparing existing solutions.
  - Consensus was that this document format will suffice and is useful.
  - The document currently only has placeholder examples for ICU, Win32, and QT.  Other libraries
    (POSIX, iconv) were briefly discussed for their applicability in addressing use cases.  Both
    POSIX and iconv are only relevant for a few of the use cases and we can add relevant examples
    where applicable.
  - It was noted that we don't have placeholder examples for Apple's Unicode libraries (Cocoa Text
    System).  We should research Apple's solutions as part of this effort.
  - It was also noted that comparisons with solutions available for other languages (Swift, Rust,
    Objective-C, etc...) may be useful.  Such examples would be welcome as well.
- Next up was identification of use cases.  We settled on the following high-level list:
  - Storage oriented (containers storing code units with an associated encoding):
    - String-like.
    - Rope-like.
  - Stream oriented (streams providing/consuming code units with an associated encoding):
    - Not interested in facet based solutions, but rather view/iterator-pair range adapters.
  - Encoding/decoding:
    - Run-time/locale dependent vs static.
    - Code point enumeration.
    - Grapheme cluster enumeration.
  - Generation/mutation:
    - Transcoding.
    - Normalization (of code point sequences; particularly Unicode normalization).
    - Append/insert (to a stored text object).
    - Remove (from a stored text object).
    - Splice/replace (within a stored text object).
    - Modify (a code unit sequence within a stored text object, possibly via a view,
      without insert/erase).
  - Algorithms:
    - Locale influence.
    - Upper/lower casing.
      - In place vs copy.
    - Search:
      - For a specific code point sequence (would not match differently normalized
        code point sequences).
      - For a code point sequence matching a normalized form.  (would match a differently
        normalized code point sequence; it was observed that a solution may not exist that
        performs better than first normalizing the text and search term and then performing
        an exact match search).
      - Boundary conditions (word boundary, grapheme boundary).
    - Comparisons/sorting:
      - Case sensitive/insensitive.
      - Lexicographical by code unit values.
      - Lexicographical by code point values.
      - Collation/locale based.
  - Character/code point classification (letter, number, case, combining, etc...).
- Adding example solutions to the use case document will take some effort, so we discussed
  soliciting help via the std-proposals list, reddit, QT maintainers, etc...  We agreed to
  do so once the doc reasonably reflects the use cases we care about for the immediate
  future.
- As a tangent, we discussed whether it might be worthwhile pursuing a requirement that
  source code encoded as UTF-8 (and perhaps UTF-16, UTF-32) with a BOM be supported.  This
  would enable portable use of characters outside the basic source character set in character
  and string literals.  Consensus was that this may be a feasible goal.
- The mailing deadline for Albuquerque is October 16th; a month from now.  We discussed what
  we might be able to have prepared by then.  Time is short for all of us, especially with
  CppCon occurring in two weeks.  One idea is to try and get the use cases document into
  sufficient shape such that we can submit it and present it as a general outline of the
  scope of features we're considering and a summary of existing practice.
- We agreed that, once we have a general outline of the kinds of solutions we're pursuing,
  then reaching out to the Unicode community would be a good action to take to confirm
  our direction.

## Assignments:
- Tom: Get the use cases doc sufficiently up to date to solicit help.
- Everyone: contribute example solutions for use cases.
- Tom: Test (Tom's) text_view view/iterators over (Zach's) text/rope containers.


# August 30th, 2017

## Draft agenda
- Provide feedback and compare perspectives on Zach's library.
- Goals for the group
  - For Albuquerque.
  - For beyond Albuquerque.
- General vision for C++ and Unicode support.
  - How do we merge what we're all working on into something cohesive and complementary?
  - What are the major gaps that remain?
- Goals and scheduling for the next meeting.

## Tasks completed:
- Mark got a Slack channel set up for us:
  - #std-text-wg on https://cpplang.slack.com

## Meeting summary:
- Attendees:
  - Tom Honermann
  - Beman Dawes
  - Zach Laine
  - Mark Zeren
- Feedback was presented to Zach for our reviews of his text library (https://github.com/tzlaine/text).
  Some of the discussion is captured below.
  - Could the library size be decreased by moving some components to their own library?
    The `rope` container for example.  It was noted that the curent `rope` class is not a generic
    container; it has text specific interfaces.  For example, constructors that take a
    `text` type, `substr()` member functions, and checks for well-formed UTF-8.
  - Does the Ranges TS already provide functionality similar to `repeated_text_view`?  The Ranges TS
    apparently does not, but `range-v3` provides similar functionality to repeat a single value a
    number of times via its `repeat_n()`.  A view projecting N iterations of a range seems generally
    useful and could potentially be included in the Ranges TS.
  - Could  `rope_view` be made constexpr?  Could it fit into a collection of constexpr container or
    view classes?
  - We discussed whether it is fair to categorize the `text` type as a minimalized `std::string` with
    an implied UTF-8 encoding and some degree of enforced encoding validation.  The answer seems
    to be yes (though Zach also has interest in enhancements for small string optimization and
    copy on write).  This raised the question of whether it is best to combine these features in a
    single type or via a layered interface.  This lead to a discussion regarding the reasons for
    manipulating text at the code unit level (when encoding validation during mutations is helpful)
    vs the code point level (which should obviate corruption of code unit sequences by ensuring
    mutations always occur at code unit sequence boundaries).
  - It was observed that, prior to mutating a string, code must already identify a position within
    the string to mutate.  Presumably, in the absence of a logic error, such identification will
    have already considered code unit sequence boundaries.  If so, encoding validation should be
    unnecessary and/or redundant (again, in the absence of defects).  This may be an argument for
    providing interfaces based on code points rather than code units.
  - We discussed error handling.  `text` provides iterators that throw on encoding/decoding error
    or that substitute a replacement character (or corresponding code unit sequence).  Beman
    indicated that this isn't sufficient; that additional error handling modes are necessary to
    provide more nuanced error handling in order to thwart potential security problems or to
    perform a more sophistocated character replacement action.  Mark also indicated the desire to
    terminate on error.
  - We also discussed handling of encoding validation errors that occur during text construction or
    mutation.  In these scenarios, character replacement isn't a reasonable option.  The `text`
    constructors throw exceptions as the least-bad way of reporting errors from constructors.  Tom
    noted that his `text_view` library avoids this problem by not performing validation during
    range construction, but rather detecting and reporting errors via iterators; errors are detected
    during iterator construction and advancement, but are not immediately reported.  Rather, a
    dereference of the iterator will either throw an exception or substitute a replacement character
    (depending on policy template parameters).  Additional member functions on the iterators allow
    querying if an error occurred before performing a dereference, if desired.
- We closed with a brief discussion of whether UTF-8 only solutions are acceptable.  Tom observed
  that UTF-8 has won the web and text interchange encoding wars, but that the encodings used as the
  internal encoding for applications is more varied.  We agreed that other encodings, particularly
  UTF-16 on Windows, will remain popular for the forseeable future and it therefore makes sense to
  provide new features that support at least UTF-16 and UTF-32 as well.
- We again went overtime and did not get through the draft agenda.
  
## Assignments:
- Everyone: Identify use cases that we want/expect to support and consider how our current proposals
  fit in.
- Mark volunteered to submit some UTF-8 test cases to Zach.
- Tom: Test (Tom's) text_view view/iterators over (Zach's) text/rope containers.


# August 16th, 2017

## Draft agenda:
- Administrivia.
- Status updates on works in progress and current plans for Albuquerque:
  - text_view (http://wg21.link/p0244)
  - Beman's encoding conversion interfaces (http://wg21.link/p0353)
  - char8_t (http://wg21.link/p0482)
  - Zach's text library (https://github.com/tzlaine/text)
  - Others?
- Goals for the group
  - For Albuquerque.
  - For beyond Albuquerque.
- General vision for C++ and Unicode support.
  - How do we merge what we're all working on into something cohesive and complementary?
  - What are the major gaps that remain?
- Goals and scheduling for the next meeting.

## Meeting summary:
- Attendees:
  - Tom Honermann
  - Beman Dawes
  - Zach Laine
  - Mark Zeren
- Like all good first meetings, we started off with technical difficulties; call-in by
  phone to the Bluejeans supplied phone number failed with a "contact support message".
  Audio via computer seemed to work well though; as did video.
- We agreed on using this GitHub repo for coordination.
- Mark suggested creating a slack channel for side discussion; we all agreed.
- We reviewed the current status and plans for the existing projects mentioned in the agenda.
  - Tom was advised to submit the version of text_view that does not rely on C++17 features
    to Boost.  Tom still has some work to do to cleanup error handling first.
  - Beman has re-worked his library to define overloads instead of variadic templates.
  - Zach is nearly ready to submit his library to Boost.
- Further discussion touched on:
  - There is general agreement that submitting our projects to Boost will be a good way to
    gain feedback and determine suitability for real world usage.
  - The feasibility of providing features (e.g., string types) specific to UTF-8, or that
    interact only with Unicode.  Concensus is mixed on this, but we agreed that the standard
    does not need to provide a direct solution for all use cases; a solution that works for
    90% (or so) of users is acceptable.
  - How our projects overlap.  For example, how closely aligned are Zach and Tom's text_view
    implementations and concepts?  It sounds like they are similar in intent, though there
    are some differences.  For example, Zach's implementation performs some encoding validation
    where as Tom's does not.
  - char8_t.  We are currently split on the utility of a new type.  A new type does not
    guarantee that data is properly encoded (e.g., UTF-8 string literals may not actually contain
    well-formed UTF-8).  A new type would enable overloading and differentiation between
    the execution encoding and UTF-8 encoding, but there is a sentiment that we should not
    differentiate between these and just work with UTF-8.
  - We are somewhat split on the importance of supporting non-Unicode encodings such as GB18030
    or the potential performance impact to systems like z/OS that would require frequent
    transcoding in order to work with Unicode-only facilities.
- The meeting went overtime; we did not discuss particular goals at this meeting.

## Assignments:
- Everyone to get more familiar with Zach's text library.
- Mark will get us setup with a channel on Slack.
