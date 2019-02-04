---
layout: post
title: osaka.rs - rust async without the noise
---

This is a background story blog for [osaka.rs](https://github.com/aep/osaka) sort of.
For a quicker intro of osaka, check out the [readme on github instead](https://github.com/aep/osaka).

since the beginning of 2018 all of my companies/projects/lifes have switched over to rust as the one and
only programming language. There are several reasons for it, but to recap here are the important ones:

 - hiring pool: rust has an incredibly talented and diverse community.
 - just the right low levelism for embedded: rust has no GC or other runtime dependencies, but still adds high level concepts that reduce bugs.
 - ecosystem: crates.io is already amazing and ticks all the right boxes to grow to the size of gems or npm


I have plenty of praise for rust. The fact alone that i am now committing my companies such as [devguard](https://devguard.io)
completely on rust should tell how serious i am.

Unfortunately rust isn't focused on embedded exactly, so there's plenty workarounds that we've come up with over the course of shipping almost 30000 devices with an all-rust linux userspace. The biggest and most impactful is osaka, an alternative async system focused on readability and easy-to-argue code flow.



What async does
------

In short, asynchronous coding allows you to write code that waits for resources that are not immediately available to the CPU without using threads.
Typically on unix this is done via poll(2). You tell the kernel that your task currently cannot proceed until one out of a list of specified external resources becomes ready.

This is incredible important on embedded network devices, where almost all of the clock time is spent waiting for network packets. Threads would be very costly because they each carry an entire executable stack, and a 4MB ram device holding several millions of connections would simply not be possible.


What's up with async in rust
----------------------------


If you're following rust's story, you will see rust had several concepts for async, even coroutines. At the time of writing, it has sort of been settled on futures.rs and its main implementation [tokio](https://tokio.rs)

Futures are sort monads, but not really, and there's plenty unhealthy discussion about their relationship to monads. In my opinion, for all practical applications, they're monads with all the ups and downs.


```rust
let f1 = socket.recv();
let f2 = |packet|{
    Ok(decode(packet)?)
};

let future = f1.and_then(f2);
```

basically you're just passing a functor 1 another functor 2 that will be called when functor 1 is complete.
It's simple, it's obvious (sort of) and generic enough that every possible flow can be implemented.

There's a whole lot of writup out there by different rustaceans on why this needs improvement, like this one [http://aturon.github.io/2018/04/24/async-borrowing/](http://aturon.github.io/2018/04/24/async-borrowing/).

From a real life perspective (i.e. the reason you're reading this), futures.rs is adding so much code size that it's main feature of being 'zero cost' is a pretty odd claim.

![](https://pbs.twimg.com/media/DlO8C3zXoAEWGWQ.jpg:large)

Tokio additonally adds hefty memory requirements by being optimized for web servers. It's author is also a strong believer in edge triggered interrupts , although nobody else is. 
It's alot harder to implement a race free edge triggered system and it makes zero performance difference with epoll anyway. If you're using tokio in high performance code, get used to 
```
futures::task::current().notify()
```

My second biggest complaint (after obesity) with futures/tokio however is that its incredible hard to argue about correctness.
Where rust removes the massive burden of correctness checking for memory allocations, it adds more burden for external resources. Kind of the same mistake java made.
The entire execution engine is hidden in a singleton, cynically probably to make it easier to argue, because thats how theoretical systems like haskell work.


this is real code:

```rust
connect::connect(domain, secret.clone()).and_then(move |(ep, mut brk, sock, addr)| {
    subscriber::connect(target, ep, &mut brk, sock, addr, secret).and_then(move |mut channel| {
        channel
            .open(headers::Headers::with_path("/v0/self-update").and(":method".into(), "POST".into()))
            .expect("open channel")
            .into_future()
            .map_err(|(e, _)| e)
            .and_then(move |(headers, st)| {
                let headers = headers.expect("eof before header");
                let headers = headers::Headers::decode(&headers.to_vec())?;
                println!("{:?}", headers);
                Ok(st) as Result<_, Error>
            }).flatten_stream()
            .map_err(Error::from)
            .for_each(move |v: Bytes| {
                println!("{}", String::from_utf8_lossy(v.as_ref()));
                Ok(())
            }).and_then(|_| {
                drop(brk);
                drop(channel);
                Ok(())
            })


```


seriously, this is impossible to understand unless you have specific knowledge of how tokios internals work, which are by the way hidden.
Let's go through it step by step



```rust
connect::connect(domain, secret.clone()).and_then(move |(ep, mut brk, sock, addr)| {
    subscriber::connect(target, ep, &mut brk, sock, addr, secret).and_then(move |mut channel| {
```

the whole argument that combinators are easier to read than callbacks already falls flat at step zero, because combinators are implemented as callbacks,
which yes... they are hard to read because the argument types are implicit. who knows what ep is.


```rust
channel
    .open(headers::Headers::with_path("/v0/self-update").and(":method".into(), "POST".into()))
    .expect("open channel")
    .into_future()
```

an experienced tokio user will know that into_future hints that this thing is a stream. However, if you're new to tokio, this makes zero sense.
There's no way to know what the return type of open is, and into_future() has no semantic meaning anyway. It's just boilerplate to put one combinator into another combinator,
both of which are irrelevant for code flow.


```rust
}).and_then(|_| {
    drop(brk);
    drop(channel);
```

this is the most bizzare and abusive part. it moves a resource into a closure that is held inside the tokio task queue until the previous future completed. This is nessesary because combinators completely break RAII.
If you don't do this, you'll get absurd race conditions and madening problems like stuck task queues when a resource is droped before its task completes.




A promise of a greater future
----------------------------

Again, there's better writups than mine about the issues with futures.rs, and many agree. In fact, the author of futures.rs has moved on to become the main driver of yet another async system called [async/await](https://github.com/rust-lang/rfcs/pull/2394/files)
It's supposed to move from combinators to yield points, kind of like cooperative multitasking. The basic code flow is then:

```rust
let a = sock();
let b = await!(sock.read());
println!("{}", b);
```

or something. Nobody has agreed on the exact syntax yet, and the discussion is once again a bikeshedding monster trashfire. It won't matter much, as the underlying concept is both terrible and genious.
The genious part is that it's atually just a generator, which is exactly how i implemented async in the clay programming language:

```C++
async string something(Socket *socket) {
    Packet packet = with socket.receive();
    return packet.decode();
}

```

the rust equivalent would be slightly uglier, because generators can't have continuation arguments

```rust
fn something(Socket socket) -> AsyncResult<String> {
    match socket.receive() {
        Ok(packet) => return Ok(packet.decode()?);
        Again(again)   => {
            yield Again(again);
        }
    };
}

```

but still alot better than what the futures.rs authors decided to do: [cram futures.rs into std](https://github.com/rust-lang/rfcs/pull/2418/files) and add an entire language feature to hide the bloat behind a code generator.
A well constructed generator syntax is incredible generic and works with any async engine, however, rust has neither well constructed generators nor a generic execution engine. 

It must standardize on future.rs because it makes sense from the perspective a web programming language like ruby to have a consistent ecosystem of async stuff that works well together. And rust clearly is intended for web and desktop apps rather than embedded devices.

osaka: tokio without the noise
----------------------------

There where several dicussions how to implement an alternative async system outside of the tokio universe. I went as far as implementing golang coroutines

```rust
osaka!{
    sync fn foo(Socket socket) -> String {
        let pkt <- socket;
        pkt.decode()
    }
}

```

but backpeddled because from a practical perspective, it makes more sense to remain somewhat compatible with tokio. The nest best abstraction layer that i would consider technically correct is [mio](https://github.com/carllerche/mio).
It is essentially a cross platform implementation of poll, very similar to the use case of libevent.

So osaka is mostly a high level concept of using yield points to feed the mio polling machine. It is tied to mio with just a single type and can be ported to anything that has similar semantics (register to poller, poller sleeps and gives back activated events).
In theory it can be ported to bare metal.

Here's the syntax:

```rust
#[osaka]
pub fn something(sock: Socket) -> Result<String> {
    let pkt = sync!(sock);
    Ok(sock.decode()?)
}
```

its in no way fancy, but has two significant advantages over async/await. Firstly, it emits a generator without a specific execution engine. the execution engine is passed as argument, if you need it.
Secondly, Result just works as intended. There's no need to wrap it in FutureResult because no such type exists. Whether your function returns Result or another type is up to you.
Lastly, sync is just:

```rust
loop{
    match bar {
        Ready(a) => break a,
        Again(a) => yield a,
    }
}
```

so anything that returns Ready/Again is sync. There is no task queue, no singleton, nothing 'magic'.
Here's some real life code:


```rust
#[osaka]
pub fn something(poll: Poll) -> Result<Vec<String>, std::io::Error> {
    let sock    = mio::UdpSocket::bind(&"0.0.0.0:0".parse().unwrap())?;
    let token   = poll.register(&sock, mio::Ready::readable(), mio::PollOpt::level()).unwrap();

    loop {
        let mut buf = vec![0; 1024];
        if let Err(e) = sock.recv_from(&mut buf) {
            if e.kind() == std::io::ErrorKind::WouldBlock {
                yield poll.again(token, Some(Duration::from_secs(1)));
            }
        }
    }
}

pub fn main() {
    let poll = osaka::Poll::new();
    something(poll).run().unwrap();
}
```


as you can see, in this case we're passing osaka::Poll as execution engine, which is essentially mio::Poll. 
We're also explicitly implementing the code that checks for EAGAIN. osaka is much much less magic than tokio, 
which also means there's some parts that require more explicit handling. osaka is intended for embedded, or other use cases where you already know how the underlying resources behave.
While in theory this could be abstracted away better, i'm not sure if it should, since tokio already does a good job for those use cases.


but what about the ecosystem
----------------------------

During a heated [twitter debate](https://twitter.com/arvidep/status/1090639300960665600), i made it quite clear that building a new async ecosystem for embedded is justified because the alternative is not using tokio. The alternative is nothing, since tokio does not and will never work on embedded.

Building things with osaka is much quicker than with tokio, and there are only so many things that are actually needed. There's already a DNS resolver for example, and [devguard](https://devguard.io) brings an entire encrypted peer to peer message broker.
Again, if you're building a webservice, chances are tokio is the better choice anyway.

Devguard has changed from tokio to osaka and lost a nice 60% binary size, and 80% memory usage while slashing tons of racing bugs and dangling resources, simply by making things simpler.
As it is with most engineering, sometimes the simpler path is less glorious, but also more stable.
