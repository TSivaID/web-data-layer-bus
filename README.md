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

## WebDataLayerBus Architecture Design

Here is a simple text-based diagram illustrating the architecture design of WebDataLayerBus:

```plaintext
                           ┌──────────────┐
                           │              │
                           │  Data Layer  │
                           │              │
                           └───────▲──────┘
                                   │
                                   │
                                   │ (Push data)
                                   │
┌─────┐      ┌─────┐      ┌─────┐  │  ┌─────┐
│     │      │     │      │     │  │  │     │
│  A  │◄─────┤  B  │◄─────┤  C  │◄─┼──┤  D  │
│     │      │     │      │     │  │  │     │
└─────┘      └─────┘      └─────┘  │  └─────┘
                                   │
                              (Subscribe)
                                   │
                       ┌───────────┴──────────┐
                       │                       │
                       │  Subscribers Registry │
                       │                       │
                       └───────────────────────┘
```

- The Data Layer is a central store for all data pushed from different modules (A, B, C, D).
- Each module (A, B, C, D) can subscribe to the Data Layer by providing a unique subscriber name and a callback function.
- The Subscribers Registry keeps track of all subscriber names and their respective callback functions.
- When new data is pushed into the Data Layer, the Subscribers Registry notifies each subscribed module by invoking their respective callback functions with the new data.

## Usage

1. **Integrate WebDataLayerBus into your project**

   To get started, simply embed the WebDataLayerBus JavaScript code snippet into your webpage. It is recommended to place the code inside the `<head>` tag, before loading any other dependencies, to ensure that the utility is available for all subsequent scripts.

   _**Original version of the WebDatalayerBus's snippet:**_

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
                try {
                  console.log("notifying " + subscriber + " with ", data);
                  window[dataLayerName + "Subscribers"][subscriber](data);
                } catch (error) {
                  console.error("Error in subscriber " + subscriber + ": ", error);
                }
              }
            }
          }
        };
      }
    
      // Expose WebDataLayerBus to the global scope
      window.WebDataLayerBus = WebDataLayerBus;
    })();
    ```

   _**Minified version of the WebDatalayerBus's snippet for production environments:**_

    ```javascript
    !function(){function e(e){e=e||"webDataLayerBus",window[e]=window[e]||[],window[e+"Subscribers"]=window[e+"Subscribers"]||{};var n=Array.prototype.push;window[e].push=function(t){if(t.subscriber&&t.callback&&"function"==typeof t.callback){if(window[e+"Subscribers"][t.subscriber])throw new Error("Subscriber "+t.subscriber+" already exists");window[e+"Subscribers"][t.subscriber]=t.callback;for(var r=0;r<window[e].length;r++)t.callback(window[e][r])}else{n.call(window[e],t);for(var s in window[e+"Subscribers"])Object.prototype.hasOwnProperty.call(window[e+"Subscribers"],s)&&function(){try{window[e+"Subscribers"][s](t)}catch(e){console.error("Error in subscriber "+s+": ",e)}}()}}}window.WebDataLayerBus=e}();
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

   function userSignoutHandlerInModuleA(data) {
     console.log("[MODULE A] User signed out:", data);
   }

   // Independent Module B
   function trackEventHandlerInModuleB(data) {
     console.log("[MODULE B] Track Event:", data);
   }
   ```

4. **Register action handlers for subscribing data layer updates**

   Register action handlers to process the data pushed to the data layer. Action handlers will be called when new data is pushed to the data layer.

   ```javascript
    // Independent Module A
    window.customWebDataLayerBus = window.customWebDataLayerBus || [];
    window.customWebDataLayerBus.push({
      subscriber: "moduleASubscriber",
      callback: function (data) {
        const actions = {
          "user-signin": userSigninHandlerInModuleA,
          "user-signout": userSignoutHandlerInModuleA,
        };
    
        const handler = actions[data.action];
        if (handler) {
          handler(data.payload);
        } else {
          console.log("[MODULE A] Unknown action: ", data.action);
        }
      },
    });
    
    // Independent Module B
    window.customWebDataLayerBus = window.customWebDataLayerBus || [];
    window.customWebDataLayerBus.push({
      subscriber: "moduleBSubscriber",
      callback: function (data) {
        const actions = {
          "track-event": trackEventHandlerInModuleB,
        };
    
        const handler = actions[data.action];
        if (handler) {
          handler(data.payload);
        } else {
          console.log("[MODULE B] Unknown action: ", data.action);
        }
      },
    });
   ```

5. **Push data to the data layer**

   Push new data to the data layer using the push method. All subscribed modules will be notified, and their callback functions will be called with the new data.

   ```javascript
   // Push data to the data layer from any module
   window.customWebDataLayerBus = window.customWebDataLayerBus || [];
   window.customWebDataLayerBus.push({ action: "user-signin", payload: { id: "abcxyz" } });
   window.customWebDataLayerBus.push({ action: "track-event", payload: { name: "user-signin", data: { id: "abcxyz" } } })
   window.customWebDataLayerBus.push({ action: "user-signout", payload: { id: "abcxyz" } });
   window.customWebDataLayerBus.push({ action: "track-event", payload: { name: "user-signout", data: { id: "abcxyz" } } })
   ```

## Browser Support

The `WebDataLayerBus` code is written using ECMAScript 5 (ES5) syntax, ensuring compatibility across a wide range of browsers. The code should function correctly in most modern browsers, as well as several older versions, including:

- Google Chrome: version 1 and later
- Mozilla Firefox: version 3.5 and later
- Safari: version 4 and later
- Microsoft Edge: all versions
- Internet Explorer: version 9 and later

While the code is compatible with older browser versions, it's recommended to use the latest browser versions for optimal performance, security, and compatibility with modern web standards.

## License

[MIT License](LICENSE)

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.
