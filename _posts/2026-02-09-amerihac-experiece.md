---
title: 'My experience at AmeriHac'
date: 2026-02-09
permalink: /posts/2026/02/amerihac-experience/
tags:
  - haskell
  - compilers
  - wasm
  - amerihac
---

This past weekend, I participated in the inagural North American Haskell Hackathon - AmeriHac! It was so much fun, despite not even knowing a single bit of Haskell going into it. I wanna write up a bit about my experience, some lessons I learned and things I hope to do in the future :)

TL;DR - Myself and some friends built a simple Racket compiler in Haskell, with two different backends for Wasm and x86-64. I focused primarily on the Wasm backend and learned a lot. Code can be found [here](https://github.com/TheDeveloper101/haskell-learning).

The AmeriHac Experience
======
So as I mentioned, I didn't have _any_ experience with Haskell coming into this hackathon. I knew a lot about it from many friends, but I had never used it before. So how did I find out about this hackathon then? Well it turns out that the director of the Haskell foundation - José - used to teach at my university! He left for the Haskell foundation during my freshman year, but many of my friends happened to take classes that he taught. They heard about the fact that this was happening, and I figured I would come along as a fun weekend trip.

In the months leading up to the hackathon, there was a general sense of "oh I should _probably_ learn a bit of haskell so that I can try to contribute to the ecosystem." 

Uhhhh...

In the spirit of Haskell and lazy evaluation, I only got around to learning Haskell the day before the hackathon started. We decided to come up to NYC on Friday so that we wouldn't be too stressed. After checking into the hotel, I decided to start learning. That night I only really got around to some basic syntax and learning the concept of a typeclass. Saturday morning, we went to the location of the hackathon, which was located at Jane Street HQ! They were an extremely generous host and it made for an amazing experience. That morning, I started reading a little bit about what a monad was. At the same time, I met another student - Dylan - who is a sophomore at UMass Amherst and a big Haskell fan. MAJOR shoutout to him for all of his help. I learned a lot from him about Haskell, and without him I wouldn't have been able to do even a third of the stuff I did this weekend. After the opening keynote (which was super cool) I, along with my friends, met with José to see what a good introductory project would be.

The Project
------
At UMD, we have a course title [CMSC430](https://www.cs.umd.edu/class/spring2026/cmsc430/) which teaches students how to build a (fairly full featured) compiler from scratch. It aims to compile Racket, and is written in Racket. The compiler is built in a very iterative manner, making sure that at each step we have a fully functioning compiler that students can use. Since pretty much every person in our group had either worked on compilers or took the class, we were suggested to re-implement as much of CMSC430 as possible in Haskell. Dylan also joined our group because he wanted to learn more about compilers. With 6 total people on this project, we decided to split things up. The idea was to define the AST first, so that half the group could work on the frontend, and the other half of the group could work on codegen and the backend. We decided we were **not** going to try to implement functions (which made things very easy). The features we aimed to support correspond to the "Hoax" language in CMSC430. This includes strings, vectors, branching, variables, heap pointers (using box), comparison, basic arithmetic, IO, branching and loops. We quickly defined an AST which was a very easy transcription from the course notes, and got to work.

Spoiler alert: we did not implement everything we wanted to.

The interlude
------
About midway through the first day, shortly after lunch, we had an opportunity for people to advertise projects and ask for collaborators. One of the projects advertised was a rewrite of the [sigma-proofs](https://github.com/sigma-rs/sigma-proofs) library in Haskell. While I didn't know anything about ZK-proofs, it seemed really cool so my thought was that I would check it out, talk to the people working on it and then come back to the compiler project. After talking for a bit, I was asked if I wanted to contribute! I was excited as the project seemed really cool. I knew quite a bit of Rust, so I figured translating from Rust to Haskell could still be a viable approach to learning. I spent a bit of time reading up about how the Schnorr identification protocol worked, which we wanted to implement, and then started reading through the protocol specification. I was able to contribute a little bit around the formation of linear relations, and learned a lot about designing good code in Haskell. In particular, I learned when to use typeclasses vs data types and what should be implemented vs leaving to users to implement. I worked until 5pm, which is when the building closed, and then met back up with friends.

Back to the project
------
While I had fun trying to contribute to the Haskell rewrite, it was honestly a bit unsatisfying because I didn't know anything about ZK. In a world where I knew more, I probably would have just stuck with it and contributed further. However, in comparison to ZK, I know a lot more about compilers. The group decided to continue working a bit after the building closed, and I listened to their progress. I decided that I wanted to come back and work on the Racket compiler in Haskell project. I was most interested in the backend and codegen in particular. Something that I had been thinking about in the back of my mind was to learn about Wasm. I figured since I am already learning Haskell, and I want to work on the backend, I might as well also learn Wasm and build a Wasm backend for our compiler. Since we had four people working on the backend, we decided to split into two teams of two. One team would work on the x86 backend, and the other would work on the Wasm backend. After dinner, we all came back to the hotel room and continued work on the compiler. I helped with creating a set of helper functions to represent various datatypes by tagging the least significant bits, and creating simple ways to convert back and forth between our AST and the raw bits that would be given to the compiler.

While that was important and necessary work, I also wanted to start learning about Wasm. I found the [haskell-wasm](https://github.com/SPY/haskell-wasm) package, and decided to start reading the documentation.

*Crickets...*

There was no documentation. At all. And I barely knew the very basics of Haskell. This will be fun. Through a lot of pain and effort, I managed to (somehow) get a function to emit Wasm code for integer literals to typecheck, and then fell asleep. It was around 3:30am when I had stopped. 

I got up the next day at 8am, and despite having less than 5 hours of sleep, I was super excited to continue working on the Wasm backend. After getting to Jane Street, I added support to emit boolean and character literals while eating breakfast. Since I added helpers to distringuish and convert between types via biit manipulation the night before, adding support for these literals was quite trivial. Just call the correct helper function to tag the bitstring, and emit an `i64c` instruction with the bitstring. However, I really wanted to check that the code I was generating was actually meaningful. With loads of help from Dylan, we defined a full Wasm compilation pipeline. It turned out what I was emitting was essentially function local stack variables. So, we wrapped the code in a dummy function (since we weren't going to actually compile functions), created a dummy module to hold our dummy function, validated it, and then invoked our module as an "exported module."

Yeah it was very jank.

The output we got was a Wasm integer, so using the helper functions I had created before, we were able to convert the integer back into our AST datatypes. And.... voilà! We managed to get integer, boolean, and character literals working! The next thing we wanted to implement were unary operators. This is the next place we got stuck.

The first unary operator we wanted to implement was `add1 e`. It is quite simple. First, it evaluates `e` and checks that it is an integer. If so, it then increments it by 1. Otherwise, you get a runtime exception. We wanted to use the `inc` instruction from Wasm. However, haskell-wasm has these typeclasses called `Producer` and `Consumer`. The rough idea as far as I can tell is that a `Producer` can create data and a `Consumer` can consume data. Well to use the `inc` instruction, we need the expression we want to increment to be both a `Producer` and a `Consumer`. There are only three things that satisfy those requirements: globals, locals, or functions. We spent a lot of time trying to figure out how to pass the result of compiling `e` into the function we use for compiling `add1`. It turns out that you can just pass it in. The solution was way easier than expected. We were worried that we needed to do some sort of checking since haskell-wasm is supposed to be a type-safe DSL, but it turns out that through the magic of Haskell types, it just works! My understanding of this is that the argument is not only required to be a `Producer` and a `Consumer`, but also it needed to be of the specific int64 data kind that the library uses. In order to uphold this contract, it automatically checks the type of the argument and ensures you only pass in arguments of the correct type.

I learned what a data kind is! The Haskell type system is very cool.

Ultimately though, the `inc` instruction desguared into an `add` instruction, and we figured it would just be easier to use the `add` instruction. To be honest I don't quite remember why I did that, but hey it works. Once we figured out `add1`, implementing it's dual `sub1` was quite trivial. The next unary operator we wanted to implement was `zero?`. Now implementing this in x86 is pretty straightforward since we store all of our values in registers, and you can use `cmp` to compare the register to 0. Then you can use `cmov` to move the boolean value into the return register. While implementing the core logic was easy, we weren't able to get the correct representation of boolean. In haskell-wasm, they don't support Wasm if statements for whatever reason. So we had to get creative. We ended up creating a separate bit representation of types for the Wasm backend in comparison to the x86 backend. This made it so that I could modify the representation of booleans. We just swapped the representation of true and false - this was because `eqz` returns a 1 if true and a 0 if false. By doing so, I could uniformly apply a bitshift and then a constant add and consistently ensure that I got the right representation for booleans after a comparison operator. 

Another headache with comparison operators, is that for whatever reason they operate on the I32 data kind. However, all of our code to this point had been using and working with the I64 data kind. We panicked for a bit, before we realized we could also just use an `extend_u` instruction to go back to 64 bit and everything would work :) I think that there should be some type magic I can work to make this be generic over the two data kinds, but I was still learning, and I had no idea how to do that. The hackathon ended shortly after, and we all said goodbye before going our separate ways. My friends and I later went to round1 and then got back to the hotel. That night, I added a bit more support for binary operations - namely addition and subtraction, as well as another unary operation to test if something is a character. 

Lessons Learned
======
Overall I had a LOT of fun, and learned a lot. I have a couple thoughts I would like to share here.

Documentation
-----
Oh boy. Where do I even begin? The haskell-wasm package had basically no documentation, and the only comments were just parts of the code that was commented out. The type signatures were very useful once I figured out what they meant, but even getting there took a while. Also, with a package as big as haskell-wasm, the lack of documentation made it really hard to understand the intended approach to doing things and how everything was meant to compose together. I _think_ I understand most of it, and hopefully this project will serve as an example to anyone who wants to try to use haskell-wasm in the future. It was quite hard to understand the intent without any examples. Also, Haskell has a lot of really cool tooling around types (shoutout hoogle), but haskell-wasm wasn't being indexed for whatever reason? This just added a lot of friction and made things hard.

LLMs
------
I did not use a single LLM this weekend when implementing my code! Would I have made more progress? Most certainly. But not using LLMs allowed me to really understand at a deeper level what was happening and how everything worked. I feel like I learned way more doing everything myself and it made for a really great experience.

Haskell
------
I learned a ton about Haskell, including some more advanced (for me) features such as the function composition operator, `fmap` and do notations for basic operations involving monads. There is still a lot to learn, and I am sure that experienced Haskellers will have many suggestions when looking at the code. The Haskell type system is quite helpful, and even though the errors may not be _as_ good as cargo, they are still quite helpful. Also, it was usually pretty obvious what the errors were when I made them, so I didn't feel the lack of cargo errors that much. Also, GHC extensions are WILD. I only learned the bare minimum about Data kinds, but I feel like my brain expanded so much. It's so cool to see a lot of cool type system stuff advanced in Haskell, and I look forward to learning about more GHC extensions.

Conclusion
======
I had so much fun this past weekend learning about Haskell and Wasm as well as building a compiler. Compilers are super neat and fun and I can't wait to learn and do more!
