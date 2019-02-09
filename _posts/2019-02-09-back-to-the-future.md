---
layout: post
title: back to the future
---


This is a follow up to [my original rant about futures](http://aep.github.io/rust-async-without-the-noise/).

Wow, this kinda blew up and there's been alot more attention than anticipated.
Which only shows that there is a lot of interest in the state of async in rust.

Besides the usual useless comments on how i am a terrible person (thank you random stranger on the internet),
there was actually alot of very good feedback. Some people went through the entire article and picked it apart
for technical details. The rust community is amazing.

The biggest point i need to address is about futures 0.3 and how the post doesn't actually offer any learnings
for the _new_ futures, since all the reasons i offered why futures are broken only applies to future 0.1.
But before we get to that, i want to reply to a few other comments.

> but it's not at all clear from this blog post how Osaka handles multiplexing connection-oriented sockets.

For completeness, here's how you poll multiple things:

```rust
#[osaka]
fn foo(poll: Poll, a: Stream, b: Stream) {
  loop {
    let again = match a.read() {
      Ready(something) => return something,
      Again(again) => again,
    };
    match b.read() {
       Ready(something) => return something,
       Again(again2) => yield again.merge(again2)
    };
  }
}
```

this is very explicit. It could be hidden behind a macro, but these are easy to build once the basic concept is clear.

> osaka uses more std than futures

osaka is... not good code. And its unfortunate that this rather rough implementation got mixed 
into the actual point i'm trying to bring to the rust core devs that futures is the wrong interface.

I cannot implement osaka the way that it should be implemented without forking rustc. 
Hence i was hoping for  the attention of rust devs to implement async as a generic interface rather than as future,
so it becomes reusable.

A good interface would be Fn->Fn.  In a perfect world like the clay programming language, 
generating yield points for Fn->Fn is part of the core language like so:

```
async string something(Socket *socket) {
    Packet packet = with socket.receive();
    return packet.decode();
}
```

In rust that is a Generator. Almost. Since generators can't take arguments, 
we need to wrap the continuation arguments into much more complex structures, such as channels.
It's more complex and we need more trickery to implement the same thing, 
which futures.rs gracefully pushes onto tokio in a symbiosis that led to std::futures effecively implementing an interface for tokio.

osaka unlike futures actually does this

```rust
impl Future for X
where X: std::ops::Generator
```

so you *can* use any generator as a future, and its something futures.rs should be doing rather than pushing futures into std with a type signature that dictates how you need to implement the execution engine (hint: it needs to be like tokio)



> There are example of embedded execution engines that are not tokio

No there are not. All the ones that have been linked in comments are re-implementations of tokio. 
I find it also very frustrating that litteraly all of them are "toy implementations" 
and the argument is that "futures was designed for this use case", but zero of these authors have any background in embedded.

I can surely apprechiate that the rust core team in good faith is trying to make embedded work, 
but consistently dismissing feedback from people who actually ship real products because "it works in theory" is pretty infuriating.


Yet for the sake of objectively arguing with strangers on the internet i'm actually going to implement an execution engine for futures 0.3 in this post.



futures under realistic constraints
===============================

While i can rage about tweets that dismiss embedded as an uninteresting fringe from the rust core team all day, 
there's actually a rather large group of readers out there who care about objective facts. 
So i'm picking some real life constraints here to benchmark against:

**flash size**

We'll pick two targets: a mips24k board running linux with 4Mb flash (hint: that's very small) and a cortex m0 with 256kb of ram (hint: that is pretty big). 


**runtime/os**

on the mips target we do not have GNU libc for license reasons (and because its massive). we'll roll with the vendor default frankenstein libc.

on the cortex m we have no threading primitives at all because we need to run cooperatively with the vendor binary code which does radio telemetry.




a slow start
============

Starting with arm i created a simple fake async resource:

