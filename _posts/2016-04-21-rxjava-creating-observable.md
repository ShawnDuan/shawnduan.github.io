---
layout: post
title:  "RxJava: Creating Observable"
date:   2016-04-21
comments: true
categories: [Android, RxJava]
---

## Ways to create `Observable`

### `Observable.create()`

Create an Observable from scratch by means of a function defining the rule/schedule (`Observable.OnSubscribe`) of event triggering. `call()` will be called once this `Observable` is subscribed.

```java
Observable observable = Observable.create(new Observable.OnSubscribe<String>() {
    @Override
    public void call(Subscriber<? super String> subscriber) {
        subscriber.onNext("Hello");
        subscriber.onNext("Hi");
        subscriber.onNext("Aloha");
        subscriber.onCompleted();
    }
});
```

### `Observable.just()`

Convert an object or several objects into an Observable that emits that object or those objects (one to nice objects are allowed.) The returned `Observable` is gonna emit these objects in the same order as they are given in the parameter list, once it is subscribed.

```java
Observable observable = Observable.just("Hello", "Hi", "Aloha");
```

### `Observable.from()`

Convert an Iterable, a Future, or an Array into an Observable that emits the containing objects in order.

```java
String[] words = {"Hello", "Hi", "Aloha"};
Observable observable = Observable.from(words);
```

> **Notice: ** These three examples creating `Observables` are equivalent.

------

### `Observable.defer()`

The Defer operator waits until an observer subscribes to it, and then it generates an fresh Observable on each subscription, typically with an Observable factory function.

Key words: defer until subscribing; fresh observable; Observable factory function.

```java
Observable.defer(new Func0<Observable<Provider.AccountInfo>>() {	// observable factory function
            @Override
            public Observable<Provider.AccountInfo> call() {
                try {
                    return Observable.just(getAccountInfo(lyveCloudProvider));
                } catch (Provider.ProviderException e) {
                    return null;
                }
            }
        });
```

> **Notice: ** Nest `Observable.just()/from()` in `Observable.defer()` is a way to generate an observable, and in the meantime, make sure this observable is not generated too early before it's actually been subscribed.
