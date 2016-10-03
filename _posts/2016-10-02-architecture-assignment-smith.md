---
layout: post
title:  "Architecture Assignment: MagicMirror Module Structure"
date:   2016-10-02 08:33:20 -0400
post_author: Ethan Smith
categories: smith
---

I spent this week focusing on the Architecture Assignment we have due soon. We divided the assignment up into 4 roughly equal segments. My segment was the detailed design, which involves laying out the implementation details of our project and how it will all fit together. We have the higher level design architecture section, so this one was much more focused on the specifics. 

To write this section I read through the relevant code in the exisiting MagicMirror project. A nontrivial part of our project will be understanding the infrastructure already put in place by MagicMirror and leveraging it to the fullest. With this in mind I got an understanding of how it works.

MagicMirror's plugin system consists of chunks of code called modules and organized into subfolders within a specific directory of the project. Each of these modules adheres to a set of guidelines on how magic mirror expects them to be organized. It will expect the following files to exist:

* modulename/modulename.js - The core file for your module, where all of the logic should be tied together
* modulename/node_helper.js - An optional file to be used if the application is being run on node.js
* modulename/public - Any files in this folder can be accesed via the browser on `/modulename/filename.ext`
* modulename/* - Anything else is considered a module specific dependency and left alone by MagicMirror's module system

Once you have this file skeleton setup you can set up the code within the core script. This will come in the form of a large call to the `Module.register()` function provided by MagicMirror. The hello world example looks like

````javascript
Module.register("helloworld",{
    // Default module config.
    defaults: {
        text: "Hello World!"
    },

    // Override dom generator.
    getDom: function() {
        var wrapper = document.createElement("div");
        wrapper.innerHTML = this.config.text;
        return wrapper;
    }
});
````

This helloworld example showcases the core functionality of a module, the default object and the getDom method that provides a hook to MagicMirror. The default object describes what configuration options are available and what their default values will be if they aren't configured. The getDom() method is expected to return an instance of DOMElement containing the display for this module. This should be a pure derivation of the DOMElement from the current state of the module when the method is called and should be side-effect free.

Past this, there are a couple of important lifecycle methods that MagicMirror will use if they exist on your module. A reference of them is provided below:

* `start()`
    This is used to notify the module that the application has completed loading and all other modules have been loaded. This is similar to a constructor for a more traditional object.

* `getScripts()`
    This should return an array of any third party librariers your module needs to load, either from a local file or from a url

* `getStyles()`
    This should return an array of stylesheets that should be loaded for this module to display, either local files or urls

* `notificationReceived(notification, payload, sender)`
    MagicMirror supports a communication system between modules and the core system (or modules and modules). This is done by handling the notificationReceived method which will be provided the notification itself, any data it has associated with it, and the module who sent it or undefined if the core system sent it.

* `socketNotificationReceived(notification, payload)`
    The node_helper can also send notifications similar to how the notificationReceived() method. Except these will be related to node.js events occurring.

* `suspend()`
    If your module is hidden, this method will be called to give you a chance to stop any ongoing tasks your module might be performing such as update timers or a long polling request to a server.

* `resume()`
    When your module is shown from being hidden, this method is called. It can be used to restart timers or animations, anything that would have been paused by `suspend()`

This covers most of the MagicMirror specific portions of the core script, the rest of it's content will be module specific code based on what your trying to accomplish.

The other major MagicMirror module file is the node_helper.js file. This is used if your MagicMirror instance is running on node and your module needs to do some form of IO through node.js. The node_helper.js file looks similar to a module:

````javascript
var NodeHelper = require("node_helper");
module.exports = NodeHelper.create({});
````

In place of an empty object `{}` you would usually place an object that defines hook methods (same as the module core script) like `start()`, or `init()`. These methods let you setup any state you need to perform IO and then send back the completed IO operations through the provided notification system with `sendSocketNotification()` method.

I like this notification system alot because it helps to enforce decoupling of modules from their dependencies. Especially the decoupling of IO from your module. A lot of errors come from not fully comprehending side effects, and most side effects come from performing IO. By forcing you to perform IO in a contained fashion you can avoid a lot of these errors and keep the logic of your module simple and devoid of any concept of IO. Your logic can only handle data and can't know about IO which I believe will lead to cleaner code.

This has been a short walkthrough of the module system provided by MagicMirror, there are some more pieces and this is not a full reference but it covers a lot of the major concepts and the remaining pieces are mostly for compatibility or edge case functionality.