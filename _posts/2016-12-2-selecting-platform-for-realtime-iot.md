---
layout: post
title: The daily selecting a platform (~language) (IoT)
---


Comparing languages to me is a matter of comparing ecosystems.
Despite having strong opinions on a few things, I value practical decision making.
So for a new project i simply pick the language whichs platform ecosystem most matches my product, to benefit
from others work in a similar direction. This is open source.

Here i am comparing ecosystems for building an IoT realtime messaging platform.
I'm starting from an existing MVP at superscale.io, which needs a rewrite.
Superscale does managed network equiptment, where the hardware sends and receives controll messages which a middleware translates to and from REST.

Superscale was written in [elixir](http://elixir-lang.org/), which was selected because of its erlang base,
which works amazingly well for message based systems.

However, when making that choice, I did not take one category serious enough: hiring.
Finding good elixir developers was such an incredible difficult experience, that i decided to rebuild the platform before this kills the company.

The other category i complete ignored is tooling for deployment and operations.
I was simply assuming Erlang - the vm elixir runs on - is so mature that this is a solved problem.
It turns out, it is solved in a classic 90s way: hire more people to ssh into servers and do stuff by hand.
Debugging erlang is horrible.

I also learned that scaling is a massive problem with erlang. It does not actually scale that well, which is incredible, considered it was designed for that.
The reason is mostly that erlang's basic design decisions have been made back when servers where assumed to grow in numbers of cpus on a common bus. Nowadays we have lots of small machines connected over lossy network (cloud and iot and everything i do really)

Starting there, here's my standard approach comparing the platforms for this product.

Ecosystem
---------


| Component     | Java | Golang | Node.js | Python | Elixir | Ruby | Kite/C++ |
|---------------|------|--------|---------|--------|--------|-----------|------|
| MQTT          | [good](https://eclipse.org/paho/clients/java/) | [good](https://eclipse.org/paho/clients/golang/) | [good](https://www.npmjs.com/package/mqtt) | [good](https://eclipse.org/paho/clients/python/) | [reasonable](https://github.com/gausby/gen_mqtt/) | [good](https://github.com/njh/ruby-mqtt) | [good](https://github.com/aep/kite) |
| Redis         | [good](https://redis.io/clients#java) | [good](https://redis.io/clients#go) | [good](https://github.com/NodeRedis/node_redis) | [good](https://pypi.python.org/pypi/redis)  | [good](https://github.com/whatyouhide/redix) | [good](https://github.com/redis/redis-rb)   |  [reasonable](https://redis.io/clients#c--) |
| Websockets    | [good](https://github.com/jetty-project/embedded-jetty-websocket-examples) | [good](https://github.com/gorilla/websocket) | [good](http://socket.io/) | [reasonable](https://www.fullstackpython.com/websockets.html) | [good](https://www.viget.com/articles/websockets-with-elixir-how-to-sync-multiple-clients) | [bad](https://github.com/igrigorik/em-websocket) | bad |
| Bug Collection | [good](https://github.com/bugsnag/bugsnag-java) | [good](https://github.com/bugsnag/bugsnag-go) | [good](https://github.com/bugsnag/bugsnag-node) | [good](https://github.com/bugsnag/bugsnag-python) | [reasonable](https://github.com/jarednorman/bugsnag-elixir) |  [good](https://github.com/bugsnag/bugsnag-ruby)   |  [bad](https://github.com/google/breakpad) |
| APM           |  [good](https://newrelic.com/java) | [good](https://newrelic.com/golang) | [good](https://newrelic.com/nodejs) | [good](https://newrelic.com/python) | horrible | [great!](https://newrelic.com/ruby) | bad |
| AWS           |  [good](https://aws.amazon.com/sdk-for-java/) | [good](https://github.com/aws/aws-sdk-go) | [good](https://aws.amazon.com/sdk-for-node-js/) | [good](https://aws.amazon.com/python/) | [bad](https://github.com/CargoSense/ex_aws) | [good](https://aws.amazon.com/sdk-for-ruby/)| bad |

[^1]: java has shittons of websocket servers, but  could not find a single one that was designed for shared resources.


Typical ways fanboys of that platform solve these problems i have
------------

| Concept       | Java | Golang | Node.js | Python | Elixir | Ruby | Kite/C++ |
|---------------|------|--------|---------|--------|--------|------|----------|
| cluster agreement  | [awful](https://zookeeper.apache.org/) | [amazing](https://www.consul.io/) | [weird](https://github.com/uber/ringpop-node) | no [^2] | [bad](https://github.com/ostinelli/syn) | no [^2] | nothing |
| simple rest api | [ugly](https://www.nabisoft.com/tutorials/java-ee/producing-and-consuming-json-or-xml-in-java-rest-services-with-jersey-and-jackson) [^3] | [ugly](https://tutorialedge.net/creating-simple-restful-json-api-with-go) [^4] | [good](https://www.thepolyglotdeveloper.com/2015/10/create-a-simple-restful-api-with-node-js/) | [ugly](https://impythonist.wordpress.com/2015/07/12/build-an-api-under-30-lines-of-code-with-python-and-flask/) [^5] | [good](https://robots.thoughtbot.com/testing-a-phoenix-elixir-json-api) | [good](http://www.sinatrarb.com/) |  [poor](https://github.com/aep/LazerSharks)|
| Deps Management | [awful](https://zeroturnaround.com/rebellabs/java-build-tools-how-dependency-management-works-with-maven-gradle-and-ant-ivy/) | [ugly](https://github.com/golang/go/wiki/PackageManagementTools) | [good](http://npmjs.com) | [awful](http://docs.activestate.com/activepython/3.2/diveintopython3/html/packaging.html) | [good](https://hex.pm/) [^6] | [good](https://rubygems.org/) | awful |
| C10M | awful [^7] | [good](https://goroutines.com/10m) | [good](https://blog.jayway.com/2015/04/13/600k-concurrent-websocket-connections-on-aws-using-node-js/)  | [bad](http://squirl.nightmare.com/medusa/async_sockets.html) | [good](http://www.phoenixframework.org/blog/the-road-to-2-million-websocket-connections) | horrible [^8] | good |

Hiring
------------

let's talk about hiring. I do active sourcing only, because i'm not facebook, so good people simply don't apply. Actice sourcing requires a large community to poke into, to extract candidates out of.


| Community     | Java | Golang | Node.js | Python | Elixir | Ruby | C++ |
|---------------|------|--------|---------|--------|--------|------|----------|
| Github.com    | 2.5M | 200K   | 2.8M    | 1.3M   | 18K    | 1.2M | 600K     |
| Stackoverflow | 35%  |     ?  |  17%    | 24%    | 0.1%   | 9%   | 19%      |
| google trends | 1    | 18     | 5       | 2      | ?      | 12   | 6        |


java wins any popularity contest, hands down. It's also common knowledge that any java position gets 10x the applications (or actually gets applications). Doesn't say anything about quality. Python,ruby,nodejs are also pretty popular, not suprising. Elixir is so unpopular, it's barely on any charts.
Golang is suprisingly less popular than i expected, and C++ still refuses to die.



Shortlist
--------------

Looking at the pure facts, there's only Java, Golang and Nodejs left.
Ruby doesn't scale for events, Python is biased towards demos and has poor behaviour in production. Elixir has a poor ecosystem. C++ is just dead dead.
I also looked at haskell. I like haskell, but i still can't convince myself it's a good idea to build a company on it.

beyond the hard facts, it comes down to my personal taste, so i implemented an mvp in all 3.

Java
-----

My business is speed. Not compile speed, not execution speed, but development speed.
People hire me because i can build entire companies in 6 months.
One of the tricks is avoiding work where possible, copypasta as much as possible from the interwebs.

Java kinda sucks at that, for two reasons:

- java stackoverflow is [full of shit](http://stackoverflow.com/questions/tagged/java)
- java is fractured. the most popular stack is android
- everyone and their mum wants to sell a [framework rather than a solution](http://google.com/search?q=java+websocke)
- Java people simply don't understand the internet as a [technology](http://www.javaworld.com/article/2853780/core-java/socket-programming-for-scalable-systems.html) or community (age related i guess). 

The popularity of java quickly becomes less relevant when you look at a framework that [isn't spring](http://zeroturnaround.com/rebellabs/the-curious-coders-java-web-frameworks-comparison-spring-mvc-grails-vaadin-gwt-wicket-play-struts-and-jsf/)

Yet, the last point is maybe the most critical for me. I have a clear policy to design for network partition, and unclean shutdown (loosing FIN, missing SIGTERM).
Everyone assumes this is an edge case, until their companies get *murdered* by DDOS, or simply by leaking their own connections.

Ladies and gentleman, i present to you: the internet. Network loss is not an edge case, 
sorry your university professor has probably never owned a company that looses money when you fuck up.

Trick question, what happens when you loose ACK after header here: [https://www.mkyong.com/java/how-to-send-http-request-getpost-in-java/](https://www.mkyong.com/java/how-to-send-http-request-getpost-in-java/), or here in a framework [https://github.com/Mashape/unirest-java/blob/master/src/main/java/com/mashape/unirest/http/HttpClientHelper.java#L138](https://github.com/Mashape/unirest-java/blob/master/src/main/java/com/mashape/unirest/http/HttpClientHelper.java#L138)

I could rant about this on any platform, because it's such an ubiquitous mistake people make when fresh out of uni,
but it's its a larger issue in java, because it matters here. Leaks are poison in a long running process/thread based platform.
In platforms like ruby, php or perl, it's not relevant, because the OS cleans up after each restful call.

Now concerning the actual code. I didn't want to bother writing java, since i'ts the only language on earth that is uncomfortable to write in vim, so i looked at the jetty example for websockets.


{% highlight java %}
ServletContextHandler context = new ServletContextHandler(ServletContextHandler.SESSIONS);
context.setContextPath("/");
server.setHandler(context);
ServletHolder holderEvents = new ServletHolder("ws-events", EventServlet.class);
context.addServlet(holderEvents, "/events/*");
{% endhighlight %}

then that servelet thing is of course just a factory

{% highlight java %}
public class EventServlet extends WebSocketServlet
{
    @Override
    public void configure(WebSocketServletFactory factory)
    {
        factory.register(EventSocket.class);
    }
}
{% endhighlight %}

and finally here's the relevant part

{% highlight java %}
public class EventSocket extends WebSocketAdapter
{
    @Override
    public void onWebSocketText(String message)
    {
        super.onWebSocketText(message);
        System.out.println("Received TEXT message: " + message);
    }

{% endhighlight %}

honestly, i find java relatively nice to read, which is probably why it's used for cheap outsourcing so much.


Golang
-----

In go i implemented [lifeline](https://github.com/thingops/lifelined) already, so i know its behaviour running it at scale.

Beauty matters if you lead a team and you need to read other peoples code all day.
Sorry to break it to you golang, but you're ugly as hell. And this isn't just me [https://www.quora.com/Do-you-feel-that-golang-is-ugly](https://www.quora.com/Do-you-feel-that-golang-is-ugly).
It feels like go just "happened" rather than having been designed, very much like C++.


{% highlight go %}
func LatestSystemImageHandler(r *http.Request) *RestfulReturn {
    var product = r.URL.Query().Get("product")
    var vendor  = r.URL.Query().Get("vendor")

    var rs []models.SystemImage;
    tt := segments.StartDatabaseSegment(r, "system_images", "select");
    err := db.Model(&rs).
    Where("product = ?", product).
    Where("vendor  = ?", vendor).
    Where("enabled = true").
    Order("version DESC").
    Limit(5).Select();
    tt.End();
    if err != nil {
        return NewRestfulPgError(err)
    }

    return &RestfulReturn{
        Data:           SnakeCase{rs},
        PageSize:       5,
        PageNumber:     1,
        TotalEntries:   len(rs),
        TotalPages:     1,
    }
}
{% endhighlight %}


for most of the primitives of the language, i can find absolutely no reason why they are designed this way, and if you ask on irc, you'll get some nasty religious bullshit. Or you're simply declared stupid. This reminds me of C++ where bad language design is still defended with dubious arguments about some theoretical future optimization, because honestly most people have no idea about language design.

But Golang really shines when it comes to pragmatic solutions to difficult design problems.
Inheritance for example, it just doesnt have it, so you cant do it wrong.
Generics? nope. Exception bubbling and stack unrolling? Nah.

Go gets away with just not implementing a whole lot of features.
That means code is uglier, but more likely to be correct.

Personally i think reading ruby is beautiful, because you can hide complexity, so i can understand what something does with my super short attention span.

{% highlight ruby %}
    user = User.where(name: 'hans')
    user.send!("yo buddy!")
{% endhighlight %}

gophers have the counter argument that hiding complexity means writing corect code is harder.
The exact behaviour of "where" and "send!" is unclear, so its easy to use wrong.
This is where convention comes in, but that's something that only ruby people seem to get in general.

In go we instead do this:

{% highlight go %}
    var user User;
    err := db.Unmarshal(&user).where(Query{name: 'hans'})
    if err != nil {
        //ah come on
    }
    err := MessageSender.send(user.ID, "yo buddy!")
    if err != nil {
        //this is boring
    }
{% endhighlight %}

It's not that bad when you compare it to C++, but kinda stinks knowing that the language could have been prettier.

Go really amazed me when it comes to production deployment. I find the static binary thing not that interesting.
It's cool but whatever, docker already solved this.
What really convinces me is that go is statically typed and compiled to machine code,
which means code that worked once has a pretty good chance of working forever. 
Go only adds a garbage collector, which makes it unusable for embedded, but oh well.

This feels like everything i was missing when i came from C++ to ruby. It compiles, ship it!

Tsting is really crap in go btw, very much like C++. Look at this fugly test code

{% highlight go %}
t.Run("update formation name ", func(t *testing.T) {
   req, err := http.NewRequest("PATCH", "/v2/formation_groups/" + fg.Id, nil)
   req.Body = ioutil.NopCloser(bytes.NewBuffer([]byte("{\"name\":\"yupsies\"}")))
   if err != nil {
       t.Fatal(err)
   }
   rr := httptest.NewRecorder()
   handler.ServeHTTP(rr, req)
   assertReturn(t, rr, 200);

   req, err = http.NewRequest("GET", "/v2/formation_groups/" + fg.Id, nil)
   if err != nil {
       t.Fatal(err)
   }
   rr = httptest.NewRecorder()
   handler.ServeHTTP(rr, req)
   assertReturn(t, rr, 200);

   rm := &RestfulReturn{}
   err = json.NewDecoder(rr.Body).Decode(rm)
   if err != nil {
       t.Fatal(err)
   }
   dt := rm.Data.(map[string]interface{})
   if (dt["name"] != "yupsies") {
       t.Fatalf("fg from get not named as expected");
   }
)
{% endhighlight %}*

that is incredible hard to read, and i honestly would not know what it does if i wouldn't have written it.

Now the part that almost made me drop go alltogether is its nasty java heritage.

go is fucking religious about packaging. You absolutely have to use git repos instead of directories or submodules, or everything falls appart.
Oh and there is [no sane way to have private repos](https://gist.github.com/shurcooL/6927554). That's a deal killer for any professional (=propriatary) work.

GOPATH is fucking broken by design, it is designed around these assumptions:

- you have no friends or coworkers (you cannot share workspace state)
- you only write golang (you cannot have subdirs in git repos)
- you only work on a single project (dependencies are global per computer)
- all your code is on github (you cannot have private repos)

This will inevitably lead the the rise of [alternative build chains](https://github.com/golang/go/wiki/PackageManagementTools). Remember java before maven? Hell, just hell.

Overcoming this is still one of the biggest strugle working with go, and working in a team will mean several weeks of finding a development process that works at all with CI and humans with different setups and all of that. This is in every of my circle ci files:

````
  override:
    - go get github.com/mattn/gom
    - gom install
    - mkdir -p vendor/src/
    - mv vendor/g* vendor/src/
    - gom build
```

and this is a typical Docker file:

````
RUN mkdir -p /go/src/liberator
WORKDIR /go/src/liberator
COPY . /go/src/liberator
RUN gom install
RUN gom build
```

go toolchain has different behaviour depending on pathname of $PWD.
code *must be in a directory named "src" during compilation* but at the same time *reposities must not have a directory called src*. 
This is just shit and [everyone](https://news.ycombinator.com/item?id=9508508), [has](http://skife.org/golang/2013/03/24/go_dev_env.html) , [incompatible](http://programmingaregood.logdown.com/posts/748682-a-not-so-ugly-go-development-environment-using-gb), [workarounds](http://stackoverflow.com/questions/29119496/storing-a-go-project-with-other-non-go-projects)


Nodejs
-----



It's strange, but i don't mind javascript. It has [so](http://www.pixelstech.net/article/1354210754-10-design-flaws-of-JavaScript) , [many](http://wiki.c2.com/?JavaScriptFlaws) ,  [problems](https://whydoesitsuck.com/why-does-javascript-suck/) that make it incredible hard to avoid bugs, - kind of of like C++ - , but the community isn't using them to make sure noobs know they're noobs. It might be because JS came from browser, where everything is a little more product focused.

That focus makes it so comfortable for me. It has all those libraries that are optimized to solve small focused use cases. Node.js is the most MVP optimized platform i know, even more than rails, simply because the people using it understand MVP. Websocket server? Sure, here's some 10 liner, i googl'd in 10 seconds and understood in 10 seconds:


{% highlight javascript %}
var ws = require("nodejs-websocket")
var server = ws.createServer(function (conn) {
    console.log("New connection")
    conn.on("text", function (str) {
        console.log("Received "+str)
        conn.sendText(str.toUpperCase()+"!!!")
    })
    conn.on("close", function (code, reason) {
        console.log("Connection closed")
    })
}).listen(8001)
{% endhighlight %}

Oh my, nodejs is the princess of MVP. I can't believe how beautiful this is. Wheren't it for the problems being so obvious:

- javascript code is absolutely unmaintainable. There are only a handful of companies using nodejs at scale. (uber, paypal)
- nodejs ecosystem has serious ADHD. There's a major shift every month.
- single thread evented is the best thing ever, unless you need cpu, then you die.
- javascript in production is just [horror](http://www.slideshare.net/sergimansilla/architecting-large-nodejs-applications-14912706) and [pain](http://blog.denivip.ru/index.php/2013/01/pitfalls-in-large-scale-application-development-on-node-js/?lang=en)


nodejs makes me sad, because it's crazy good, but unusable because the underlying language still sucks. The only hope is to wait it out, because nodejs so large, and the issues are well understood, that eventually it'll be solved.



Tips for running golang
--------
So far i'm running lifeline with 3K connected devices and it's working fine. I leaked a channel once (forgot to timeout FIN, hah) and could 
find it without much trouble using [influx metrics](https://github.com/vrischmann/go-metrics-influxdb), but there's plenty more tools.


- include time for solving GOPATH pains with the team, every week.
- use channels rarely. don't use libraries that use channel.
- split your projects into subrepos and microservices as much as possible.
- use a vendoring tool to fix the issues with subrepos.
- find a testing strategy that involves not using go test
- don't expect go to be fun. it's like java. it's made for work, not fun.


.


[^2]: i couldn't find much code in python and ruby communities about their approach to cluster consensus, but lots of theoretical blabla. This might be a problem with my attention span.
[^3]: the general consensus in the java community appears to be on xml being the standard exchange format. nobody uses xml. xml sucks.
[^4]: go developers "gophers" hate convention over configuration. Well, i love it, and most of their code is full of boring configuration. Also go has very little frameworks.
[^5]: All "simple" examples i found are poorly structured, abd use abstractions at the wrong layers.
[^6]: hex is pretty good, but elixir also inherited erlangs rebar, which is absolute hell. it reminds me of autoconf
[^7]: i literally stood up and left the lecture when my prof said the internet is syncronous and requires threads. java seems to be designed around the same stupid idea that network is cpu bound somehow. As a consequence, it becomes cpu bound through all the concurrency crap people do. Most blogs i found say that you need to tune the vm locks and other bullshit.
[^8]: ruby people do alot of things very well, but this is one of the problems they just don't understand. Most blogs i found on the topic are simply wrong or offload the issue to something like resque. This feels very similar in python (not suprising, considered their similarity)


