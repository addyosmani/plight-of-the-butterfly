# Object.observe()

**Introduction**

Object.observe() is a proposed method for natively observing data changes in 
JavaScript. It's slated for ECMAScript 7 and is currently implemented behind the 
experimental JavaScript flag in Chrome Canary thanks to Rafael Weinstein. With 
some luck, we'll see it land in more browsers in the future.

**So, what are we observing?**

* Changes to raw JavaScript objects
* When properties get added, changed, deleted
* When arrays have elements spliced in and out of them
* and so on.

**Why do we care about observing data changes?**

_Show markup + model animation (2)_

* In short, Model-view control separation
* HTML is a great declarative mechanism, but it's totally static
* Ideally you just want to declare the relationship between your data and the 
  DOM and keep the DOM up to date. This creates leverage.
* Saves you a lot of time writing really repetitive code that just sends data to 
  and from the DOM between your app's internal state or the server
* Code example (2-dirty-checking-angular in-slide.)
    * Here we're looking at an Angular 
      [app](https://github.com/addyosmani/workspace/blob/master/2.%20Dirty-checking%20-%20Angular/phone-test.html) 
      that does dirty checking. It wants to know when this array of phone/todo 
      items has been changed.
* The web ecosystem should have more ability to innovate and evolve its own 
  declarative mechanisms, e.g.
    * Constraint-based model systems
    * Auto-persistence systems
    * We'll look at some examples of these systems soon.

**What does the world look like today?**

* **Container objects (Ember and Backbone)**
    * _Show code example_
    * [Code](https://github.com/addyosmani/workspace/tree/master/1.%20Set%20traps%20-%20Backbone%20and%20Ember)
    * Container objects are where a framework creates objects which on the 
      inside hold the data. They they have accessors to the data and they can 
      capture what you set or get and internally broadcast.
    * This works well. It's relatively performant. Good algorithmic behavior. 
      Expense of discovering what changed is proportional to the number of 
      things that changed. 
    * Another problem is now you're using this different kind of of object. 
      Generally speaking you have to convert from data you're getting from the 
      server to these objects so they're observable. 
    * This doesn't compose particularly well with existing JS code because most 
      code assumes it can operate on raw data. Not these specialized objects.
      
* **Dirty checking (Angular)**
    * _Show animation (3)_
    * _Show code example_
    * [Code](https://github.com/addyosmani/workspace/tree/master/2.%20Dirty-checking%20-%20Angular) 
      or [this](http://jsfiddle.net/hmyZB/) or 
      [this](http://plnkr.co/edit/1jhnFZ)
    * Basic idea is that anytime data could have changed, I go and check if it 
      did change. Benefit here is that I get to use raw JavaScript object data. 
      It composes well.
    * The downside is that it has bad algorithmic behavior and is potentially 
      very expensive. Expense is proportional to the total number of observed 
      objects. I may need to do a lot of dirty checking.
    * Also may need a way to trigger dirty-checking when data *might* have 
      changed. Lots of clever tricks frameworks use for this.
    * It's unclear if this is ever going to be perfect.

**Introducing Object.observe()**

* _Show animation (4)_
* So what we would like is to have the best of both worlds
* Raw data objects (regular JavaScript objects) if we choose to AND we don't 
  want to have to dirty check everything all the time
* Something with good algorithmic behavior
* Something that composes well

**Code walk-though**

* Some more examples
    * Persistence
        * You could imagine a persistence mechanism saving the state of data 
          directly to something like IndexedDB or localStorage. Or perhaps a 
          constraint based system which enforces relationships between data 
          properties or form values based on equations.
        * Based on relationships between data and how it changes.
    * Base case
        * Web platform features
        * This is using Polymer, a framework based on web components which is 
          similar to Angular or Ember. It has something a little like an MVC 
          mechanism. 
            * Idea is that it binds data to the DOM.
            * The DOMContentLoaded event is fired when the document has been 
              completely loaded and parsed…
                * <template> tag lets us declare in HTML inert DOM fragments
                * We've got some template binding which describes how <template> 
                  tag manages instances and Node.bind() which describes how DOM 
                  nodes are bound to data values
                * **repeat attribute**
                    * **Will create maintain exactly instance with {{ bindings 
                      }} for every element in the array collection, when it is 
                      provided.**
                    * Now onto the stuff we really care about.

> **We activate a template by setting data model on it. This causes any bind, 
> repeat or if attributes to begin acting.. you can see us doing this with 
> t.model.colors.**

            * It's a colors array and I can push a new color into the array. As 
              you can see its there but nothing interesting is happening to the 
              DOM. This is because I need to tell the DOM to go and do some 
              dirty checking. I do this with 
              Platform.performMicrotaskCheckpoint(). 
            * Then I tell it that, it discovers that my new color has showed up 
              but I need to go check all the potential places data could have 
              changed and it needs to be told.
            * Frameworks that use this approach can be clever about being able 
              to do this on any data that could have changed. But, it's not 
              perfect.
        * What we would like to see
            * An example of using a version of the same thing altered to work 
              with Object.observe()
            * Again, let's add our color to the colors array. You see it shows 
              right up. Basically any way I mutate this data the DOM is going to 
              follow along with me.
            * The idea is to pinpoint when data changes and react to what 
              changes when it changed
    * Going back to the idea of a persistence mechanism
        * We have another example
        * I create a class here which doesn't do much called Foo
        * I then ask persistDB to retrieve me my parallel array of Foo
        * So now I have this records array which represents a serialized version 
          of a collection of these Foos
        * I'm going to push onto it a new Foo
        * You'll see that this serialization mechanism just saved it 
          automatically to disk. 
        * I can go add data to that record. Do whatever I want. recs[0].berlin = 
          "berlin"
        * It's just going to save it out.
        * When I reload, there we go, my data is there in the record.
        * Basically when I change my record it is automatically serialized to 
          IndexedDB
    * Constraint mechanism
        * What we have here is a circle - a circle class. I've loaded my 
          constraint library. Three properties on my circle are radius, 
          circumference and area. They're related in a familiar way and you 
          probably recognize these equations.
        * Constraint mechanism basically just represents these using methods 
          which resolve data moving in one direction or other.
        * I can create the circle.  It's going to resolve the radius to assign 
          to circumference and area. If I update the area its going to resolve 
          to assigning to radius and circumference.
        * It's working. It happens to be one way of doing computed properties 
          where you want to have one property be depending on one property. 
          People care about this in MVC frameworks.
        * In this case all the properties are interdependent and its resolved 
          depending on which property you assign to. The idea is that the circle 
          is internally observing itself and setting dependent properties.
    * Another observation mechanism
        * We have an MVC app with both persistence and the constraint solver
        * I can create a new circle. It's saved for me right away. I can set a 
          radius and it assigns to circumference and area. I can update any of 
          these values and it updates the other values. Everything is 
          automatically saved. I can delete too.
        * One thing to notice is because of the timing of the way 
          Object.observe() works, you have a lot of power over the way things 
          work.
        * To illustrate this, I'm going to modify each of these circles and 
          increase the radius by a power of two and what you see here is the 
          resolver updated the internal properties of each of these circles 
          first. The persistence mechanism saw that two circles changed so it 
          serialized them to disk as a single transaction.
        * Because of the way that ordering of delivery happens, you have control 
          over when things need to happen and you can get some nice behaviour of 
          treating lots of changes as a transaction. Very consistent view of the 
          world and its pretty desirable.

 

**What can we observe?**

* _Show animation (5)_
* _Show code example_
* [Code](https://github.com/addyosmani/workspace/tree/master/3.%20Object.observe%20-%20observe%2C%20mutate%2C%20change) 
  Observe an object, mutate a property, see the change report
* You can find out when properties have been added. When they've been deleted.
* Basically, the _set_ of properties on an object ("new", "deleted", 
  "reconfigured") and it's prototype changing ("prototype")
* Or you can find out about changes to the value of data properties ("updated"). 
  Not accessors. I'll come back to this in a second.

**_Accept types._**

**_You need some dialog for this part._**

* _Show maybe animation 9_
* [Code](https://github.com/addyosmani/workspace/tree/master/5.%20Object.observe%20with%20acceptList)
* Object.observe(obj, callback, opt_acceptList)
* Observers can specify only those types of changes they wish to hear about. 
* If not provided, defaults to the "intrinsic" object change types ("new", 
  "updated", "deleted", "reconfigured", "prototype").
    * [Code](https://github.com/addyosmani/workspace/tree/master/6.%20Defaults%20to%20intrinsic%20object%20change%20types)

**Notifications**

* _Show animation (6)_
* Notification is similar to mutation observers
* It happens at the end of the micro-task
* In the browser context, this is almost always going to be at the end of the 
  current event handler
* The timing is nice because generally one unit of work is finished and now 
  observers get to do their work. Nice turn based processing model.
* object -> getNotifier(this).notify() -> changes
    * [Code](https://github.com/addyosmani/workspace/tree/master/4.%20notifier.notify%20-%20what%20can%20be%20observed)
    * Report when the value of _data_ properties changes ("updated")
    * _Anything else the object's implementation chooses to report 
      (notifier.notifyChange)_
* Experience on the web platform is that synchronous is the first thing you try 
  because its the easiest to wrap your head around.
    * Problem is it creates a fundamentally dangerous processing model
    * If you're writing code and say, update the property of an object, you 
      don't really want a situation having update the property of that object 
      could have invited some arbitary code to go do whatever it wanted. It's 
      not ideal to have your assumptions invalidated as you're running through 
      the middle of a function
    * If you're an observer, you ideally don't want to be called if someone is 
      in in the middle of something. You don't want to be asked to go to do work 
      on an inconsistent state of the world.
    * End up doing a lot more error checking. Trying to tolerate a lot more bad 
      situations and generally, its a hard model to work with.
    * Async is harder to deal with but its a better model at the end of the day.

**Single callback observer**

* Show animation (8)
* Callback delivery: a single _callback _can be used as an "observer" for lots 
  of objects
* The callback will be delivered the full set of changes to all objects it 
  observes at the "end of the microtask" (Note that the similarity to Mutation 
  Observers)

**Accessors Properties**

* We mentioned earlier that only the value changes are observable for data 
  properties. Not for computed properties or accessors
* The reason is that JavaScript doesn't really have the notion of changes in 
  value to accessors. An accessor is just a collection of functions. 
* If you assign to an accessor JavaScript just invokes the function there and 
  from its point of view nothing has changed. It just gave some code the 
  opportunity to run.

_(if referring to the specific example)_

* _Problem is semantically we can look at this assignment to the value we 
  assigned 5 to it. We ought to be able to know what happened here. _
* _Problem is that this is an unsolvable problem. Example demonstrates why._
* _Really no way for any system to know what this meant because this can be 
  arbitary code. It can do whatever it wants in this case._
* _It's updating the value every time it is accessed and so asking when does it 
  change doesn't have a huge amount of meaning._

**Synthetic change records**

* Show animation (7)
* The solution to this problem is synthetic change records
* Basically, if you want to have accessors or computed properties it is your 
  responsibility to notify when these values changed
* It's a little extra work but it is designed as a sort of first-class feature 
  of this mechanism and these notifications will be delivered with the rest of 
  the notifications from underlying data objects. From data properties.

* Observing accessors & computed properties can be solved with _notifier.notify_
    * Most observation systems want some form of observing derived values. There 
      are lots of ways to do this
    * Object.observe makes no judgement as to the "right" way.
    * Computed properties should be accessors which _notify_ when the internal 
      (private) state changes.
        * [Example of a simple object with an accessor 
          ](https://github.com/addyosmani/workspace/tree/master/8.%20notifier.notify%20-%20computed%20properties)_[notify](https://github.com/addyosmani/workspace/tree/master/8.%20notifier.notify%20-%20computed%20properties)_[ing.](https://github.com/addyosmani/workspace/tree/master/8.%20notifier.notify%20-%20computed%20properties)
    * Again, webdevs should expect libraries to help make notifying and various 
      approaches to computed properties easy (and reduce boilerplate)

_(referring to example)_

* _Idea here is that we have this circle and there's a radius property_
* _In this case the radius is an accessor and when its value changes its 
  actually going to notify for itself that the value changed._
* _This will be delivered with all other changes to this object or any other 
  object_
* _Essentially, if you're implementing an object you want to have synthetic or 
  computed properties on you have to pick a strategy for how this is going to 
  work._
* _Once you do, this will fit into your system as a whole_

**Large-scale changes**

* Objects may wish to describe larger semantic changes which will affect lots of 
  properties in a more compact way (instead of broadcasting tons of property 
  changes)
* notifier.performChange + notifier.notify serve this purpose
    * [Code](https://github.com/addyosmani/workspace/tree/master/8.%20notifier.notify%20-%20computed%20properties)
* notifier.performChange("big-change", function() { … } ) + 
  notifier.notify("big-change") example
    * [Code](https://github.com/addyosmani/workspace/tree/master/9.%20notifier.performChange%20-%20largescale%20changes)
    * Everything inside of the "perform function" is considered to be the work 
      of "big-change".
    * Observers which _accept_ "big-change" will only receive the "big-change" 
      record
    * Observers which do not will receive the underlying changes resulting from 
      the work which "perform function" did.
* This is how Array.observe works.
    * [Code](https://github.com/addyosmani/workspace/tree/master/10.%20Array.observe()%20-%20splice%20change%20record)
    * Array.observe treats large scale changes to itself (e.g. splice, unshift 
      or anything which implicitly changes its length) as a "splice" change 
      record. Internally, it does notifier.performChange("splice", ….)

 
**Audience for Object.observe()**

* Show animation 10
* It's unlikely App developers will want to use Object.observe directly, as it 
  simply provides a record of all changes that occur. 
* More likely, JS libraries (like ObserveJS) will evolve which input what 
  Object.observe reports and provide a view of what changed with more useful 
  invariants 
* (like "What I'm reporting is the net-effect of what has changed from your 
  point of view" -- filtering out noops and coalescing changes).

* **Observing DOM nodes: the uncharted frontier.**
    * Object.observe says nothing about the behavior of "host" objects
    * WebIDL specifies that DOM properties are _accessors_ and thus not directly 
      observable _unless they choose to notify_.
    * Initially, no DOM objects will _notify_ on changes to IDL properties (e.g. 
      offsetWidth, etc...)
    * Hopefully, input elements will soon be specified to notify when their 
      "input properties" (e.g. input.value) change (this is critical for 
      data-binding systems which must currently rely on events).
    * It's highly unlikely that a broad set of DOM object properties will be 
      observable because it would impose too high a performance cost.

**Lastly, Rafael implemented a utility library now called ObserveJS**

* Consider it a polyfill for Object.observe()
* It offers an aggregate view of the world that sums up changes and delivers a 
  report of what has changed
* Two things which are powerful are
    * 1) You can observe paths so you can say I want to observe "foo.bar.baz" 
      from a given object and they'll tell you when the value at that path 
      changed. if the path is unreachable, it considers the value undefined
    * 2) It will tell you about array splices. Array splices are basically the 
      minimal set of splice operations you will have to perform on an array in 
      order to transform the old version of the array into the new version of 
      the array. This is sort of a transform or a different view of the array. 
      It's the minimum amount of work you need to do to move from the old state 
      to the new state.
    * Generally what you want to know.

**Conclusions**

Object.observe() is currently available for testing behind the experimental JS 
flag in Chrome Canary. Please try it and ObserveJS out and give us feedback. We 
hope you find them useful.

 