```rust
struct Later(u32);
impl Future for Later {
    type Output = u32;
    fn poll(self: std::pin::Pin<&mut Self>, wk: &std::task::LocalWaker) -> Poll<u32> {
        Poll::Ready(self.0)
    }
}

async fn boop(beep: Later) -> u32 {
    let a = await!(beep);
    a * 2
}

fn main() {
    let then = boop(Later(32));
}

```

that's a reasonable interface until here/ Pretty close to Fn->Fn

**Except all of this is inside std, so no embedded!**

```
error[E0433]: failed to resolve: use of undeclared type or module `std`
 --> src/main.rs:4:5
  |
4 | use std::future::{Future};
  |     ^^^ use of undeclared type or module `std`

```

I wasn't expecting to give up this early, considered the repeated claim that futures is designed for embedded,
but maybe i'm doing it wrong?

moving on to mips/linux then, which at least has an std implementation.



the next thing we need is sort of poll that thing. Could be pretty easy, since all we need is this waker thing to call poll.
Apparantly it you're supposed to construct it from this function:

```
pub unsafe fn local_waker<W>(wake: Arc<W>) -> LocalWaker

```
Arc is a pretty scary requirement, since that requires locking primitives to workd.
An Arc is also not 'zero cost', depending on your cpu (bad on mips) but moving on, trying to implement a "Wake" since that's required. 

But we're getting stuck here:

```
38 |     Pin::new(&mut then).poll(&wk);
   |     ^^^^^^^^ within `impl std::future::Future`, the trait `std::marker::Unpin` is not implemented for `std::future::GenFuture<[static generator@src/main.rs:17:35: 20:2 beep:Later {Later, ()}]>`
```

Well, I guess you can't pin stack values? No idea why, but apparantly it works fine when you move it on the heap:

```rust
    let mut then = Box::pin(boop(Later(32)));
    let wk = unsafe { std::task::local_waker(std::sync::Arc::new(Wakething{}))};
    println!("{:?}", Pin::new(&mut then).poll(&wk));
```

not sure how that falls in line with "better than callbacks because fewer pointer indirections", but i'm not sure if that matters either.


interrupting all the focuses
============

next thing we want to do is implement a microcontroller wakeup scenario. We can't actually test it because fututures doesn't actually compile for arm.. but we'll at least validate the claim that it was designed for embedded in theory.

Assuming you somehow managed to implement all the thread primitives for your platform, which is tons of work, we'll now need to wakeup tasks from real interrupts somehow.


we'll simulate an interrupt like this:
```rust
let external_event = thread::spawn(||{
    thread::sleep(time::Duration::from_millis(10));
    interrupt();
});
```


Getting a future to complete when the interrupt triggers, is easy in clay:

```
global future = some_future_stack;
interrupt(event) {
    future(event)
}
```

since rust does not allow us to pass arguments into generators (or futures), we need to use a channel.
std::sync::mpsc::channel is nice, and we can easily use try_recv inside pollErr

```rust
struct Interrupt (mpsc::Receiver<u32>);
impl Future for Interrupt {
    type Output = u32;
    fn poll(self: std::pin::Pin<&mut Self>, wk: &std::task::LocalWaker) -> Poll<u32> {
        match self.0.try_recv() {
            Ok(t)  => Poll::Ready(t),
            Err(_) => Poll::Pending,
        }
    }
}

```

if we poll this thing, it returns immediately with "Pending" without any additional context what it is waiting for.
This is a major difference to osaka.
We need some way to know when to call poll() again. Ideally, we'd just call it from interrupt() because we know its ready then,
or we could return some sort of token in Pending to tell an external caller that we're waiting for the channel to become ready.

Waker does not let us access the inner type either, so

**the only available option is to do exactly what tokio does**


```rust
thread_local! {
    static REACTOR: Arc<Reactor> = Arc::new(Reactor::new());
}

fn poll(self: std::pin::Pin<&mut Self>, wk: &std::task::LocalWaker) -> Poll<u32> {
    REACTOR.with(|reactor| reactor.register(&self));

```


note the implementation of await!:

