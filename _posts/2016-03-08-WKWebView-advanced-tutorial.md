---
layout: post
title: WKWebView advanced tutorial (catch JS events, access properties etc...) (Swift)
---


# WKWebView advanced tutorial (catch JS events, access properties etc..) (Swift)

## Introduction
I have never been a fan of cross-platform, HTML based iOS and Android frameworks (PhoneGap, Cordova). They always seem to lag behind in features and responsiveness, and that's a compromise I'm rarely willing to take.

However, sometimes you can't avoid embedding HTML and JavaScript into your project. In those situations, iOS uses the `WKWebView` component for loading and displaying web pages embedded within the application. `WKWebView` is based on Safari browser and uses `webkit` so it's speed and responsiveness are on par with the latest and greatest of the mobile browser world.

In this tutorial, I'll show how to:
1. Embed a `WKWebView`inside of your iOS Universal App
2. Load an HTML webpage via URL
3. Perform an action on the `WKWebView` html via JavaScript as a result of native controll action
4. Respond to JavaScript events from your native application
5. Access JavaScript variables from your native application

## Preparation
In order to test our `WKWebView` behaviour, we will create a simple, static HTML page using `node.js`. NodeJS is a simple, lightweight JavaScript based web server, and as such - is perfect for the needs of this tutorial.

If you only need the iOS tutorial, you can skip this step.

