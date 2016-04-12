# Why RxJava Instead of EventBus


#### One pipeline vs Multiple pipelines

EventBus or Otto has "**one pipeline**", then you create multiple events going down, e.g. `AnswerAvailableEvent`, within this one pipeline.
In RxJava you don't need all these `XxxxEvent`, you can create **multiple pipelines**

#### Reactive models

Modular reactive code. Have multiple models doing the data or network logic for you while having clean Activities and Fragments. (https://medium.com/@diolor/rxjava-as-event-bus-the-right-way-10a36bdd49ba#.24pwqj5bi)

* Modularity and easy testing

#### Nested Events

This is a common pitfall for event bus, which would lead to complex and hard-to-debug code.



## Observer pattern

The simplest example is `Button` -> `OnClick(View)` -> `OnClickListener`


