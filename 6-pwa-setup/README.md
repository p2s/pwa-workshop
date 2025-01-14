---
title: 6. Showing the install button prompt
lang: en
---

# Step 6 : Showing the install button prompt

One of the advantages of PWA is that they can be installed if they respect certain criteria which depend on the browser. Here are the criteria for [Chrome](https://developers.google.com/web/fundamentals/app-install-banners/#criteria):

* The web app is not already installed.
  * and prefer_related_applications is not true.
* Meets a user engagement heuristic (currently, the user has interacted with the domain for at least 30 seconds)
* Includes a web app manifest that includes:
  * short_name or name
  * icons must include a 192px and a 512px sized icons
  * start_url
  * display must be one of: fullscreen, standalone, or minimal-ui
* Served over HTTPS (required for service workers)
* Has registered a service worker with a fetch event handler

Originally, the browser handled all the steps related to installation from the presentation of the banner to the addition of the app icon to the homescreen.
However, it is now possible for the app to handle the presentation of the UI that leads to the PWA install prompt.
In this case, the browser manages whether we should display that button and the presentation of the prompt.
The app should verify if it can present a setup UI by listening to the  and ask the browser to show the popup of the user requests it.

Let's add an **install** to our PWA by following these steps:

* Set the property [prefer_related_applications](https://developers.google.com/web/fundamentals/app-install-banners/native#prefer_related_applications) to `false` in the manifest file.

```json
{
  "prefer_related_applications": false,
}
```

* Add a button somewhere in your page and hide it by default

```html
<button id="setup_button" onclick="installApp()">Installer</button>
```

```css
#setup_button {
    display: none;
}
```

* In the main JavaScript file, intercept the `beforeinstallprompt` event which is fired when the PWA meets to install criteria. In this event event handler, we need to keep a reference to the event and show the setup button.

```js
let deferredPrompt; // Allows to show the install prompt
let setupButton;

window.addEventListener('beforeinstallprompt', (e) => {
    // Prevent Chrome 67 and earlier from automatically showing the prompt
    e.preventDefault();
    // Stash the event so it can be triggered later.
    deferredPrompt = e;
    console.log("beforeinstallprompt fired");
    if (setupButton == undefined) {
        setupButton = document.getElementById("setup_button");
    }
    // Show the setup button
    setupButton.style.display = "inline";
    setupButton.disabled = false;
});
```

* In the `installApp()` function, show the setup prompt.

```js
function installApp() {
    // Show the prompt
    deferredPrompt.prompt();
    setupButton.disabled = true;
    // Wait for the user to respond to the prompt
    deferredPrompt.userChoice
        .then((choiceResult) => {
            if (choiceResult.outcome === 'accepted') {
                console.log('PWA setup accepted');
                // hide our user interface that shows our A2HS button
                setupButton.style.display = 'none';
            } else {
                console.log('PWA setup rejected');
            }
            deferredPrompt = null;
        });
}
```

* Optionally, we can listen for the app installed event to perform any additional setup when the install finished

```js
window.addEventListener('appinstalled', (evt) => {
    console.log("appinstalled fired", evt);
});
```

Now's the time to test. Don't hesitate to reload the page by clearing the cache. In macOS, the shortcut is **⇧⌘R**

![PWA install button](./readme_assets/pwa_setup_button.png)
![PWA install prompt](./readme_assets/pwa_setup_prompt.png)

Here is how an installed PWA looks on macOS

![PWA installed](./readme_assets/pwa_installed.png)