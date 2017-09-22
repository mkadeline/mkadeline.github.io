---
layout: post
title: Dagger 2 Android - Scoping
date: 2017-09-22
category: programming
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
Our MainActivity will not need access to the Athlete or Records objects and so the TimerComponent can inject the Timer directly. However, other activities require the WorkoutModule and so that will need to be injerited further down the line, in our WorkoutComponent, which now also has access to the Timer:
```java
@Workout
@Subcomponent(modules = {WorkoutModule.class})
public interface WorkoutComponent {

    ActiveSessionActivity inject(ActiveSessionActivity activeSessionActivity);
    SummaryActivity inject(SummaryActivity summaryActivity);

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
```
Finally, the MyApp that extends Application:
```java
public class MyApp extends Application {

    private AppComponent appComponent;
    private TimerComponent timerComponent;
    private WorkoutComponent workoutComponent;

    public static MyApp get(Context context) {
        return (MyApp) context.getApplicationContext();
    }

    @Override
    public void onCreate() {
        super.onCreate();
        createAppComponent();
    }

    public void createAppComponent() {
        appComponent = DaggerAppComponent.builder()
                .appModule(new AppModule(this))
                .build();
    }

    public TimerComponent createTimerComponent() {
        // This is where the component is instantiated
        // So it will only be instantiated once as Application is only instantiated once
        timerComponent = appComponent.plus(new TimerModule());
        return timerComponent;
    }

    public TimerComponent getTimerComponent() {
        return timerComponent;
    }

    public void releaseWorkoutComponent() { workoutComponent = null; }

    public WorkoutComponent createWorkoutComponent(String name) {
        workoutComponent = timerComponent.plus(new WorkoutModule(name));
        return workoutComponent;
    }

    public WorkoutComponent getWorkoutComponent() {
        return workoutComponent;
    }
}
```
For ease of use we will create the WorkoutComponent in MyApp, however it requires the TimerComponent for its construction. We have therefore designed our WorkoutComponent in such a way as it cannot exist without the TimerComponent. If for example, we set AthleteActivity as the launch point, and attempt to create a WorkoutComponent without a TimerComponent, we get the expected NullPointerException as the JVM complains we're attempting to access the plus() method on a null TimerComponent.
This is obviously a design flaw in our particular implementation but it shows the degree of control Dagger 2 still requires.

Another interesting point is whether or not we'd need to control the lifecycle of the Component. By controlling the scope we're allowing for local singletons, for as long as the component lives. In some examples I've seen, these temporary components are explicitly dereferenced by
```java
XXXComponent = null;
```
however given we ensure the source of our WorkoutComponent is MyApp, by calling getWorkoutComponent() we will always get the latest workoutComponent instance that was returned by createWorkoutComponent(). The old WorkoutComponent is dereferenced and the scope, its objects and Modules die with it.
