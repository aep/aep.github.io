---

layout: post

title: so long rust, why i built ZZ

---


My company [devguard](https://devguard.io) was - by numbers of deployments - the largest user of rust before we stopped using rust entirely.

So that plus me not being subtle about it, leaves people to question *their* choice of rust. I'll try to answer that as best as i can.



Let me first establish the context of what we do. Devguard is an embedded systems company.

We primarily do IoT at massive scale with insane deadlines on impossibly cheap hardware.

The reason devguard is a successful company without any sales whatsoever, is that we do stuff nobody can do.


With that in mind, the first question you should ask yourself when questioning if rust is right for you is: are you doing anything unusual?

The rust strategy - like almost all large communities -  is very much prioritized by popularity. 



**If you do x86 cloud based web services, rust is a good choice**


We at devguard do not do any of those things. Our biggest customer is mips custom chips, which is eternally broken on rust due to very novice toolchain design.


rust has great good support for building apps that work on linux, windows and mac. Very much like Qt.

If you target anything else, you'll be [fighting upstream](https://github.com/rust-lang/rust/issues?q=is%3Aissue+is%3Aopen+mips+label%3AO-MIPS)



**If you have no responsibility, you can prioritize fun over results**


Does your company or project have to make money? Does anything burn if my software fails?

If it's a startup, both of those are likely no, so you're fine.


Rust's primary mission is to be [*empowering*](https://www.rust-lang.org/). What that vague mission statement in my experience means is that they're welcome to newcomers. That's a *good* thing for business, because less onboarding cost.


What it however also means is that they prioritize that over deep technical knowledge.

Yes, coding can be fun, and I love seeing the creative ideas that come out of the maker community ... at the maker fare, not in automotive emergency braking systems. Sometimes rules have reasons, and sometimes you need to shut up and read the [scientific papers](https://books.google.de/books?id=ewNtCQAAQBAJ) of people who do in fact know better.



**If you have a cult, who cares about interoperability**


My biggest and most personally frustrating complaint about rust is that it somehow became a purpose on its own, rather than a tool to do something.


For example there's a cognitive dissonance thing going on that rust has [no runtime cost](https://users.rust-lang.org/t/not-quite-zero-cost-abstraction/11514/5), because that's what they advertise, and anyone finding workarounds for the actual very real runtime cost will feel the wrath of [the cult](https://www.theregister.co.uk/2020/01/21/rust_actix_web_framework_maintainer_quits/).


Rust makes bold claims about being a system programming language, that is if you're willing to [build a new system around rust](https://www.esp32.com/viewtopic.php?t=499) because any of the millions of dollars that chip vendors put in SDKs are [somehow not useful](https://github.com/rust-embedded/book/issues/62)


The recent ecosystem break with async has put the nail in the coffin for us, because again they put the burden of actually making their poor design work [on the community](https://ferrous-systems.com/blog/embedded-async-await/), again hiding the true runtime cost (TLS) away from the user while ignoring any prior work on the topic.



**Where do we go from here?**


Is rust a bad language? no! Should you use it? yes!


That is unless you need to do anything commercial near hardware, then you should probably stay with the tools that have a huge amount of tooling to deal with their aged design. This is why i'm building [zz](https://github.com/aep/zz). Drawing from rusts enthusiasm to make system programming less terrible, but also from decades of experience building hardware products at scale.


The two main missions are:

 - make raw pointers safe at compile time
 - improve the existing C ecosystem


We can finally have all those good tools that web service programmers have, but for real life embedded.

Could you build web services with it? Probably, but I do think nodejs is the defacto standard there, which is why zz also compiles to nodejs modules     automagically. Essentially ZZ is all about acknowledging that ecosystems already exist, and there's no reason to replace them.