```rust
macro_rules! await {
[..]
$crate::future::poll_with_tls_waker(unsafe {
    $crate::pin::Pin::new_unchecked(&mut pinned)
})
[..]
```

requires TLS anyway. So if you do not have a working TLS implementation, or don't want to afford the extreme overhead, you're pretty screwed anyway.
In my case rustc emits incorrect TLS frames for static binaries on mips24k, so it crashes immediately.
On microcontrollers, TLS is very hard to stabilize. [I implemented a linker](https://github.com/aep/elfkit) and can tell tales of how fricking broken TLS is. The whole idea is insane.


We'll move on simulating everything on x86_64 , which is clearly the only platform that futures is intended for.


standing at the edge
====================

Still, the claim that you CAN implement a different behaviour than tokio stands on being able to implement level triggered behaviour,
which is simpler, more stable and easier to work with, rather than the edge triggered that tokio enforces (because its easier to port to windows apparantly).
We'll try with a simple linux udp server using mio.


```rust


lazy_static! {
    static ref POLL : std::sync::Arc<mio::Poll> =  std::sync::Arc::new(mio::Poll::new().unwrap());
}


struct Udp{
    sock: UdpSocket,
}


impl Future for Udp {
    type Output = Vec<u8>;
    fn poll(self: std::pin::Pin<&mut Self>, wk: &std::task::LocalWaker) -> Poll<Vec<u8>> {

        POLL.register(&self.sock, mio::Token(0), mio::Ready::readable(), mio::PollOpt::level()).unwrap();

        let mut b = vec![0;1024];
        match self.sock.recv_from(&mut b) {
            Ok((len, _))  => {
                b.truncate(len);
                Poll::Ready(b)
            }
            Err(_) => Poll::Pending,
        }
    }
}

```


first attempt: not so good. you can of course not re-register to poll.

```
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Os { code: 17, kind: AlreadyExists, message: "File exists" }', src/libcore/result.rs:997:5
```

tokio solves this with an extreme amount of overhead per wakeup. We'll try to be a little better by moving the registration into the constructor.

```rust
impl Udp{
    pub fn new() -> Self {
        let sock = UdpSocket::bind(&"127.0.0.1:11111".parse().unwrap()).unwrap();
        POLL.register(&sock, mio::Token(0), mio::Ready::readable(), mio::PollOpt::level()).unwrap();
        Self{sock}
    }
}

```

and voila, it works.

```
Pending
Ready("bla\n")
```


trying to do it in a loop is a little confusing, since this doesnt work:

```rust
async fn boop(beep: Udp) {
    loop {
        let a = await!(beep);
                       ^^^^ value moved here, in previous iteration of loop
```

apparantly you need to clone it, or use an iterator interface, so more allocations for the 'zero cost much better than fn->fn' thing,
but finally i got it working. here is the full code: [https://gist.github.com/aep/bd44329ddc58c81efb242932fa59aece](https://gist.github.com/aep/bd44329ddc58c81efb242932fa59aece)

So it is possible.




conclusion
===========


unfortunately there was no way to benchmark futures on the target devices for real, since it doesnt even compile. We'll compare the theoretical impact

|                      | tokio        | futures 0.3 | osaka      | fn->fn      |
|----------------------|--------------|-------------|------------|-------------|
| size bloat           | refference   | smaller     | smaller    | zero        |
| activation overhead  | extreme      | extreme     | still bad  | single jump |
| works on cortex m    | no           | no          | not yet    | yes         |
| works on mips        | no           | no          | yes        | yes         |
| TLS singleton        | yes          | yes         | no         | no          |
| per activation alloc | yes          | yes         | no         | no          |
| push model           | no           | no          | yes        | yes         |
| pull model           | yes          | yes         | yes        | yes         |
| edge triggered       | yes          | yes         | yes        | yes         |
| level triggered      | no           | yes         | yes        | yes         |




As i said earlier, everything is possible to do in everything. You can run doom in a browser and some people will be happy with that.
osaka exists because futures is too expensive for embedded and that would be perfectly fine if async didn't emit a future, but a more generic interface.