Begin by cloning [this](https://github.com/mislavjavor/WKWebViewTutorial) `git` repository.

Make sure that you have **NodeJS** installed. There are a lot of tutorials on how to install NodeJS on the platform of your choice so that won't be covered in this tutorial.

After that move to the cloned folder and run

``` bash
npm install
```
and start the server by calling

``` bash
node server.js
```
Now go to your browser and open `localhost:3000`. You should see an image of either *Brad Pitt* or *Edward Norton* in the role of **Tyler Durden** from the 90's SF classic *Fight Club*

All this simple sample site does is expose a JS function `changeImage(actorName)`. Inspecting the JS code shows

``` javascript
function changeImage(actorName){
    var image = $("#tyler_durden_image");
    if(actorName == "pitt"){
        image.attr("src", "/durden_pitt.jpg");
        image.trigger("imagechanged", [true]);
    } else if(actorName == "norton"){
        image.attr("src", "/durden_norton.jpg");
        image.trigger("imagechanged", [true])
    } else{
        image.trigger("imagechanged", [false]);
    }
}

$("#tyler_durden_image").on("imagechanged", function(event, isSuccess){
    if(isSuccess){
        console.log("did it");
    }
});
```

We attached an event `imagechanged` to the `img` DOM object and we trigger it when the function changes the image. We also respond to that event by logging `"did it"` onto the JS console.

If you've done everything correctly, you should have your Node server running smoothly on `localhost:3000` and you're ready to begin with the iOS development.

## Creating the WKWebView and the WebView Wrapper

> If you'd like to access the files of the project used in this sample, it's available
> on github - https://github.com/mislavjavor/WKWebViewTutorial-iOS

### Setting up

Create a new iOS Universal App. Select `Swift` as a language. Once the project is open in Xcode, create a new `.swift` file called **WKWebViewWrapper**. This whill be the file into which we put our code for accessing variables and calling events

### Creating a WKWebView using Storyboards

#### Allow Arbitrary Loads

In order to comply to Apples *App Transport Security* you must whitelist all the domains your app will use. We will allow all domains with *Allow Arbitrary Loads*
> You should never use *Allow Arbitrary Loads* - this is for demonstration purposes only

Copy and paste this code into your *Info.plist* file

``` plist
<key>NSAppTransportSecurity</key>
<dict>
  <key>NSAllowsArbitraryLoads</key>
      <true/>
</dict>
```

#### Create and bind storyboard views

In your *storyboard* , place one `UIView` and one `UIButton`. Assuming the `UIView` is called `WKContainerView` and the `UIButton` is called `ChangeImageButton`, your *storyboard* should look like this:

![storyboard](http://i.imgur.com/eBBl9Gs.png)

1. *Ctrl+drag* `WKContainerView` to the `ViewController` as an `outlet` and call it `containerView`.
2. *Ctrl+drag* `ChangeImageButton` to the `ViewController` as an `action` of the `TouchUp Inside` type and call it `changeImageButtonClicked`

#### Create a WKWebView and load *localhost:3000*

In the *ViewController*, import `WebKit`

In your *ViewController* override the `viewWillAppear` method and initialize the `WKWebView` inside like this:

``` swift
override func viewWillAppear(animated: Bool) {
        super.viewWillAppear(animated)
        let wkWebView = WKWebView(frame: CGRect(x: 0, y: 0, width: view.frame.width, height: containerView.frame.height))
        view.addSubview(wkWebView)
        wkWebView.loadRequest(NSURLRequest(URL: NSURL(string: "http://localhost:3000")!))
    }
```

This process will, of course, vary massively from project to project since it's UI specific. Creating UIs is not the topic of this tutorial so those specifics are not important.

What is important is how you load the request into the `WKWebView`. You do this by calling
``` swift
if let url = NSURL(string: "http://localhost:3000") {
    wkWebView.loadRequest(NSURLRequest(URL: url));
}
```

It's implemented somewhat differently in the upper snippet of code but for a reason. The code in the latter snippet is good while the good in the first snippet is bad

> I'm leaving to the reader to deduce why the first code snippet is much worse than the second one

#### Finishing up in the ViewController

After everything, your *ViewController* should look like this

``` swift
import UIKit
import WebKit

class ViewController: UIViewController {

    @IBOutlet weak var containerView: UIView!

    override func viewDidLoad() {
        super.viewDidLoad()
    }

    override func viewWillAppear(animated: Bool) {
        super.viewWillAppear(animated)
        let wkWebView = WKWebView(frame: CGRect(x: 0, y: 0, width: view.frame.width, height: containerView.frame.height))
        view.addSubview(wkWebView)
        wkWebView.loadRequest(NSURLRequest(URL: NSURL(string: "http://localhost:3000")!))
    }

    @IBAction func changeImageButtonClicked(sender: AnyObject) {

    }
}
```

### Creating the WKWebViewWrapper

Navigate to the *WKWebViewWrapper.swift* file and do the following:

1. Create a class called `WKWebViewWrapper`
2. Import `Foundation` and `WebKit`
3. Make `WKWebViewWrapper` class inherit `NSObject` and implement the `WKScriptMessageHandler` protocol
4. Make an initializer that thakes a single `WKWebView` as a parameter

The end result should look something like this:

``` swift
import Foundation
import WebKit

class WKWebViewWrapper : NSObject, WKScriptMessageHandler{

    wkWebView : WKWebView

    init(forWebView webView : WKWebView){
        wkWebView = webView
        super.init()
    }

    func userContentController(userContentController: WKUserContentController, didReceiveScriptMessage message: WKScriptMessage) {

    }
}
```

Once you've done this, create a method called `setUpPlayerAndEventDelegation`. In it, we'll configure the `WKWebView` for receiveing JS events.

Before implementing that method, create a constant called `eventNames` in which you will save the names of all the events that your objects can fire (to be more precise - all the events that *you want to catch*)
e.g.
``` swift
let events = ["imagechanged", "documentReady"]
```

Now in the `setUpPlayerAndEventDelegation` create a `WKUserContentController` and assign it to the `controller` property of `wkWebView.configuration`. After that, use the controller to add all the events and make `self` the event listener.

`self` can be the event listener only because before we made `WKWebViewWrapper` implement the `WKScriptMessageHandler` protocol.

The `setUpPlayerAndEventDelegation` function should look something like this

``` swift
func setUpPlayerAndEventDelegation(){

        let controller = WKUserContentController()
        wkWebView.configuration.userContentController = controller

        for eventname in eventNames {
            controller.addScriptMessageHandler(self, name: eventname)
        }
    }
```

#### Initializing events as a dictionary of <String, EventHandler>

Here we use the very interesting property of the Swift programming lanugage that states
> Functions are first class objects

In the `WKWebViewWrapper` class, create a variable called `eventFunctions`. It should be a dictionary where the key is a `String` and the value a function that receives a `String` and returns `Void`. Declare the variable like this

``` swift
var eventFunctions : Dictionary<String, (String)->Void> = Dictionary<String, (String)->Void>()
```

in the `setUpPlayerAndEventDelegation` function, initialize each and every one of the functions declared in `eventNames` to be an empty function (we to this to assure that it's different than `null`)

To do that, in the `for eventname in eventNames` loop, under the `addScriptMessageHandler`, add the following line of code

``` swift
eventFunctions[eventname] = { _ in }
```

What this does is it initializes an empty function that does nothing for every entry in the `eventNames` constant

#### Inject event handlers
> Requires JQuery on the web server. It can be done without it, but I prefer using JQuery

In the `setUpPlayerAndEventDelegation`s for loop add the following line at the end:

``` swift
wkWebView.evaluateJavaScript("$(#tyler_durden_image).on('imagechanged', function(event, isSuccess) { window.webkit.messageHandlers.\(eventname).postMessage(JSON.stringify(isSuccess)) }", completionHandler: nil)
```

When we called `addScriptMessageHandler` before, WKWebView created a new `messageHandler` object on the `webkit` object that it injected during initialization. The name of the `messageHandler` is the name we gave to it in the `addScriptMessageHandler` and calling the `postMessage(String)` function on that message handler triggers the `userContentController` function that we implemented in order to satisfy the `WKScriptMessageHandler` protocol.

The `userContentController` function in our `WKWebViewWrapper` will be called every time the `postMessage` function is called on a `messageHandler`.

In the `userContentController` we will handle this triggering in the following way:

``` swift
func userContentController(userContentController: WKUserContentController, didReceiveScriptMessage message: WKScriptMessage) {
        if let contentBody = message.body as? String{
            if let eventFunction = eventFunctions[message.name]{
                eventFunction(contentBody)
            }
        }
    }
```

Now every time the function gets triggered, one of those "empty" functions that we declared earlier get called. Those functions will later be implemented by the ViewController that uses the `WKWebViewWrapper` so this function will effectively trigger those functions.

### Accessing javascript properties

#### Reasoning behind certain decisions

The main issue we will face here is approaching this issue from a proper perspective. We would like for our variables in the native app to be synchronized with the variables in the JavaScript frontend.

The easiest way to do this would be to simply call

``` swift
wkWebView.evaluateJavaScript("yourJavaScriptVariable", {
    result in
      //Handle your variable
})
```

but this gets us deep into closures and makes everything dependant on callbacks.
It's safe to say that this is not the ideal solution for every situation.

However, in some cases it may just be enought to get the job done without much hesitation.

Personally, what I like to do is define a JavaScript object with relevant data, and send it to the application once a second
via an event. The time period of one second is arbitrary and you can set it anyway you like.

If your object gets too heavy, you might want to employ batching and have several separate functions. Store all things in
compartments filtered by access times and weight. This will be very specific for each project and I won't be getting
into this topic very much in this article.

As a rule of thumb, I allocate a 100th of a second to each primitive variable sans strings. I have not empirically tested
this practice, but it's proven to be efficient in all of my tests so far.

#### Implementation

In your JavaScript, call

``` javascript
window.setInterval(function(){
  //We'll fill this later in the tutorial
}, 1000)
```
> NOTE: All of this can be done via injection of JavaScript in the WKWebView, but for the sake of tutorial, this
> gives the content much more clarification

This function repeats itself every 1000 milliseconds. This would be our "very slow" function for sending heavy objects.
You could implement something like this:

``` javascript
window.setInterval(function(){

}, 10)
```
for sending one `int` variable 100 times a second. Remember to never put any logic in these functions. They are for fetching only.

Let's prepare the scene for the sending of the event to the application:

Firstly, create an object which contains all of your data. So in the JavaScript project add:

``` javascript
 var applicationState = {
   // your application state
 }
```

In our concrete case it will be

``` javascript
var applicationState = {
  actorName = window.actorName
}
```

since this is the only variable we have. This may not be the best way to demonstrate this,
but I believe it server the purpose of demonstrating the principle quite well

After you created your `applicationState` object, you can now send it to your `Swift` code in your `setInterval` function.

``` javascript
window.setInterval(function(){
  window.webkit.messageHandlers.updateApplicationState.postMessage(JSON.stringify(window.applicationState))
}, 10)
```
Now add `updateApplicationState` into your `eventNames` variable. In the section where you handle the events, add special case
for the `updateApplicationState` event, parse the applicationState JSON object and update your local or external variables

### Performing actions of JavaScript properties and general behaviour

Now we've established getting our properties is a matter of interval updates, but accessing them is even easier.
All you need to do is call `evaluateJavaScript` with the desired javascript and you're done. Let's handle this the following way.
Create a function in JavaScript that performs some setting operation, such as

``` javascript
function setVariable(string actorName){
  window.actorName = actorName
}
```

Now in your wrapper, call

``` swift
wkWebView.evaluateJavaScript("setVariable(\(self.actorName)") //self.actorName is just a placeholder of course
```

or any other action you wish to perform

### Using the WKWebView

And you're more or less done. All you need to do now, is create the instance of your *Wrapper* in the ViewController and
add a function by calling

``` swift
wrapper.eventFunctions["functionName"] = {
  result in
    // Handle
}
```

Now with this knowledge, you should be able to modify the github projects and make your iOS device perform
actions on your JavaScript app

<div id="disqus_thread"></div>
<script>
    /**
     *  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
     *  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables
     */
    /*
    var disqus_config = function () {
        this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
        this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
    };
    */
    (function() {  // DON'T EDIT BELOW THIS LINE
        var d = document, s = d.createElement('script');

        s.src = '//mislavjavor.disqus.com/embed.js';

        s.setAttribute('data-timestamp', +new Date());
        (d.head || d.body).appendChild(s);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>

<script language="JavaScript">var fhsh = document.createElement('script');var fhs_id_h = "3111137";
fhsh.src = "//s1.freehostedscripts.net/ocount.php?site="+fhs_id_h+"&name=Visits&a=1";
document.head.appendChild(fhsh);document.write("<span id='h_"+fhs_id_h+"'></span>");
</script>
