---
layout: post
title:  "ReactiveX / RxJava / RxAndroid"
date:   2016-04-13
categories: [Android, RxJava]
---

* TOC
{:toc}


## What is ReactiveX / RxJava / RxAndroid

* **ReactiveX** is an API that focuses on **asynchronous composition** and **manipulation of observable streams of data or events** by using a combination of the Observer pattern, Iterator pattern, and features of Functional Programming.

![ReactiveX](/assets/20160413/reactivex-flow.png)

* **RxJava** is part of ReactiveX, a group of open source libraries. They have libraries in many different languages, including JavaScript, Groovy, Ruby, Java, C#, and the list goes on. 

* **RxAndroid** is a lightweight extension to RxJava that providers a `Scheduler` for Android’s Main Thread, as well as the ability to create a `Scheduler` that runs on any given Android Handler class. 

> **Functional Reactive Programming** is all about streams of data. A stream produces data at different points in time. An observer is notified whenever data in that stream and does something with it. It's a little different than the Observer pattern, since there's a focus on composition. You can map, filter, and reduce streams—that's where the "functional" part comes in.

## What can RxJava do?

* Replace Event Bus libraries
* Replace AsyncTask, and do it better.
* And more...

### RxJava vs. Event Bus

From what I can tell, RxJava implemented event bus doesn't show many advantages comparing to event bus libraries like Otto. So if you just want an event bus, maybe you should hang on with Otto. But if you choose to jump on board of RxJava, it's an easy decision to make getting rid of any event bus library to reduce the package size, becaue you can easily implement an event bus with RxJava.

* **Otto**

```java
@Subscribe
public void answerAvailable(ItemSelectedEvent event) {
    Log.d(TAG, "call event: " + event);
    doSomething();
}
        
// post event
BusMaster.getBus().post(new ItemSelectedEvent(position, pairs));

// unregister
BusMaster.getBus().unregister(this);
```
* **RxBus**

```java
// subscribe event
mItemClickSubscription = RxBus.getInstance().register(ItemSelectedEvent.class,
                new Action1<ItemSelectedEvent>() {
                    @Override
                    public void call(ItemSelectedEvent event) {
                        Log.d(TAG, "call event: " + event);
                        doSomething();
                    }
                });
                
// post event
RxBus.getInstance().post(new ItemSelectedEvent(position, pairs));
                
// unsubscribe() when not needed.
mItemClickSubscription.unsubscribe();
```

### RxJava vs. AsyncTask

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
    })
    .subscribeOn(Schedulers.io())					// thread where `subscribe()` happened.
    .observeOn(AndroidSchedulers.mainThread());		// thread where `Subscriber` runs in.

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

#### Error handling

* There's no easy way to handle error messages in the UI for errors that happen in the background.

* `onError()` is called if an Exception is thrown at any time. And the operators don't have to handle the Exception. Leave it up to the `Subscriber` to determine how to handle issues with any part of the `Observable` chain because Exceptions skip ahead to `onError()`.

#### Preventing Leaks

* `AsyncTasks` leak references to their enclosing Activity/Fragment if proper care is not taken.

* When you call `Observable.subscribe()` a `Subscription` object is returned. To prevent a possible memory leak, in your Activity/Fragment’s onDestroy, `unsubscribe()` the `Observable` if it has not been unsubscribed. Unsubscribing will stop notifications of items to your `Subscriber` and will allow the garbage collection of all related objects, preventing any RxJava related memory leaks.


#### Activity/Fragment lifecycle

*  If you back out of the Activity/Fragment or rotate the device while this AsyncTask is running, you will get a NullPointerException and a resulting crash when trying to access the Activity and/or the views since they are now gone and null.

* Using a library called `RxLifeCycle` will solve the challenge.

```java
webService.doSomething(someData)
          .compose(bindToLifecycle())
          .subscribe(result -> resultText.setText("It worked!"), 
                      e -> handleError(e));
```

> bindToLifecycle() will hook your Observable chain into the Fragment lifecycle callback events and auto-unsubscribe causing execution to stop so that there’s no risk of running running your Fragment subscription code in an invalid state.  This also prevents leaking of Fragments and Activities since they are dereferenced during the unsubscription. 

#### Caching on rotation

......


### What're the other benefits using RxJava

* #### Composing multiple Async tasks

	RxJava also lets you compose logically separated tasks, using the operators it provides: `map()`, `flatMap()`, etc.


* #### Supported by Retrofit, Realm, etc.

* #### More...

	Simpler, readable code, etc

## RxJava and RetroLambda

Lambdas is cool -- it helps you reduce code size and more. But you need Java 8 to support it. If you are using Java 5/6/7, you'd need RetroLambda as the backport of Java 8's lambda expressions. There are both risks and awards -- it's your own decision. 

