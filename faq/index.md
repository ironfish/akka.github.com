---
layout: page-full-width
title: Frequently Asked Questions
---

* auto-gen TOC:
{:toc}

## Akka Project

### Where does the name Akka come from?

It is the name of a beautiful Swedish [mountain](https://lh4.googleusercontent.com/-z28mTALX90E/UCOsd249TdI/AAAAAAAAAB0/zGyNNZla-zY/w442-h331/akka-beautiful-panorama.jpg)
up in the northern part of Sweden called Laponia. The mountain is also sometimes
called 'The Queen of Laponia'.

Akka is also the name of a goddess in the Sámi (the native Swedish population)
mythology. She is the goddess that stands for all the beauty and good in the
world. The mountain can be seen as the symbol of this goddess.

Also, the name AKKA is the a palindrome of letters A and K as in Actor Kernel.

Akka is also:
* the name of the goose that Nils traveled across Sweden on in [The Wonderful Adventures of Nils](http://en.wikipedia.org/wiki/The_Wonderful_Adventures_of_Nils) by the Swedish writer Selma Lagerlöf.
* the finnish word for 'nasty elderly woman' and the word for 'elder sister' in the Indian languages Tamil, Telugu, Kannada and Marathi.
* the word for 'earlier' in Korean, written like this: 아까
* a town in Morocco
* a near-earth asteroid

## Actors in General

### My actors send replies, but they are lost, and the sender is `deadLetters`

This happens most easily when using Java: a `tell()` operation needs to be told
what the sending actor’s reference is, so that this reference can be passed to
the target actor together with the message. When within an actor, always add
`getSelf()` as second argument:

    otherActor.tell(msg, getSelf());

For Scala users this can only happen outside of actors, since the `!` operator
picks up the sender reference implicitly from the surrounding actor’s context.

### How can I get compile time errors for missing messages in `receive`?

One solution to help you get a compile time warning for not handling a message
that you should be handling is to define your actors input and output messages
implementing base traits, and then do a match that the will be checked for
exhaustiveness.

Here is an example where the compiler will warn you that the match in
receive isn't exhaustive:

<pre class="prettyprint lang-scala linenums">
object MyActor {
  // these are the messages we accept
  sealed abstract trait Message
  case class FooMessage(foo: String) extends Message
  case class BarMessage(bar: Int) extends Message

  // these are the replies we send
  sealed abstract trait Reply
  case class BazMessage(foo: String) extends Reply
}

class MyActor extends Actor {
  import MyActor._
  def receive = {
    case message: Message ⇒ message match {
      case BarMessage(bar) => sender ! BazMessage("Got " + bar)
    }
  }
}
</pre>

## Remoting

### I want to send to a remote system but it does not do anything

Make sure that you have remoting enabled on both ends: client and server. Both
do need hostname and port configured, and you will need to know the port of the
server; the client can use an automatic port in most cases (i.e. configure port
zero). If both systems are running on the same network host, their ports must
be different

If you still do not see anything, look at what the logging of [remote
life-cycle
events](http://doc.akka.io/docs/akka/current/scala/remoting.html#Remote_Events)
tells you (normally logged at INFO level) or [switch on
logging](https://github.com/akka/akka/blob/v2.2.0/akka-remote/src/main/resources/reference.conf#L106)
of all sent and received messages (logged at DEBUG level).

### Which options shall I enable when debugging remoting issues?

Have a look at the `reference.conf` for [akka-remote](http://doc.akka.io/docs/akka/current/general/configuration.html#akka-remote), the typical candidates are:
* `akka.remote.log-sent-messages`
* `akka.remote.log-received-messages`
* `akka.remote.log-remote-lifecycle-events` (this also includes deserialization errors)

### What is the name of a remote actor?

When you want to send messages to an actor on a remote host, you need to know
its [full path](http://doc.akka.io/docs/akka/current/general/addressing.html),
which is of the form

    akka.protocol://system@host:1234/user/my/actor/hierarchy/path

Observe all the parts you need here:

* `protocol` is the protocol to be used to communicate with the remote system. 
   Most of the cases this is `tcp`.

* `system` is the remote system’s name (must match exactly, case-sensitive!)

* `host` is the remote system’s IP address or DNS name, and it must match that
  system’s configuration (i.e. `akka.remote.netty.hostname`)

* `1234` is the port number on which the remote system is listening for
  connections and receiving messages

* `/user/my/actor/hierarchy/path` is the absolute path of the remote actor in
  the remote system’s supervision hierarchy, including the system’s guardian
  (i.e. `/user`, there are others e.g. `/system` which hosts loggers, `/temp`
  which keeps temporary actor refs used with `ask()`, `/remote` which enables
  remote deployment, etc.); this matches how the actor prints its own `self`
  reference on the remote host, e.g. in log output.

### Why are replies not received from a remote actor?

The most common reason is that the local system’s name (i.e. the
`system@host:1234` part in the answer above) is not reachable from the remote
system’s network location, e.g. because `host` was configured to be `0.0.0.0`,
`localhost` or a NAT’ed IP address.

### sender/getSender() disappears when I use Future in my Actor, why?

When using future callbacks, inside actors you need to carefully avoid closing over
the containing actor’s reference, i.e. do not call methods or access mutable state
on the enclosing actor from within the callback. This breaks the actor encapsulation
and may introduce synchronization bugs and race conditions because the callback will
be scheduled concurrently to the enclosing actor. Unfortunately there is not yet a way
to detect these illegal accesses at compile time.

Read more about it in the docs for [Actors and the JMM](http://doc.akka.io/docs/akka/current/general/jmm.html#Actors_and_shared_mutable_state)

## Microkernel

### Why doesn’t `sbt dist` copy my subproject into the kernel distribution?

For this to happen, that project’s `name` (in the project’s settings) and `id`
(argument to the `Project` factory) must match exactly.

### How do I use Play2 Mini with Akka?

In case you need to start Play2 Mini programatically you can use::

    play.core.server.NettyServer.main(Array())

Read more about it in the docs for [Http](http://doc.akka.io/docs/akka/current/modules/http.html)

## Debugging

### How do I turn on debug logging?

To turn on debug logging in your actor system add the following to your configuration:

`akka.loglevel = DEBUG`

To enable different types of debug logging add the following to your configuration:

* `akka.actor.debug.receive` will log all messages sent to an actor if that actors `receive` method is a `LoggingReceive`

* `akka.actor.debug.autoreceive` will log all _special_ messages like `Kill`, `PoisonPill` e.t.c. sent to all actors

* `akka.actor.debug.lifecycle` will log all actor lifecycle events of all actors

Read more about it in the docs for [Testing Actor Systems](http://doc.akka.io/docs/akka/current/scala/testing.html#Tracing_Actor_Invocations)
