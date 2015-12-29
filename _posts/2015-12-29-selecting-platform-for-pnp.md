---
layout: post
title: The daily selecting a platform (~language)
---


Comparing languages to me is a matter of comparing ecosystems.
Despite having strong opinions on a few things, I value practical decision making.
So for a new project i simply pick the language whichs platform ecosystem most matches my product, to benefit
from others work in a similar direction. This is open source.

Here i am comparing ecosystems for building a pick and place controller.
Openpnp is already written in java, which is probably a good choice. The reason i'm revalidating that choice
is because a processor that can run java is about twice the cost of something not supported by the jvm (for example mips).

I didn't read the code of openpnp yet because its GPL infected, but Jason is a smart guy,
so there is probably very little NIH ([not invented here](https://en.wikipedia.org/wiki/Not_invented_here)).
Let's go library shopping.


|Function       | Java | Ruby | Node.js | Python | Elixir | Go | Qt | Naked C++ |
|---------------|------|------|---------|--------|--------|-----|----|----|
|serial port    |  [good](https://docs.oracle.com/cd/E17802_01/products/products/javacomm/reference/api/javax/comm/SerialPort.html)   | [good](https://rubygems.org/gems/serialport/) |  [good](https://github.com/voodootikigod/node-serialport) | [good](https://github.com/pyserial/pyserial) | [good](https://github.com/bitgamma/elixir_serial)  |[good](https://github.com/tarm/serial)| [good](http://doc.qt.io/qt-5/qserialport.html) | [good](https://github.com/wjwwood/serial)  |
|cnc controll (gcode) | [good](https://github.com/callaa/JGCGen)   | [good](https://github.com/ArchimedesPi/gcodify)  | [good](https://github.com/ryansturmer/node-gcode)   |  [good] (https://pypi.python.org/pypi/gcodeutils/1.1)      |[good](https://github.com/suranyami/eliximote) |[good](https://github.com/joushou/gocnc)| [ok](https://github.com/zapmaker/GrblHoming)  |[ok](https://github.com/FictionIO/GCODE) |
|vision (opencv) | [good](http://opencv.org/opencv-java-api.html) | [good](https://github.com/ruby-opencv/ruby-opencv) [^1] | [good](https://github.com/peterbraden/node-opencv) | [good](http://docs.opencv.org/3.0-beta/doc/py_tutorials/py_tutorials.html) [^1] | [hopeless](http://stackoverflow.com/questions/6727889/opencv-on-erlang) |[good](https://github.com/lazywei/go-opencv)| no | [good] (http://opencv.org/) |
| eagle cad parser | [ok](https://github.com/hisashin/Eagle_Parser) | [good](https://github.com/gcds/libeagle) | [shitty](https://www.npmjs.com/package/eaglepunch) | [good](https://pypi.python.org/pypi/pyeagle/) | no | no| no | no |
| REST  server | [good](https://jersey.java.net/) | [good](http://www.sinatrarb.com/) | [good](https://github.com/restify/node-restify) | [good](http://flask.pocoo.org/) | [good](https://github.com/h4cc/awesome-elixir#rest-and-api) |[good](http://thenewstack.io/make-a-restful-json-api-go/)| [ok](http://stackoverflow.com/questions/11558237/creating-simple-webservice-in-c-qt-acting-as-server-providing-json-data)  [^2] | [bearable](https://github.com/eidheim/Simple-Web-Server) |
| Video Streaming | bad [^3] | [good](https://github.com/hubertlepicki/sinatra-flv-streaming-example) | [good](http://binaryjs.com/) | [good](http://blog.miguelgrinberg.com/post/video-streaming-with-flask) | ok [^4] |[good](https://github.com/sparks/rter/blob/master/prototype/videoserver/src/videoserver/videoserver.go)| [ok](http://derekmolloy.ie/custom-video-streaming-player-using-libvlc-and-qt/) | [good](http://stackoverflow.com/questions/26577025/c-opencv-webcam-stream-to-html) |

[^1]: ruby and python have a global interpreter lock. It's questionable how much processing can really be done inside their vms. Alternative vms fix that, but they won't work on mips either.
[^2]: i wrote half the stuff on that list so i'm going to say ok, although i know that "me" doesn't count as ecosystem. the truth is, the ecosystem for web services in Qt is terrible.
[^3]: I couldn't figure out how to run a rest thing plus a pure binary stream in the same context. This is really where the bloat of java design shows. Everything is just so not modular, despite using as many design patterns as possible.
[^4]: Massive scale video processing is amazing in erlang, but i just can't find anything out of the box as for other platforms.

Some of the libs have different goals, and its a bit comparing apples vs bananes, but the point is to get an idea of how the ecosystem of those platforms shares a common interest in the product i'm making.

As expected, ruby and python win at ecosystem hands down, even for something less web-y as a cnc machine. I couldn't really find a big difference either between the two. Pip is still garbage, and ruby still sucks at event handling. Nothing new and nothing fatal.

What suprised me is that the java ecosystem is alot better than C++, despite C++ being the obvious choice for 
good old machine builders. But that's probably because the C++ ecosystem is generally a complete desaster.
I really really wish elixir was good, but it just can't compete. I love elixir but i can never justifiy using it.

