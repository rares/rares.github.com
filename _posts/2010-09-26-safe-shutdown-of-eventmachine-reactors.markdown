---
layout: post
comments: false
sharing: false
date: 2010-09-26
categories: ruby eventmachine
---
<p>
  Recently I found myself trying to devise a way to prevent a
  process from shutting down until an EventMachine
  <a href="http://en.wikipedia.org/wiki/Reactor_pattern" target="_blank">Reactor Loop</a>
  was finished performing some network IO.
  I found a particular pattern quite effective and wanted to it share here.
</p>

<p>
  In-case you haven't done much work with EventMachine, it provides a mixin (called a <a href="http://eventmachine.rubyforge.org/EventMachine/Deferrable.html">Deferrable</a>) that allows
  the developer to model their activity in the reactor loop as callbacks (success and failure). Using this pattern,
  we can create a class to register
  pieces of work and unregister them when the work is complete. In this way, we can make assertions on whether
  or not it is safe to shut down a process or not depending or not if we have work left to complete.
</p>

<p>
  The exact situation I am trying to solve for is the following: I have a client that is pulling messages off
  of a queue and sending messages to another system (via another protocol). At any given time, we will have
  N number of IO jobs active in the event loop. But what happens when we send -TERM to this process? EventMachine
  starts closing these open connections and network calls that we want to succeed will fail. What I wanted was
  a way to register individual pieces of work (pulling the message off the queue and pushing to the remote service)
  and check to see if all the work is done (thus safe to allow the process to exit).
</p>

<p>
 So consider the following class:
</p>

```ruby
class DeferrableProgressMonitor
  attr_reader :total

  def initialize
    @marked, @total = [], 0
  end

  def register(deferrable)
    @total += 1
    @marked << deferrable

    deferrable.callback { |object| @marked.pop }
    deferrable.errback { |e, object| @marked.pop }

    nil
  end
  alias :<< :register

  def current
    @marked.size
  end

  def wait?
    @total > 0 && !@marked.empty?
  end
end
```

<p>Here is a small piece of code that uses it:</p>

```ruby
EM.run do
  monitor = DeferrableProgressMonitor.new
  trap("TERM") do
    unless monitor.wait?
      EM.stop
    end
  end

  monitor << thing_that_returns_a_deferrable
end
```

<p>
  So the signal trap will make a call to check to see if there is any outstanding work
  left. if there is, we keep on going (my actual code sets a timer that continually keeps checking #wait?).
  Otherwise, we shut the loop down.
</p>

<p>
  The are a few gotchas here:
  <ul>
    <li>
      The signature for Deferrable#callback and Deferrable#errback will fail if it
      doesn't match the call to Deferrable#set_deferred_status. So this pattern breaks down a bit as a general solution
      but works well if you have a well-defined activity whose completion is critical.
    </li>
    <li>
      Be mindful handling errors, anything that raises can cause you counters to be off, thus preventing
      the process without sending -KILL to it. You would want to try to catch what you can and utilize
      Deferrable#set_deferred_status to control success/failure.
    </li>
  </ul>
</p>
