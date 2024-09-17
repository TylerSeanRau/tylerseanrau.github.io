---
layout: post
title: CppCon Day 1 2024!
---

Hello readers!!!

This year I have the great opportunity to attend
CppCon in person! Special thanks to my current
employer: [Boxbot](https://www.boxbot.io/) for
subsidizing the trip! (P.S. We're hiring)

---

## The sched

The talks I attended this day were:

- Peering forward — C++’s next decade - Herb Sutter

- Back to Basics: Unit Testing - Dave Steffen

- The Power of Reducing Variable Scope - Jason Turner

- When Lock-Free Still Isn't Enough: An Introduction
  to Wait-Free Programming and Concurrency
  Techniques - Daniel Anderson

- The Most Important Design Guideline is
  Testability - Jody Hagins

- How Meta Made Debugging Async Code Easier with
  Coroutines and Senders - Jessica Wong and
  Ian Petersen

---

I want to keep this short so I won't regurgitate the
hours of talks but I will note some key points:

* Reflection is shaping up to be an awesome tool for
  us to use. C++26 is shaping up nicely with it and
  hopefully more to come before feature freeze in 9
  months. Letting programmers more directly express
  intent will reduce codebase sizes and mental
  burden.

* Profiles, specifically safety profiles, seem like
  they'll be a wonderful addition to our toolbelt.
  An example of a safety profile would be making it
  so all std::vector::operator[] calls are bounds
  checked. From what I understood from Herb's talk
  these safety profiles being integrated in the
  standard mean you don't need to fiddle around
  with your (or library code) to enable them.

* Testing really is as important as I've always thought.
  On testing:
    * a quote from Jody in his talk was "The chicken believe's
      in breakfast but the pig is committed to breakfast".

      If you're shipping an algorithm that drives a car
      then at some point before shipping you better have
      been personally in the car during a real test where
      the hands are off the control.

      Be committed to testing. Don't just believe in it.

    * Companies live and die by testing. If you've got
      no tests, no testing strategy, no testing automation
      then you're not long for this world.

    * Testability is not apart of the list of software
      best practices. It IS the list. (Attempting to
      quote Dave, will update when I see the slides)

    * "Good tests kill flawed theories; we remain
      alive to guess again." - Karl Popper

    * Verifiable end to end tests dramatically more
      valuable than simple unit tests

* Debugging async code is hard but possible. Async
  stack traces from Folly and Unifex look like a great
  trend in the right direction.

* Helping is the critical piece of enabling wait-free
  code. When one thread sees an activity in progress
  when it is running then it should help not hurt
  that other thread's progress. (Paraphrased from
  Daniel Anderson)

---

Some citations needed, slides will be up on
[https://github.com/CppCon/CppCon2024](https://github.com/CppCon/CppCon2024) soon.

---

Thanks for your time,
Tyler Sean Rau