> While not really correlated to RxAndroid, lambdas are constructs that get along very well with functional programming: they reduce the amount of code we write and help us in further avoiding the callback hell, one of the reasons why we started using RxAndroid in the first place. Moreover, they are in fact anonymous inner classes, optimized with the use of Singleton in case of stateless functions, thus avoiding multiple allocations when unnecessary: this alone make them valuable and powerful allies!


## Concepts

### Observer pattern

The simplest observer pattern example is `Button` -> `OnClick(View)` -> `OnClickListener`

RxJava's Observer pattern: `Observable` -> `subscribe` -> `Observer`, and "event". An `Observable` emits events; a `Subscriber` consumes those events. An Observable may emit any number of items (including zero items), then it terminates either by successfully completing, or due to an error. For each Subscriber it has, an Observable calls Subscriber.onNext() any number of times, followed by either Subscriber.onComplete() or Subscriber.onError().

The one key way RxJava differes from [standard observer pattern](http://en.wikipedia.org/wiki/Observer_pattern) is: 

> Observables often don't start emitting items until someone explicitly subscribes to them2. In other words: if no one is there to listen, the tree won't fall in the forest.


### Subjects

A Subject is an Object in RxJava that has the properties of both an Observable and an Observer/Subscriber. It can both subscribe to Observables to receive data, as well as emit data to Subscribers who subscribe to it. It can also emit data by directly calling the onNext method. In this manner, it can be used as a middle-man who receives data, manipulates it in some way, then emits it to its own subscribers.

There are two type parameters for a Subject: T and R. T is type of data it will emit (from the Observer interface), while R is the type of data it will receive (From the Observable class). Typically, these will be the same type unless data manipulation is taking place.

There are a few types of specialized Subjects that are worth noting for this implementation. The most basic Subject implementation is PublishSubject. PublishSubject will simply emit data to each subscriber each time its onNext method is called (either manually or from an Observable).

A SerializedSubject can be used to wrap another Subject to make calls to its methods thread-safe.

### Operators

We don't want to modify `Observable` because (1) We may not have the control over the `Observable` -- it maybe came from someone else's library; (2) This `Observable` could be used in multiple places, but different modifications are needed.

We don't want to modify `Subscriber` neither, because (1) We want `Subscriber` to be as lightweight as possible since they might be run on UI thread; (2) `Subscribers` are meant to be things that *reacts*, not *mutates*

Thus, RxJava provides us the way to transform item between `Observable` and `Subscriber` -- `Operators`. Operators can be used in between the source Observable and the ultimate Subscriber to manipulate emitted items.

### More Concepts...

...

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

* **with lambdas**

```java
Observable.just("Hello, world!")
    .map(s -> s + " -Dan")
    .subscribe(s -> System.out.println(s));
```

### Load and display images

Suppose we have an `imageCollectorView`, which can show multiple images, and we can use addImage(Bitmap) to add any amount of images to it. Given `File[] folders`, and we want to load and show all png files in imageCollectorView.

The idea is we want to load images on background thread, and display it on UI thread.

* **Old way**

```java
new Thread() {
    @Override
    public void run() {
        super.run();
        for (File folder : folders) {
            File[] files = folder.listFiles();
            for (File file : files) {
                if (file.getName().endsWith(".png")) {
                    final Bitmap bitmap = getBitmapFromFile(file);
                    getActivity().runOnUiThread(new Runnable() {
                        @Override
                        public void run() {
                            imageCollectorView.addImage(bitmap);
                        }
                    });
                }
            }
        }
    }
}.start();
```

* **RxJava**

```java
Observable.from(folders)
    .flatMap(new Func1<File, Observable<File>>() {
        @Override
        public Observable<File> call(File file) {
            return Observable.from(file.listFiles());
        }
    })
    .filter(new Func1<File, Boolean>() {
        @Override
        public Boolean call(File file) {
            return file.getName().endsWith(".png");
        }
    })
    .map(new Func1<File, Bitmap>() {
        @Override
        public Bitmap call(File file) {
            return getBitmapFromFile(file);
        }
    })
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Action1<Bitmap>() {
        @Override
        public void call(Bitmap bitmap) {
            imageCollectorView.addImage(bitmap);
        }
    });
```

* **With Lambda**

```java
Observable.from(folders)
    .flatMap((Func1) (folder) -> { Observable.from(file.listFiles()) })
    .filter((Func1) (file) -> { file.getName().endsWith(".png") })
    .map((Func1) (file) -> { getBitmapFromFile(file) })
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe((Action1) (bitmap) -> { imageCollectorView.addImage(bitmap) });
```
