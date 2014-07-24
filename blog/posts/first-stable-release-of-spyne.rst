.. title: First stable release of Spyne
.. slug: first-stable-release-of-spyne
.. date: 2012-10-29 20:35:22 UTC
.. tags: 
.. link: 
.. description: 
.. type: text


My first getting involved with Soaplib was, as it is with every other open
source project, to scratch an itch. And a big one at that: Web development.

I first had the chance to get my hands dirty with web development (i.e. actually
ship a project) back when PHP 4 was rocking the internets, and I absolutely
hated it. The whole PHP/HTML/Javascript/CSS/AJAX stack was a big stinking mess
of ugly hacks on top of other ugly hacks. Yes, it all kind of worked. But at a
huge cost: The blood and tears of web developers who tirelessly memorized the
quirks of every single browser version/OS combination under the sun. This is
why, to this day, we have a distinction between “Frontend Engineers” and
“Backend Engineers”.

Doing server-side development was mildly better, because at least the lucky ones
could avoid having to deal with multiple runtimes. Still, the capriciousness of
the client demanded a high amount of flexibility from the server-side code,
which made it very easy for web developers to shoot themselves in the foot –
there just were too many moving parts that were so difficult to abstract away.
So we ended up chock-full with embarrassing bugs like “SQL injection” which are
actually trivial to avoid if one could find a single sane moment to think about
it.

While working towards my masters degree in the safe confines of sound
theoretical foundations, I had developed a certain kind of disdain for the
ad-hoc world of web development, which I had to delve into every now and then
due to some personal obligations. When I finally finished my studies and found
myself back in front of a browser full-time, I knew I had to do something.

You know you have been through a decent academic institution if your first
reflex is to conduct a comprehensive survey of prior art when tasked with
solving any kind of problem. While doing my share of studying existing
approaches to web development, the pattern I could not help noticing was that
HTTP, initially conceived as a simple way of transferring hypertext, was
actually being used as an RPC protocol, and especially so with the emergence of
non-browser clients in the web development scene. The browser technology as well
as cross-browser compatibility libraries had come a long way as well: Browsers
had decent Javascript support and much better standards-compliance and a
plethora of Javascript libraries smoothed over the remaining quirks. So in fact,
we did not need the amount of flexibility we used to need three years ago, but
were (and still mostly are) stuck with the tools and mentality developed in the
early stages of the world wide web.

So those of you who know what I have been (publicly) doing for the past couple
of years will inevitably wonder: Are you suggesting the Simple Object Access
Protocol (Soap for short) as a better Http?

Hell no. Today, it is very fashionable to hate Soap and no, I won’t try to
pretend that it’s not for good reasons. It’s not well-defined, complex,
inefficient, bloated, so on and so forth. Assuming for a brief moment that Xml
is a suitable serializer for inter-node messaging, I think the only thing that
Soap did right was to extend the Xml Schema standard which, in my humble
opinion, is a well-thought-out standard that does what it is supposed to do
painlessly.

Of course, when I stepped up as the maintainer of Soaplib back in 2009, my
opinions on this matter were not as crystal clear as they were today. I just
wanted my server-side functions to accept and return well-defined data without
worrying about what the client would do with them, and Soap simply looked like
it was up for the task by trying to be the champion of well-definedness.
Soaplib, Rpclib and Spyne

Originally created by Aaron Bickell, Soaplib 0.8.1 was a barely functional
prototype of a very promising design. After my taking over Soaplib’s
maintainership, my first goal was to have the original design actually work.
After a fast-paced development that saw releases 0.9 through 0.9.4, Soaplib 1.0
was released. It was just a Soap server that only supported Http encapsulation
via Wsgi. Soap was trapped in its own walled garden.

I quickly went on to work on soaplib 2.0, which sought to transform the current
code base to a more modular architecture where one could re-use Soaplib code
without having to deal with the Wsgi api. When that goal was also reached, I
knew I was done with Soap. So I started adding support for other protocols to
the Soaplib codebase, which made the name of the project rather inappropriate.
To stay loyal to the roots of the project, I decided to rename it to Rpclib.

Rpclib also saw a lot of releases and actually managed to attract a bunch of
brave souls who based their production code on Rpclib’s buggy api that was in
constant flux. Some tirelessly reported or fixed bugs and sometimes added entire
chapters to the documentation. (seriously guys, I can’t thank you enough) At the
time of my visit to Santa Clara where I attended PyCon 2012, Rpclib had come a
long way from what Soaplib was almost two years ago, to the version 2.7.0-beta.
My impression about the state of the project was that it mostly worked, yet
lacked polish. The thing I was not sure about was exactly how much.

Fate rolled its dice and I ended up giving a lightning talk about Rpclib. I
think it was well-received, because I received a lot of positive comments about
the project, as well as constructive criticism about how I should improve the
project to increase its adoption. There, it became obvious that most people will
stay away, no questions asked, from software which came with a beta tag. Oh and
it’s also quite painful to pronounce “Rpclib”. So I renamed the project to Spyne
and took on the task of making the next release a release candidate. As per the
semi-formal versioning contract, this meant stabilizing the api, revising the
documentation word by word, generally squashing bugs that popped up from
everywhere (with the invaluable help of the early adopters), and while doing all
that adding new features that we at Arskom needed.

I started an endeavor with my first commit to Soaplib repo in May of 2009. With
the release of Spyne-2.9.0 in September 29th, 2012, I consider that endeavor to
be fulfilled. Now it’s time to build on top of Spyne and patch its shortcomings
along the way.
What is Spyne?

Spyne is an RPC toolkit that imposes a clear distinction between Model, View and
Controller, where View is split in two as “Protocol” and “Transport”. This means
the user function (Controller) that has well-defined input and output types
(Model) deals only with native Python types. The protocol handles
(de)serialization and sanitization of incoming and outgoing data and the
transport is responsible for handling concurrency and moving the bits around.
Spyne also supports automatically generating interface definition documents from
api definitions, (as of now, only WSDL 1.1 is supported) pluggable asynchronous
task queues, and more. You can read the “Introduction to Spyne Concepts”
document for a more detailed explanation.

Protocols, transports and interface documents are completely pluggable. It’s
e.g. possible to use SOAP over ZeroMQ or Json via HTTP without touching user
code. It’s also possible to accept requests via one protocol and respond via
another.

However, not every protocol has equivalent capabilities. For example, if you
want to return structured data, you should use Xml or Json as output protocol
instead of HttpRpc, as pure HTTP can only return a byte stream in response to a
request, so only supports primitive return values. Spyne does its best for
bridging such gaps in functionality, for example by flattening request object
hierarchy where necessary. (E.g. By default, this ({"call": {"obj": {"i": 5,
"s": "str"}}}) JsonDocument request is equivalent to the following HttpRpc
request: /call?obj_i=5&obj_s=str)

Not every transport is equal either. E.g. Twisted’s HTTP implementation holds
the whole HTTP request in memory, so is not really suitable to receive big file
uploads via HTTP. (See the infamous bug #288 for more info.)

Finally, producing server-side HTML with Spyne is also possible but not yet
fully supported. There are experimental output-only protocols that serialize
data as Html tables or Html MicroFormats. It remains to be seen whether or how
these would fit into the current web development landscape.

So if you’re interested in using or helping with Spyne, head over to the docs
and start hacking! For patches, comments and suggestions, do not hesitate to get
in touch with us. Relevant information is in the readme document.

