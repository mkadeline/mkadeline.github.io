---
layout: post
title: "Dagger 2 Android Dependency Injection"
date: 2017-09-20
category: programming
---

n exploring object creation, at some point in modern programming, we need to turn to dependency injection. The separation of concerns principle and ensuring our objects are modular means at times we can have a ridiculous number of objects, each doing one thing. While that makes testing, maintenance and upgrading much easier, it can make following your object's dependencies more difficult.

At times however, including when exploring dagger 2 with its three pronged approach to dependency injection, I have questioned whether the additional code and structure is actually easier than simply instantiating objects or using boilerplate dependency injection. What follows is an attempt to understand how Dagger 2 would operate in a smaller project and how that then scales in a bigger project.

I'd love some critique on the 'facts' below and on my thought process.

My first stop in investigating this was considering how 'expensive' object creation and destruction is. When I first considered dependency injection I imagined some new method of creating objects that was inherently more efficient than the 'new ObjectX()' method I'd been taught. Looking at the final injectors showed me that's not the case but I headed down the object creation rabbithole anyway.
Is it expensive to create Java objects, in either time or memory resources? Memory resources would depend on the object itself, so to consider time, is it a slow task to create an object? My first thought would be not really. Assuming only internal objects, i.e. no external connections or sockets, simply setting aside space on the heap then filling it with data has not seemed like an expensive process when using assembly or C, and it feels as though the JVM would have been optimised to a stage where it would be equally as painless. Likewise, given Java is almost OOP incarnate, it seems counter-intuitive to have a language focus around an expensive process, without dealing with it in some way.

It seems these sourcecs would agree.

