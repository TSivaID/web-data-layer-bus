# Web Data Layer Bus

WebDataLayerBus is a lightweight and flexible data management utility designed to facilitate seamless communication between independent modules in web applications. By utilizing a shared data layer and the publish-subscribe pattern, WebDataLayerBus enables different components to subscribe to updates and react to changes in the data layer, ensuring efficient data synchronization across your application.

The core functionality of WebDataLayerBus revolves around its ability to push data to a shared data layer, while automatically notifying subscribed modules of any updates. This approach simplifies data management and promotes modular, decoupled architecture in web applications, without the need for external dependencies or complex integration.

## Features

- **Customizable data layer names:** Allows you to create multiple data layers with unique names, avoiding conflicts between different parts of your application.
- **Subscribe to updates in the data layer:** Modules can subscribe to specific events in the data layer, receiving updates when new data is pushed.
- **Push new data to the data layer:** Modules can push new data or events to the data layer, triggering updates for subscribed modules.
- **Notify all subscribers when new data is pushed:** When new data is pushed to the data layer, all subscribed modules are notified, allowing them to react accordingly.
- **Late subscription handling:** If a module subscribes after some data has already been pushed to the data layer, it will still receive and process all the previous messages, ensuring that the loading order of independent modules does not matter.
- **Error handling:** Prevents duplicate subscriptions by throwing an error if a module tries to subscribe with an already-existing subscriber name.
- **Minified version available:** A minified version of the library is provided for production use, without any `console.log` messages. It's ready to be embedded in your webpage without any build process.

## Usage

1. **Embed WebDataLayerBus code**

   Copy the the follwing js code snippet to your webpage inside `<head>` tag before call all depdendencies

   _Original version:_

   ```javascript
   (function () {
     function WebDataLayerBus(dataLayerName) {
       dataLayerName = dataLayerName || "webDataLayerBus";
       window[dataLayerName] = window[dataLayerName] || [];
       window[dataLayerName + "Subscribers"] = window[dataLayerName + "Subscribers"] || {};

       var originalPush = Array.prototype.push;
       window[dataLayerName].push = function (data) {
         if (data.subscriber && data.callback && typeof data.callback === "function") {
           if (window[dataLayerName + "Subscribers"][data.subscriber]) {
             throw new Error("Subscriber " + data.subscriber + " already exists");
           }
           window[dataLayerName + "Subscribers"][data.subscriber] = data.callback;

           // Process the existing dataLayer
           for (var i = 0; i < window[dataLayerName].length; i++) {
             data.callback(window[dataLayerName][i]);
           }
         } else {
           // Call the original push function with the provided arguments
           originalPush.call(window[dataLayerName], data);
           console.log("pushed with ", data);

           // Notify subscribers
           for (var subscriber in window[dataLayerName + "Subscribers"]) {
             if (Object.prototype.hasOwnProperty.call(window[dataLayerName + "Subscribers"], subscriber)) {
               console.log("notifying " + subscriber + " with ", data);
               window[dataLayerName + "Subscribers"][subscriber](data);
             }
           }
         }
       };
     }

     // Expose WebDataLayerBus to the global scope
     window.WebDataLayerBus = WebDataLayerBus;
   })();
   ```

   _Minified version:_

   ```javascript
    !function(){function t(t){t=t||"webDataLayerBus",window[t]=window[t]||[],window[t+"Subscribers"]=window[t+"Subscribers"]||{};var e=Array.prototype.push;window[t].push=function(n){if(n.subscriber&&n.callback&&"function"==typeof n.callback){if(window[t+"Subscribers"][n.subscriber])throw new Error("Subscriber "+n.subscriber+" already exists");window[t+"Subscribers"][n.subscriber]=n.callback;for(var r=0;r<window[t].length;r++)n.callback(window[t][r])}else{e.call(window[t],n);for(var i in window[t+"Subscribers"])Object.prototype.hasOwnProperty.call(window[t+"Subscribers"],i)&&window[t+"Subscribers"][i](n)}}}window.WebDataLayerBus=t}();
   ```

2. **Initialize the WebDataLayerBus**

   Create a new instance of WebDataLayerBus and pass a custom data layer name as an argument below the embedded code. If no argument is provided, the default name "webDataLayerBus" will be used.

   ```javascript
   (function () {
     new WebDataLayerBus("customWebDataLayerBus");
   })();
   ```

3. **Create action handlers**

   Create action handlers that will process the data pushed to the data layer.

   ```javascript
   // Independent Module A
   function userSigninHandlerInModuleA(data) {
     console.log("[MODULE A] User signed in:", data);
   }

   function userCategoryFetchHandlerInModuleA(data) {
     console.log("[MODULE A] User categories fetched:", data);
   }

   function userCategorySavedHandlerInModuleA(data) {
     console.log("[MODULE A] User categories saved:", data);
   }

   // Independent Module B
   function userSigninHandlerInModuleB(data) {
     console.log("[MODULE B] User signed in:", data);
   }
   ```

4. **Register action handlers for subscribing data layer updates**

   Register action handlers to process the data pushed to the data layer. Action handlers will be called when new data is pushed to the data layer.

   ```javascript
   // Independent Module A
   window.customWebDataLayerBus.push({
     subscriber: "moduleASubscriber",
     callback: function (data) {
       switch (data.action) {
         case "user-signin":
           userSigninHandlerInModuleA(data.payload);
           break;
         case "user-category-fetch":
           userCategoryFetchHandlerInModuleA(data.payload);
           break;
         case "user-category-saved":
           userCategorySavedHandlerInModuleA(data.payload);
           break;
         default:
           break;
       }
     },
   });

   // Independent Module B
   window.customWebDataLayerBus.push({
     subscriber: "moduleBSubscriber",
     callback: function (data) {
       switch (data.action) {
         case "user-signin":
           userSigninHandlerInModuleB(data.payload);
           break;
         default:
           break;
       }
     },
   });
   ```

5. **Push data to the data layer**

   Push new data to the data layer using the push method. All subscribed modules will be notified, and their callback functions will be called with the new data.

   ```javascript
   // Push data to the data layer
   window.customWebDataLayerBus.push({ action: "user-signin", payload: { id: "abcxyz" } });
   window.customWebDataLayerBus.push({ action: "user-category-fetch", payload: { id: "abcxyz", categories: [] } });
   window.customWebDataLayerBus.push({
     action: "user-category-saved",
     payload: { id: "abcxyz", categories: ["News"] },
   });
   window.customWebDataLayerBus.push({
     action: "user-category-fetch",
     payload: { id: "abcxyz", categories: ["News"] },
   });
   ```

## License

[MIT License](LICENSE)

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.
