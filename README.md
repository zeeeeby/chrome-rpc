# Chrome RPC

A simple library for typed messaging between different parts of a Chrome extension (background script, content script, popup, etc.) 

No more subscribing to Chrome events or parsing message results - just write your methods and call them directly with full TypeScript support.  

## Usage Example

`methods.ts`

```typescript
export function getData(key: string) {
	return storage.get(key)
}
export function setData(key: string, value: any) {
	return storage.set(key, value)
}
export function removeData(key: string) {
	return storage.remove(key)
}
```

`background.ts`

```typescript
import * as methods from "./methods"

// Methods type is automatically inferred from the imported module
export type Methods = typeof methods

// Create a handler for incoming messages
const methodsApi = new Responder<Methods>("methods")

// Register all methods from the imported module
methodsApi.subscribeUniversal(async (name, args) => {
	const method = methods[name] as (...args: any[]) => any

	return method(...args)
})

export type Events = {
	dataChanged: (key: string) => void
	userLoggedIn: (userData: { id: string; name: string }) => void
	themeChanged: (theme: "light" | "dark") => void
}

const events = new Requester<Events>("events", {})

// Send events from background script
events.proxy.dataChanged("user-settings")
events.proxy.userLoggedIn({ id: "123", name: "John" })
events.proxy.themeChanged("dark")
```

`popup.ts, options.ts, content-script.ts`

```typescript
const events = new Responder<Events>("events")
const methods = new Requester<Methods>("methods")

// Subscribe to events
events.subscribe("dataChanged", async (key) => {
	// Update data
	const newData = await methods.proxy.getData(key)
	// ...
})

events.subscribe("userLoggedIn", (userData) => {
	// Update UI with user information
})

events.subscribe("themeChanged", (theme) => {
	// Apply new theme
	document.body.classList.toggle("dark", theme === "dark")
})
```

## Tab Filtering

When creating a Requester, you can specify `chrome.tabs.QueryInfo` to filter specific tabs:

```typescript
const events = new Requester<Events>("events", {
	active: true, // Only active tabs
	currentWindow: true, // Only current window
	url: ["*://*.github.com/*"], // Only GitHub tabs
})
```

This allows you to send messages only to tabs that match specific criteria.

## Features

-   Full method and argument typing
-   Async operation support
-   Automatic data serialization
-   Simple proxy interface for method calls
-   Support for multiple handlers per method
