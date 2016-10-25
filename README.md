# pi

Greenthreading in zepto with channels.

It is extremely primitive and small (45 LOC in total).

## Installation
```
zeps install hellerve/pi
```

## Usage

The two main concepts in pi are channels and actors.

Actors are created by forking, i.e. saying that a function
should be scheduled to run at some point (thread-like).
They can also `yield`, which means they tell the scheduler
they want to give up their role as currently running and
be awoken when it's their next turn.

Channels are a way for actors to communicate, i.e. send
values from one actor to another. Actors can put values
into the channel and take from the channel. If the channel
is empty, the call to `chan:take` will block the current actor
until it can provide a value.

Let's dive into an example.

```clojure
(define channel (chan:empty)

(fork (lambda ()
        (begin
          (write "first thread")
          (write (++ "Received: " (chan:take channel))))))
(fork (lambda ()
        (begin
          (write "second thread")
          (channel:put "Hello, I will be printed"))))
(fork (lambda ()
        (begin
          (write "third thread")
          (channel:put "Noone will take me :(")))))
(yield)
```

This is short, but dense, and it is hard to wrap your head around
what will happen. So, let's get through this line by line.

First, we create an empty channel. Then we `fork` a thread. It
will print out `first thread` and then be put back to sleep. The
second thread will print and the put something into the channel.
Then the third thread will awake, print and put something into the
channel. At this point, the first thread will be woken up, realize that
there is something in the channel and happily print
`Received: Hello, I will be printed`. The second value in the channel
will not be taken and so will be swallowed when the program terminates.
The call to `yield` at the end ensures the main thread gives up control
flow.

## How?

The magic of continuations as values and mutable global data (_yuck_).

<br/>
Have fun!
