---
layout: post
title: "Understanding Android Intents"
categories: android
excerpt_separator: <!--more-->
imageid: android.png
---

Intents are the key mechanism for starting, requesting and passing data between mechanisms, but I wanted to understand what was happening behind the Intents, what <!--more-->API's were in use and why Android chooses to do it this way.

### Examples of Intents

Some quick examples of an Intent, including the flow.
```Java
Intent intent = new Intent(this, SecondActivity.class); // Define an explicit intent 
intent.putExtra("Key", "Value"); // Add a bundle object to the Intent, more on that later.
startActivity(intent); // Call startActivity, using the created intent.
```
The putExtra can be overloaded to include many primitives and we can pass serialised objects too.

In the above example, the ```Intent``` object is constructed, using one of its six constructors, as an explicit Intent conatining the current Activity as a ```Context```.
These constructors include:
``` java
Intent() // Creates and empty intent
Intent(Intent o) // Copies another intent
Intent(String action) // Creates an Intent using a given action
Intent(String action, Uri uri) // Creates an Intent using a given action and with a given data Uri
Intent(Context packageContext, Class<?> cls) //Create an intent for a specific component.
Intent(String action, Uri uri, Context packageContext, Class<?> cls) // Create an intent for a specific component with a specified action and data.
```
As we can see above, we have different constructors for implicit intents, directed at any class that can handle that ```String action``` and ```Uri uri```, and explicit intents, directed at the specified ```Class<?> cls```.

The Intent Structure is structured to first be considered an implicit Intent with, according to the developer docs, action/data pairs as primary attributes. For example, ```ACTION_VIEW - content://contacts/people/1``` will display information about the contact with identifier 1. 
There are then many secondary attributes, which override or cause Android to ignore some or all evaluation of the primary attributes. For example, 
* The **category** attribute, which will specify certain restrictions on which activity, component or application can handle the intent. For example, ```CATEGORY_APP_MUSIC``` specifies the Intent would like the component to be able to handle music files. From the sender, these categories are used to describe a possible target. From the receiver's point of view, the categories tell Android what types of Intents the component can handle.
* The **type** attribute will specify the MIME type, which is usually inferred. Setting this will override Android evaluation and force the type.
* The **component** attribute, which allows us to specify the exact .class to target. This is the largest shortcircuit available when declaring an Intent. Usually, the target class is inferred from the action/data pair and the categories, and potentially the MIME type if necessary, and the selected by Android. Setting this atttribute will avoid any suitable component evaluation and send the user directly to the component. That means, once this is set, all other attributes become optional.
* The **extras** attribute is a ```Bundle``` of extra information. The Bundle is essentially a Java ```Map```, mapping values to ```String``` keys. The values must however be serializable and parcelable, and so the ```Map``` API is too flexible, allowing stream etc. The values can be added using ```putExtra(String key, ? value)```, or a Bundle added directly, ```putExtra(Bundle bundle)```.

### Intent Resolution
Explicit Intents are fairly clear and often used to simply load internal activities as the user moves about the app. Implicit Intents must go through a process of Intent Resolution, and it is the developer's responsibility to provide enough information about the intent to ensure successful resolution.
This is done by matching the intent with all of the ```<intent-filter>``` descriptions the system holds.
The filter uses action, type and category to make its decision and any activities that would like to handle these Intents must list the action and type as one it handles, and must list *all* the cateogries. This means the filter must list ```CATEGORY_DEFAULT``` as one it handles.
The docs go on to say, 
> Android automatically applies the the CATEGORY_DEFAULT category to all implicit intents passed to startActivity() and startActivityForResult(). So if you want your activity to receive implicit intents, it must include a category for "android.intent.category.DEFAULT" in its intent filters.

This seems to be a response to implicit intents with no category, that are then given ```CATEGORY_DEFAULT``` as a result. Given we need our list of ```<intent-filter>``` categories to match the Intent categories exactly, ```CATEGORY_DEFAULT`` must be on there for us to receive them. Why a default category is applied to Intents without any category in the first place, i.e. why Intents *need* a category, I'm not sure at the moment. 

For those still unsure of Implicit Intents, a good analogy I've heard are a room full of tradespeople representing the applications on a device, who each can perform one or more specific jobs. An application may need a roof to be completed, including building the timber frame, insulating the ceiling and laying the roof tiles. The application, not knowing who can help, will speak to the organiser of all tradespeople (in this case Android) and tell them what they need done. They'll provide an action (build a roof), with several categories (build the timber frame, laying the insulation, lay the roof tiles).
The tradespeople will have, at some stage, told Android what they can do. However, they do not tell Android what they cannot do. Android will assume if they have not said they can do it, they simply can't do it. Android will evaluate the job to be done, and select one or more tradespeople who can build a timber frame, lay insulation and lay the roof tiles. If any tradespeople can do only two of those tasks, they are not considered. If one tradesperson can do all three, but has only told Android they can build a timber frame and lay the roof tiles, they are not considered. Intent filters need to match the entire list of categories in order to be considered.
Also, consider a particularly skilled builder who can do many things. Aside from the three requested jobs, they can do plumbing, carpentry etc. Having these extra skills will not affect being matched for this particular job. That is, having extra intent filters that are not included in this intent will not disqualify the app from being considered.
Finally, consider a very specialised builder, who can do only certain jobs with certain materials. For example, they can build a tiled roof, but not a steel roof, but can do everything involved with building that tiled roof (insulation, timber frame etc) or can build a brick wall but not a wood cladding wall. If you are looking to handle many type of Intents, but only with specific combinations or **action**, **type** and **category**, then you'll need create several intent filters, in the same way the builder would need to have a long conversation with the organiser about which jobs they can handle to avoid the wrong clients and not miss out on any correct ones.