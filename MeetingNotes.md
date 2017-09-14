# Meeting Notes

The next meeting is scheduled for Wednesday, September 27th from 2:30pm-4:00pm EDT.

- [September 13th, 2017](#september-13th-2017)
- [August 30th, 2017](#august-30th-2017)
- [August 16th, 2017](#august-16th-2017)

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
