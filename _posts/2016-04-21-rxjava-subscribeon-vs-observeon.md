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

`SubscribeOn` operator designates which thread the Observable will begin operating on, no matter at what point in the chain of operators that operator is called. `ObserveOn` on the other hand, affects the thread that the Observable will use below where that operator appears. 

For this reason, you may call `ObserveOn` multiple times at various points during the chain of Observable operators in order to change on which threads certain of those operators operate. Meanwhile, only one `SubscribeOn` should called to designate the starting thread.

* `subscribeOn()`: where events are created.
* `observeOn()`: where events are manipulated and consumed.

![schedulers](/assets/images/subscribeon-vs-observeon.png)