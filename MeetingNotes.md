# Meeting Notes

The next meeting is scheduled for Wednesday, August 30th at 2:30pm EDT.

[August 16th, 2017](#august-16th-2017)

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
