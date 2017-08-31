# Meeting Notes

The next meeting is scheduled for Wednesday, September 13th from 2:30pm-4:00pm EDT.

- [August 30th, 2017](#august-30th-2017)
- [August 16th, 2017](#august-16th-2017)

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
