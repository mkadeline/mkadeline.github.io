---
layout: post
title: Dagger 2 Android - Scoping
---
In my last article I got around a basic use of Dagger 2 to implement a true singleton. Now I'd like to expand that and take advantage of Dagger 2's inheritance property and Scoping.

Again the premise is an interval Timer App with a global Timer instantiated once and used everywhere. However this time, we'll introduce an Athlete that, on every session will do some pushups or situps, which are added to their own PushupRecord and SitupRecord objects. The objects will only live as long as the session. Each new session will bring about a new Athlete and set of records however we need to ensure the objects are 'singletons' during the workout and on any activities they are required before destruction.

Starting from the bottom, we define a new Module to provide the dependencies:

```java
@Module
public class WorkoutModule {

    private final String name;

    public WorkoutModule(String name) {
        this.name = name;
    }

    @Provides @Workout
    PushupRecord providePushuprecord() {
        PushupRecord pRecord = new PushupRecord();
        return pRecord;
    }

    @Provides @Workout
    SitupRecord provideSituprecord() {
        SitupRecord sRecord = new SitupRecord();
        return sRecord;
    }

    @Provides @Workout
    Athlete provideAthlete(PushupRecord pushupRecord, SitupRecord situpRecord) {
        Athlete athlete = new Athlete(name, pushupRecord, situpRecord );
        return athlete;
    }
}
```

The Athlete object will require a name, which must be provided to the module so it may provide it to the Athlete. This is how we inject primitive etc. dependencies into our injected Objects.

Now we need to set up the component stucture to provide the dependencies.
Firstly, we introduce an AppComponent, to allow for easier inheritence if we were to add additional features:
```java
@Component(
        modules = { AppModule.class }
)
public interface AppComponent {

    TimerComponent plus(TimerModule module);
    WorkoutComponent plus(WorkoutModule module);

}
```
This is our 'Super Component', one which the others will subcomponent. Becuase we are implementing subcomponents, we do not need to explicitly make anything available as they will be automatically available to subcomponents.

Now our new TimerComponent:
```java
@Singleton
@Subcomponent(modules={AppModule.class, TimerModule.class})
public interface TimerComponent {

    void inject(MainActivity mainActivity);
    WorkoutComponent plus(WorkoutModule module);

}
```
Our MainActivity will not need access to the Athlete or Records objects and so the TimerComponent can inject the Timer directly. However, other activities require the WorkoutModule and so that will need to be injerited further down the line, in our WorkoutComponent:
```java
@Workout
@Subcomponent(modules = {WorkoutModule.class})
public interface WorkoutComponent {

    ActiveSessionActivity inject(ActiveSessionActivity activeSessionActivity);

}
```
The WorkoutComponent can inject dependencies into ActiveSessionActivity and will operate within the @Workout custom scope I have defined in Workout.java:
```java
@Scope
public @interface Workout {
}
``` 
By defining our custom @Workout scope we ensure all Athletes, PushupRecords and SitupRecords are singletons while that scope is live, which means we'll get a new Athlete, and Record objects when we instantiate a new WorkoutComponent.
So we now are able to completely control the creation and destruction of our objects through Dagger, without boilerplate and importantly, without having to consider which Athlete we'll be working on. We do however need to control the instantiation of WorkoutComponent, which will have a clear entry and exit point in the lifecycle of the app.

In our AthleteActivity, we get the name of the Athlete and create the WorkoutComponent:
```java
public class AthleteActivity extends Activity {

    // Activity methods and Propertoes

    public void onEnterButtonClicked(View view) {
        EditText nameBox = findViewById(R.id.athleteName);
        String name = nameBox.getText().toString();

        //Build the workout component and get dependencies ready
        MyApp.get(this).createWorkoutComponent(name);

        Intent intent = new Intent(this, CountdownActivity.class);
        intent.putExtra("name", name);
        startActivity(intent);
    }
}
