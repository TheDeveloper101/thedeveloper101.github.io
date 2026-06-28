---
title: "Preventing Timing Side Channels in High Level Synthesis"
excerpt: "A type system extension to enable secure HLS without timing side channels"
collection: portfolio
---

High level synthesis (HLS) is an attractive approach to designing specialized hardware accelerators. It allows for users to write code in a "high level" language such as C++, and then rely on the compiler to synthesize hardware that matches the code. However, HLS on its own is not able to produce secure or predictable hardware. The research language and compiler Dahlia is able to bridge some of this gap by providing a novel type system to generate predictable hardware, but cannot provide security guarantees. I worked on this project with two of my classmates as a part of CMSC838L - a graduate level course on Programming Languages and Architecture. We extended the Dahlia compiler to ensure that it generates secure hardware that is resistant to timing side-channels by using a technique known as information flow control. The code can be found [here](https://github.com/zbrachinara/dahlia). A detailed writeup of our work can be found below:
<iframe src="{{ site.baseurl }}/files/Final Report-2.pdf" width="100%" height="400px" style="border: none;"> </iframe>
