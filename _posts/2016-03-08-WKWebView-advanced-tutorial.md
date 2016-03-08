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
    
    init(forWebView webView : WKWebView){
        
    }
    
    func userContentController(userContentController: WKUserContentController, didReceiveScriptMessage message: WKScriptMessage) {
        
    }
    
}
```

Once this is complete you need to implement the `scriptHandler`



