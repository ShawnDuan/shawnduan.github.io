---
layout: post
title:  "RxJava"
date:   2016-04-12
categories: [Android, RxJava]
---

## What is ReactiveX / RxJava / RxAndroid

ReactiveX is an API that focuses on asynchronous composition and manipulation of observable streams of data or events by using a combination of the Observer pattern, Iterator pattern, and features of Functional Programming.

RxJava is part of ReactiveX, a group of open source libraries. They have libraries in many different languages, including JavaScript, Groovy, Ruby, Java, C#, and the list goes on. 

RxAndroid is a lightweight extension to RxJava that providers a Scheduler for Android’s Main Thread, as well as the ability to create a Scheduler that runs on any given Android Handler class. 

> Functional Reactive Programming is all about streams of data. A stream produces data at different points in time. An observer is notified whenever data in that stream and does something with it. It's a little different than the Observer pattern, since there's a focus on composition. You can map, filter, and reduce streams—that's where the "functional" part comes in.

> The core idea, though, is that everything is a stream.


## RxJava vs AsyncTask

### Error handling

* There's no easy way to handle error messages in the UI for errors that happen in the background.

* `onError()` is called if an Exception is thrown at any time. And the operators don't have to handle the Exception. Leave it up to the `Subscriber` to determine how to handle issues with any part of the `Observable` chain because Exceptions skip ahead to `onError()`.

### Preventing Leaks

* `AsyncTasks` leak references to their enclosing Activity/Fragment if proper care is not taken.

* When you call Observable.subscribe() a Subscription object is returned. To prevent a possible memory leak, in your Activity/Fragment’s onDestroy, check Subscription.isUnsubscribed() for if your Subscription is unsubscribed, and if not call Subscription.unsubscribe(). Unsubscribing will stop notifications of items to your Subscriber and will allow the garbage collection of all related objects, preventing any RxJava related memory leaks.


### Activity/Fragment lifecycle

*  If you back out of the Activity/Fragment or rotate the device while this AsyncTask is running, you will get a NullPointerException and a resulting crash when trying to access the Activity and/or the views since they are now gone and null.

* Using a library called `RxLifeCycle` will solve the challenge.

```java
webService.doSomething(someData)
          .compose(bindToLifecycle())
          .subscribe(result -> resultText.setText("It worked!"), 
                      e -> handleError(e));
```

> bindToLifecycle() will hook your Observable chain into the Fragment lifecycle callback events and auto-unsubscribe causing execution to stop so that there’s no risk of running running your Fragment subscription code in an invalid state.  This also prevents leaking of Fragments and Activities since they are dereferenced during the unsubscription. 

### Caching on rotation

### Composing multiple web service calls

RxJava also lets you compose logically separated tasks


## RxJava and Event Bus

You can easily implement an event bus using RxJava.

### One pipeline vs Multiple pipelines

EventBus or Otto has "**one pipeline**", then you create multiple events going down, e.g. `AnswerAvailableEvent`, within this one pipeline.
In RxJava you don't need all these `XxxxEvent`, you can create **multiple pipelines**

### Reactive models

