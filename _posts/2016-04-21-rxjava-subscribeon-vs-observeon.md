---
layout: post
title:  "RxJava: subscribeOn vs observeOn"
date:   2016-04-21
comments: true
categories: [Android, RxJava]
---

## subscribeOn() vs observeOn()

One common misconception about RxJava is that it’s asynchronous, but everything is synchronous by default, actually. When you build a stream, you are just constructing it to the point where we actually subscribe to it. When you subscribe, you build it all together and then actually execute it. **Until you call subscribe, you’re only constructing a stream.**

By default, an Observable and the chain of operators that you apply to it will do its work, and will notify its observers, on the same thread **on which its Subscribe method is called**. 

`subscribeOn` operator designates which thread the Observable will **begin** operating on, no matter at what point in the chain of operators that operator is called. `observeOn` on the other hand, affects the thread that the Observable will use **below** where that operator appears. 

For this reason, you may call `observeOn` multiple times at various points during the chain of Observable operators in order to change on which threads certain of those operators operate. Meanwhile, only one `subscribeOn` should called to designate the starting thread.

* `subscribeOn()`: where events are created.
* `observeOn()`: where events are manipulated and consumed.

> **Special Case**

> In one situation, multiple calling `subscribeOn` would be meanful, that is you want to designate a specific thread for `doOnSubscribe`. The first `subscribeOn` below `doOnSubscribe` determines the thread for `doOnSubscribe`. In this case, multiple `subscribeOn` exist in the stream -- the very first one determines which thread the Observable will **begin** operating on.

![schedulers](/assets/images/subscribeon-vs-observeon.png)