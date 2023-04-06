# Web Data Layer Bus

WebDataLayerBus is a custom data management utility that allows different independent modules to subscribe to updates in a shared data layer. It provides a simple yet powerful solution for managing data across multiple components without relying on any external libraries.

The architecture of WebDataLayerBus is based on the publish-subscribe pattern. When new data is pushed to the data layer, all subscribed modules are notified, and their respective callback functions are called with the new data.

## Features

- Customizable data layer names
- Subscribe to updates in the data layer
- Push new data to the data layer
- Notify all subscribers when new data is pushed
- Loading independent modules order does not matter since it processes all messages in the array when a new subscriber is added.

## Usage

1. **Embed WebDataLayerBus code**

   Copy the the follwing js code snippet to your webpage inside `<head>` tag before call all depdendencies

   ```javascript
   (function () {
     function WebDataLayerBus(dataLayerName) {
       dataLayerName = dataLayerName || "webDataLayerBus";
       window[dataLayerName] = window[dataLayerName] || [];
       window[dataLayerName + "Subscribers"] = window[dataLayerName + "Subscribers"] || {};

       var originalPush = Array.prototype.push;
       window[dataLayerName].push = function () {
         // Call the original push function with the provided arguments
         originalPush.apply(window[dataLayerName], arguments);
         console.log("pushed with ", arguments[0]);

         // Notify subscribers
         for (var subscriber in window[dataLayerName + "Subscribers"]) {
           if (Object.prototype.hasOwnProperty.call(window[dataLayerName + "Subscribers"], subscriber)) {
             console.log("notifying " + subscriber + " with ", arguments[0]);
             window[dataLayerName + "Subscribers"][subscriber](arguments[0]);
           }
         }
       };

       this.subscribe = function (subscriber, callback) {
         if (window[dataLayerName + "Subscribers"][subscriber]) {
           throw new Error("Subscriber " + subscriber + " already exists");
         }
         window[dataLayerName + "Subscribers"][subscriber] = callback;

         // Process the existing dataLayer
         for (var i = 0; i < window[dataLayerName].length; i++) {
           console.log("notifying " + subscriber + " with ", window[dataLayerName][i]);
           callback(window[dataLayerName][i]);
         }
       };
     }

     // Expose WebDataLayerBus to the global scope
     window.WebDataLayerBus = WebDataLayerBus;
   })();
   ```

2. **Initialize the data layer**

   Create a new instance of WebDataLayerBus and pass a custom data layer name as an argument below the embedded code. If no argument is provided, the default name "webDataLayerBus" will be used.

   ```javascript
   var customWebDataLayerBusInstance = new WebDataLayerBus("customWebDataLayerBus");
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
   window.customWebDataLayerBusInstance.subscribe("moduleASubscriber", function (data) {
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
   });

   // Independent Module B
   window.customWebDataLayerBusInstance.subscribe("moduleBSubscriber", function (data) {
     switch (data.action) {
       case "user-signin":
         userSigninHandlerInModuleB(data.payload);
         break;
       default:
         break;
     }
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

In this way, independent modules can subscribe to updates in the shared data layer and process the data pushed to the data layer. The order in which the independent modules are loaded does not matter since it processes all messages in the array when a new subscriber is added.

## License

MIT License

Copyright (c) [2023] [T. Siva](https://github.com/TSivaID)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.