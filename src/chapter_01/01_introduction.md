# Introduction

<div class="warning">

!!! DISCLAIMER !!!

This document is a work in progress. I aim to finish a first release in the next few weeks. Until then, this document might contain some minor errors as not all code examples have not yet been extensively tested.

</div>

## What are ranges?

<div class="warning">

TODO

Add short and concise introduction to what ranges are and what the range-v3 library is about.

</div>

## Why this document?

- Only little documentation to be found scatter across the web
- Eric Niebler himself -- the Author of range-v3 -- calls the official documentation "woefully incomplete"
- I want to have a concise overview of the library for myself to refer to
- I want to get more familiar with the library
- Moreover, since many views/algorithm/actions are anyway available in the STL, I will get more familiar with the STL as well
- Maybe someone else will find this useful, hence I make this document public

## Why not simply use C++20/23 ranges?

- range-v3 is more complete at this point
- C++20 only introduces a subset of the views available in range-v3
- C++23 extends the list of available views, but still does not add actions
- If your company is not keeping up to date with the latest C++ standards, range-v3 can be used with C++14-compatible compilers

## Other sources

A lot of the content in this documentation is loosely based on great articles, videos, and other forms of media.
Here is a (incomplete) list of resources that I used to learn more about the range-v3 library.

- The [official GitHub repository](https://github.com/ericniebler/range-v3), the official [documentation](https://ericniebler.github.io/range-v3/) and [resources mentioned therein](https://github.com/ericniebler/range-v3#documentation)
- Blog articles
  - [A beginner's guide to C++ Ranges and Views](https://hannes.hauswedell.net/post/2019/11/30/range_intro/) by Hannes Hauswedell
  - ...
- YouTube videos
  - [Ranges for the Standard Library](https://www.youtube.com/watch?v=mFUXNMfaciE&ab_channel=CppCon) by Eric Niebler at CppCon 2015
  - [New Algorithms in C++23](https://youtu.be/VZPKHqeUQqQ?si=MNZUNZWhdVVkMIxu) by Conor Hoekstra at CppNorth 2023
- Other software libraries
  - [Ranges tutorial](https://docs.seqan.de/seqan3/3-master-user/tutorial_ranges.html) of the [SeqAn3 library](https://github.com/seqan/seqan3)
- Books
  - [Fully Functional C++ with Range-v3](https://www.walletfox.com/publications_ranges.php) by WalletFox