* [This post](https://stackoverflow.com/questions/3485844/object-creation-in-java-is-slow/3485895#3485895) saw 4 million objects created at a cost of around 170 nanoseconds per object. Whether using a properly initialised array of int primitives would be faster is not really relevant when geniune objects only cost us 170 nanoseconds of performance, something confirmed in [this test](http://web.archive.org/web/20081223052756/http://www.christophkiehl.com/performance-tuning-object-creation-overhead).
* [This post](https://www.ibm.com/developerworks/java/library/j-jtp04223/index.html) on IBM.com agrees but is light on the empirical details.
It seems the major issue according to these sources, including a [warning against unnecessary object creation on the android documentation](https://developer.android.com/training/articles/perf-tips.html#object_creation) is garbage collection, and the pause involved while memory is freed. I'd like to investigate JVM and Java GC in detail, but getting back to DI.

It seems to me the main reason for dependency injection is not computing efficiency but rather programmer efficiency. Dealing with object dependencies, particularly dealing with the data in those objects that forms further dependencies, can get confusing when the dependency tree is extensive.
It becomes more complex when we consider that a single instance may need to be shared among a few or all of its dependencies. When is that instance then instantiated? A simple app=wide singleton pattern is easy enough to implement but one area of Dagger 2 I really like is the ability to scope the dependency areas to create temporary singletons.

This is where i feel Dagger 2 really comes into its own.

To explore this, consider a simple Interval or Fartlek timer app. One in which there is a 'work' period, a 'rest' period, and then a number of sets to repeat until the exercise session is finished.

Using the MainActivity to set up the app session details:

```java
public class MainActivity extends Activity {

private int sets;
private int timeOn;
private int timeOff;
private int minutes;
private int secs;
TextView setsView = findViewById(R.id.sets_view);
TextView workView = findViewById(R.id.time_on_view);
TextView restView = findViewById(R.id.time_off_view);

@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);
}

public void incrementSets(View view) {
sets++;
setsView.setText(sets);
}

public void decrementSets(View view) {
if (sets > 0) sets--;
setsView.setText(sets);
}

public void incrementTimeOn(View view) {
timeOn++;
minutes = timeOn/60;
secs = timeOn%60;
String time = String.format("%02d:%02d", minutes, secs);
workView.setText(time);
}

public void decrementTimeOn(View view) {
if (timeOn > 0) timeOn--;
minutes = timeOn/60;
secs = timeOn%60;
String time = String.format("%02d:%02d", minutes, secs);
workView.setText(time);
}

public void incrementTimeOff(View view) {
timeOff++;
minutes = timeOff/60;
secs = timeOff % 60;
String time = String.format("%02d:%02d", minutes, secs);
restView.setText(time);
}

public void decrementTimeOff(View view) {
if (timeOff > 0) timeOff--;
minutes = timeOff/60;
secs = timeOff % 60;
String time = String.format("%02d:%02d", minutes, secs);
restView.setText(time);
}

public void onStart(View view) {
Intent intent = new Intent(this, ActiveSessionActivity.class);
intent.putExtra("sets", sets);
intent.putExtra("timeOn", timeOn);
intent.putExtra("timeOff", timeOff);
startActivity(intent);
}
}
```
Then we instantiate our ActiveSessionActivity class that will contain the timers, and relies on an instance of timer.

```Java
public class ActiveSessionActivity extends Activity {

private final Timer timer = new Timer();
int seconds;
final TextView titleView = findViewById(R.id.get_ready);
final TextView timeView = findViewById(R.id.countdown_time_view);

@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_active_session);

// set up object information
Intent intent = getIntent();
seconds = timer.preCountdown;
timer.countdown = true;
timer.paused = false;
timer.sets = intent.getIntExtra("sets", 0);
timer.timeOn = intent.getIntExtra("timeOn", 0);
timer.timeOff = intent.getIntExtra("timeOff", 0);

// set up the activity
int minutes = seconds/60;
int secs = seconds%60;
String time = String.format("%02d:%02d", minutes, secs);
timeView.setText(time);
titleView.setText("Get Ready!");
runTimer();
}

public void runTimer() {
final Handler handler = new Handler();
handler.postDelayed(new Runnable() {
@Override
public void run() {
if (!timer.paused) {
seconds--;
}
if (seconds == 0) {
timerFinish();
}
int minutes = seconds/60;
int secs = seconds%60;
String time = String.format("%02d:%02d", minutes, secs);
timeView.setText(time);
handler.postDelayed(this, 1000);
}
}, 1000);
}

public void timerFinish() {
timer.sets--;
if (timer.countdown) {
timer.countdown = false;
seconds = timer.timeOn;
timer.work = true;
titleView.setText("Work!");
} else if (timer.work) {
timer.work = false;
seconds = timer.timeOff;
timer.rest = true;
titleView.setText("Rest!");
} else if (timer.rest) {
timer.rest = false;
if (timer.sets == 0) {
sessionFinish();
} else {
seconds = timer.timeOn;
timer.work = true;
titleView.setText("Work!");
}
}
}

public void sessionFinish() {
timer.paused = true;
timer.resetSession();
Intent intent = new Intent(this, MainActivity.class);
startActivity(intent);
}

public void onSessionPause(View view) {
timer.paused = true;
}

public void onSessionResume(View view) {
timer.paused = false;
}
}
```
Ignoring the inaccurate timer this will produce or the cumbersome logic, we can see a new instance of the Timer object is created every time we finish a session. With such a simple app and a lightweight object, clearly not an issue, but I'll use it as an opportunity to explore singleton dependency injection in Dagger 2.
If we assume we only ever want one Timer object to be created and to have the MainActivity simply update that object with new values after each session, which the ActiveSessionActivity will access.
The immediate issue is this object must be injected over the two activities. So, following [this talk by Jake Wharton](https://youtu.be/plK0zyRLIP8) and [this article](http://guides.codepath.com/android/Dependency-Injection-with-Dagger-2) we implement the following:

Firstly, a TimerModule which will be responsible for the object creation:
```java
@Module
public class TimerModule {

@Provides
@Singleton
Timer provideTimer() {
Timer timer = new Timer();
return timer;
}
}
```
and a TimerComponent, that will perform the actual injection, using the instance provided by TimerModule.
```java
@Singleton
@Component(modules={AppModule.class, TimerModule.class})
public interface TimerComponent {

void inject(MainActivity mainactivity);
void inject(ActiveSessionActivity activeSessionActivity);
}
```
Included in this TimerComponent is the AppModule class:
```java
@Module
public class AppModule {

    Application mApplication;

    public AppModule(Application application) {
        mApplication = application;
    }

    @Provides
    @Singleton
    Application providesApplication() {
        return mApplication;
    }

}
```
We need a singleton of Timer and to ensure a true singleton we need to ensure our scope is correct. By tying the TimerComponent to the AppModule, which is responsible for creating the Application class that starts the app, we've effectively created a global scope. While Dagger supports explicit Scope declarations (and indeed, any implementation requiring local or temporary singletons will require explicit scoping), this has created the global scope. Simply annotating our provideTimer() method with @Singleton would only ensure a Singleton for each instance of TimerComponent.
The final piece of the puzzle is completed in the class MyApp:
```java
public class MyApp extends Application {

    private TimerComponent timerComponent;

    @Override
    public void onCreate() {
        super.onCreate();

        // %Dagger_Component_Name%
        timerComponent = DaggerTimerComponent.builder()
                // list of modules that are part of this component need to be created here too
                .appModule(new AppModule(this))
                .timerModule(new TimerModule())
                .build();

    }
    public TimerComponent getTimerComponent() {
            return timerComponent;
    }
}
```
The MyApp, which we've now set as the launch point of the application, extending from Application, will build the TimerComponent, using the AppModule and TimerModule classes, and then inject the Timer class wherever it is needed.

So from one model class and two activities, we've blown out to four classes, two activities and an interface.

Consider however the benefit this scoping and intelligent placement of components can bring if you needed several Timer components, each owned by a distinct area of the App. 
I'll now consider a more appropriate example, requiring global, application lifetime Singletons, local Singletons over several activities and objects that die with their Activity.