Modular reactive code. Have multiple models doing the data or network logic for you while having clean Activities and Fragments. (https://medium.com/@diolor/rxjava-as-event-bus-the-right-way-10a36bdd49ba#.24pwqj5bi)

* Modularity and easy testing

### Nested Events

This is a common pitfall for event bus, which would lead to complex and hard-to-debug code.


## What's the other benefits using RxJava

blabla


## RxJava and RetroLambda

While not really correlated to RxAndroid, lambdas are constructs that get along very well with functional programming: they reduce the amount of code we write and help us in further avoiding the callback hell, one of the reasons why we started using RxAndroid in the first place. Moreover, as aforementioned, they are in fact anonymous inner classes, optimized with the use of Singleton in case of stateless functions, thus avoiding multiple allocations when unnecessary: this alone make them valuable and powerful allies!




## Concepts


### Observer pattern

The simplest example is `Button` -> `OnClick(View)` -> `OnClickListener`


### Subjects

A Subject is an Object in RxJava that has the properties of both an Observable and an Observer/Subscriber. It can both subscribe to Observables to receive data, as well as emit data to Subscribers who subscribe to it. It can also emit data by directly calling the onNext method. In this manner, it can be used as a middle-man who receives data, manipulates it in some way, then emits it to its own subscribers.

There are two type parameters for a Subject: T and R. T is type of data it will emit (from the Observer interface), while R is the type of data it will receive (From the Observable class). Typically, these will be the same type unless data manipulation is taking place.

There are a few types of specialized Subjects that are worth noting for this implementation. The most basic Subject implementation is PublishSubject. PublishSubject will simply emit data to each subscriber each time its onNext method is called (either manually or from an Observable).

A SerializedSubject can be used to wrap another Subject to make calls to its methods thread-safe.

### Operators

We don't want to modify `Observable` because (1) We may not have the control over the `Observable` -- it maybe came from someone else's library; (2) This `Observable` could be used in multiple places, but different modifications are needed.

We don't want to modify `Subscriber` neither, because (1) We want `Subscriber` to be as lightweight as possible since they might be run on UI thread; (2) `Subscribers` are meant to be things that *reacts*, not *mutates*

Thus, RxJava provides us the way to transform item between `Observable` and `Subscriber` -- `Operators`. Operators can be used in between the source Observable and the ultimate Subscriber to manipulate emitted items.



## Sample Code

### Hello World

* **Basic Code**

```java
Observable<String> myObservable = Observable.create(
    new Observable.OnSubscribe<String>() {
        @Override
        public void call(Subscriber<? super String> sub) {
            sub.onNext("Hello, world!");
            sub.onCompleted();
        }
    }
);

Subscriber<String> mySubscriber = new Subscriber<String>() {
    @Override
    public void onNext(String s) { System.out.println(s); }

    @Override
    public void onCompleted() { }

    @Override
    public void onError(Throwable e) { }
};

myObservable.subscribe(mySubscriber);
// Outputs "Hello, world!"
```

* **Simpler Code**

```java
Observable<String> myObservable = Observable.just("Hello, world!");

Action1<String> onNextAction = new Action1<String>() {
    @Override
    public void call(String s) {
        System.out.println(s);
    }
};

// myObservable.subscribe(onNextAction, onErrorAction, onCompleteAction);
myObservable.subscribe(onNextAction);
// Outputs "Hello, world!"
```

* **Chain the method calls together**

```java
Observable.just("Hello, world!")
    .subscribe(new Action1<String>() {
        @Override
        public void call(String s) {
              System.out.println(s);
        }
    });
```

* **Chained code with Java 8 lambdas**

```java
Observable.just("Hello, world!")
    .subscribe(s -> System.out.println(s));
```

* **Introduce `map()` operator**

```java
Observable.just("Hello, world!")
    .map(new Func1<String, String>() {
        @Override
        public String call(String s) {
            return s + " -Dan";
        }
    })
    .subscribe(s -> System.out.println(s));
```

with lambdas

```java
Observable.just("Hello, world!")
    .map(s -> s + " -Dan")
    .subscribe(s -> System.out.println(s));
```

### AsyncTask vs RxJava

* **AsyncTask**

```java
private class MyAsyncTasks extends AsyncTask<Void, Void, String> {
    @Override
    protected void onPostExecute(String s) {
        Snackbar.make(mRootView, s, Snackbar.LENGTH_LONG).show();
        mAsyncButton.setEnabled(true);
    }

    @Override
    protected String doInBackground(Void... params) {
        return longRunningOperation();
    }
}

mAsyncButton.setOnClickListener(v -> {
    mAsyncButton.setEnabled(false);
    new MyAsyncTasks().execute();
});
```

* **RxJava**

```java
Observable<String> observable = Observable.create(new Observable.OnSubscribe<String>() {
    @Override
    public void call(Subscriber<? super String> subscriber) {
        subscriber.onNext(longRunningOperation());
        subscriber.onCompleted();
    }
}).subscribeOn(Schedulers.io()).observeOn(AndroidSchedulers.mainThread());

mRxButton.setOnClickListener(v -> {
    mRxButton.setEnabled(false);
    observable.subscribe(new Subscriber<String>() {
        @Override
        public void onCompleted() {
            mRxButton.setEnabled(true);
        }

        @Override
        public void onError(Throwable e) {

        }

        @Override
        public void onNext(String s) {
            Snackbar.make(mRootView, s, Snackbar.LENGTH_LONG).show();
        }
    });
});
```



