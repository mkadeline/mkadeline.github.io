---
layout: post
title: Dagger 2 Android - Scoping
date: 2017-09-22
categories: programming android
imageid: android.png
---

In my last article I got around a basic use of Dagger 2 to implement a true singleton. Now I'd like to expand that and take advantage of Dagger 2's inheritance property and Scoping.

Again the premise is an interval Timer App with a global Timer instantiated once and used everywhere. However this time, we'll introduce an Athlete that, on every session will do some pushups or situps, which are added to their own PushupRecord and SitupRecord objects. The objects will only live as long as the session. Each new session will bring about a new Athlete and set of records however we need to ensure the objects are 'singletons' during the workout and on any activities they are required before destruction.

Starting from the bottom, we define a new Module to provide the dependencies:

<script src="https://gist.github.com/mkadeline/4a4d3a9cc383d8602e123f6fd5cd4232.js"></script>

The Athlete object will require a name, which must be provided to the module so it may provide it to the Athlete. This is how we inject primitive etc. dependencies into our injected Objects.

Now we need to set up the component stucture to provide the dependencies.
Firstly, we introduce an AppComponent, to allow for easier inheritence if we were to add additional features:

<script src="https://gist.github.com/mkadeline/64c711e67f08849be6927c0e0886dfa7.js"></script>

This is our 'Super Component', one which the others will subcomponent. Becuase we are implementing subcomponents, we do not need to explicitly make anything available as they will be automatically available to subcomponents.

Now our new TimerComponent:

<script src="https://gist.github.com/mkadeline/6dc4a7730f5c45be4e66fdf1bd0e3146.js"></script>

Our MainActivity will not need access to the Athlete or Records objects and so the TimerComponent can inject the Timer directly. However, other activities require the WorkoutModule and so that will need to be injerited further down the line, in our WorkoutComponent, which now also has access to the Timer:

<script src="https://gist.github.com/mkadeline/4e115d533d34356c91df80026f5521d2.js"></script>

The WorkoutComponent can inject dependencies into ActiveSessionActivity and will operate within the @Workout custom scope I have defined in Workout.java:

<script src="https://gist.github.com/mkadeline/c67c6ce80b4c5cb52c200e409fa390bf.js"></script>

By defining our custom @Workout scope we ensure all Athletes, PushupRecords and SitupRecords are singletons while that scope is live, which means we'll get a new Athlete, and Record objects when we instantiate a new WorkoutComponent.
So we now are able to completely control the creation and destruction of our objects through Dagger, without boilerplate and importantly, without having to consider which Athlete we'll be working on. We do however need to control the instantiation of WorkoutComponent, which will have a clear entry and exit point in the lifecycle of the app.

In our AthleteActivity, we get the name of the Athlete and create the WorkoutComponent:

<script src="https://gist.github.com/mkadeline/0473ced9a961e4cdea8df7d4c626dfdb.js"></script>

Finally, the MyApp that extends Application:

<script src="https://gist.github.com/mkadeline/a91fd4e55da5d50d52ce7e367766fab3.js"></script>

For ease of use we will create the WorkoutComponent in MyApp, however it requires the TimerComponent for its construction. We have therefore designed our WorkoutComponent in such a way as it cannot exist without the TimerComponent. If for example, we set AthleteActivity as the launch point, and attempt to create a WorkoutComponent without a TimerComponent, we get the expected NullPointerException as the JVM complains we're attempting to access the plus() method on a null TimerComponent.
This is obviously a design flaw in our particular implementation but it shows the degree of control Dagger 2 still requires.

Another interesting point is whether or not we'd need to control the lifecycle of the Component. By controlling the scope we're allowing for local singletons, for as long as the component lives. In some examples I've seen, these temporary components are explicitly dereferenced by

```java
XXXComponent = null;
```

however given we ensure the source of our WorkoutComponent is a MyApp field, by calling getWorkoutComponent() we will always get the latest workoutComponent instance that was returned by createWorkoutComponent(). The old WorkoutComponent is dereferenced and the scope, its objects and Modules die with it.
