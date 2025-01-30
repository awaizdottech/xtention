facts about the extension manifest

- It must be located at the root of the project.
- It supports comments (//) during development, but these must be removed before uploading your code to the Chrome Web Store.
- The only required keys are "manifest_version", "name", and "version".
- Required value 'version' is missing or invalid. It must be between 1-4 dot-separated integers each between 0 and 65536.
- Missing 'manifest_version' key. Its value must be an integer either 2 or 3. See developer.chrome.com/extensions/manifestVersion for details. Could not load manifest.
- icons are optional during development, they are required if you plan to distribute your extension on the Chrome Web Store. using PNG files recommend, but other file formats are allowed, except for SVG files.

````Icon size | Icon use
16x16 |	Favicon on the extension's pages and context menu.
32x32 |	Windows computers often require this size.
48x48 |	Displays on the Extensions page.
128x128 |	Displays on installation and in the Chrome Web Store.```

- ```
  Extension component	| Requires extension reload
  The manifest | Yes
  Service worker | Yes
  Content scripts | Yes (plus the host page)
  The popup | No
  Options page | No
  Other extension HTML pages | No
````

```

Key point: Update chrome types npm package frequently to work with the latest Chromium version.
```

- Match patterns consist of three parts: <scheme>://<host><path>. They can contain '\*' characters.
- Extensions can use the Storage API and IndexedDB to store the application state.
- The activeTab permission grants the extension temporary ability to execute code on the active tab. It also allows access to sensitive properties of the current tab.

This permission is enabled when the user invokes the extension. In this case, the user invokes the extension by clicking on the extension action.

💡 What other user interactions enable the activeTab permission in my own extension?

Pressing a keyboard shortcut combination.c
Selecting a context menu item.
Accepting a suggestion from the omnibox.
Opening an extension popup.

The "activeTab" permission allows users to purposefully choose to run the extension on the focused tab; this way, it protects the user's privacy. Another benefit is that it does not trigger a permission warning.

- Success: The Scripting API does not trigger a permission warning.
- 💡 Can I use the Scripting API to inject code instead of a style sheet?

Yes. You can use scripting.executeScript() to inject JavaScript.

- The "\_execute_action" key runs the same code as the action.onClicked() event, so no additional code is needed.
- A popup is similar to a web page with one exception: it can't run inline JavaScript.
- The Tabs API allows an extension to create, query, modify, and rearrange tabs in the browser.
- Many methods in the Tabs API can be used without requesting any permission. However, we need access to the title and the URL of the tabs; these sensitive properties require permission. We could request "tabs" permission, but this would give access to the sensitive properties of all tabs. Since we are only managing tabs of a specific site, we will request narrow host permissions.

Narrow host permissions allow us to protect user privacy by granting elevated permission to specific sites. This will grant access to the title, and URL properties, as well as additional capabilities. Add the highlighted code to the manifest.json file:

- 💡 What are the main differences between the tabs permission and host permissions?

Both the "tabs" permission and host permissions have drawbacks.

The "tabs" permission grants an extension the ability to read sensitive data on all tabs. Over time, this information could be used to collect a user's browsing history. Host permissions allow an extension to read and query a matching tab's sensitive properties, plus inject scripts on these tabs. The warnings that they generate at installation can be alarming for users. For a better onboarding experience, we recommend implementing optional permissions.

- 💡 Can I use Chrome APIs directly in the popup?

A popup and other extension pages can call any Chrome API because they are served from the chrome schema. For example chrome-extension://EXTENSION_ID/popup.html.

- The TabGroups API allows the extension to name the group and choose a background color.
- Chrome will shut down service workers if they are not needed. We use the chrome.storage API to persist state across service worker sessions. For storage access, we need to request permission in the manifest:
- Service workers don't have direct access to the window object and therefore cannot use window.localStorage to store values. Also, service workers are short-lived execution environments; they get terminated repeatedly throughout a user's browser session, which makes them incompatible with global variables. Instead, use chrome.storage.local which stores data on the local machine.
- All event listeners need to be statically registered in the global scope of the service worker. In other words, event listeners shouldn't be nested in async functions. This way Chrome can ensure that all event handlers are restored in case of a service worker reboot.
- Key point: Extension service workers can use both web APIs and Chrome APIs, with a few exceptions.
- setTimeout & setInterval can fail because the scheduler will cancel the timers when the service worker is terminated. Instead, extensions can use the chrome.alarms API.
