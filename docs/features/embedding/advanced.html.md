---
title: Advanced Embed
layout: docs
categories: ["Features","Embedding"]
---

# Advanced Embed

This embedding technique is for developers who require programmatic interaction between the Vanilla iframe and the parent window. It employs easyXDM and a special container layer to achieve this, and requires a much more indepth setup than our standard embed solution.

## Setting it up

1. Include easyXDM: `/js/easyXDM/easyXDM.min.js`
2. Include Vanilla's embed object: `/js/vanilla.embed.js`
3. Create a div to contain Vanilla and give it an ID.
4. Call `Vanilla.embed({options})`.

### Options

Vanilla.embed takes an associative array of options:

`root` **Required**. The root url of the Vanilla. Example: http://community.yoursite.com

`container` **Required**. The ID or dom element of the container for Vanilla.

`initialPath` **Optional**. The initial path to browse to when embedding the forum. Our example below demonstrates using the hash part of the URL to automatically set this, e.g. `site.com/embedpage#/categories/some-category`. 

`sso` **Optional**. An SSO string that will automatically sign the user into Vanilla. See https://github.com/vanillaforums/jsConnectPHP/blob/master/functions.jsconnect.php#L130

`autoStart` **Optional**. Whether or not to start the embed when embed() is called. If this is false then you must call embed.start() to start the embed. Default: true

`onReady()` **Optional**. A function to call when the embed is ready.

`notifyLocation(path)` **Optional**. A function that will be called when the url of the embedded iframe changes. You can use this callback to update your history. By default this function adds the path as the current locations #. Combine this with initialPath to implement your own custom history.

`height(x)` **Optional**. A function that is called to set the height of the embedded iframe. If you override this method then you can access the embedded iframe using this.iframe.

### Methods

`start()` Start the embed. Only call this method if you set autoStart to false.

`setLocation(path)` Manually set the location of the embedded iframe. Just specify the path and not the full domain of the embed.

## Custom Callbacks

The embed API allows custom Javascript to be called between the parent and child iframes.

## Exposing a method to Vanilla

You can expose a method to the child iframe with the following code:

    Vanilla.embed.fn.functionName = function(args,...) { … }

The embed requires all functions on both sides to be explicitly registered to prevent a third party hijacking the embed and calling functions with unforeseen consequences.

Vanilla embed supports a callback function on both ends of the embed. If you want to support a callback function then declare it as the last argument of your function. When the other endpoint of the embed calls your function using `callRemote()` it can then specify a callback and the framework will handle calling the callback across the domain. 

## Calling a method in Vanilla

You can call a method in Vanilla with the following code:

    Vanilla.embed.callRemote(“functionName”, args [, callback]);
    
If you supply the callback argument to `callRemote()` then it will be supplied as the last argument to the remote function. It is the remote function’s job to call the function. Keep in mind that using `call()`, `apply()`, etc. to set a context for the function will not work as a context cannot be passed across the domain.

### Functions you can call on Vanilla

`Vanilla.embed.signout()` Calling this function will sign the user out of Vanilla. It won’t refresh the page so you may want to do that afterwards.


## Example implementation

```
<html>
   <head>
      <meta name="viewport" content="width=device-width,minimum-scale=1.0,maximum-scale=1.0" />
      <style>
         body {
            margin: 0;
         }
         #embedVanilla iframe {
            width: 100%;
         }
      </style>
      <script type="text/javascript" src="js/easyXDM/easyXDM.min.js"></script>
      <script type="text/javascript" src="js/vanilla.embed.js"></script>
   </head>
   <body style="background: #ff6600;">
      <div style="height: 50px">foo</div>
      <div id="embedVanilla"></div>
      <script type="text/javascript">
         easyXDM.DomHelper.requiresJSON("js/easyXDM/json2.js"); // Help older browsers parse JSON.

         var foo = new Vanilla.embed({
            container: "embedVanilla", // The element or its ID.
            root: "http://yoursite.local", // This is the location of the forum.
            initialPath: window.location.hash.substring(1), // initial path, please start it with a "/"
            onReady: function() {
               // A lot of the functions you perform need to be done after the embed is ready.

               // This how you set the location of the embedded frame.
               // foo.setLocation('/categories');
            }
         });
      </script>
   </body>
</html>
```