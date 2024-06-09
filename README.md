# webext-zustand

Use zustand to share state between pages and background in web extensions.

## 🚀 Quick Start

```bash
npm install webext-zustand
```

- Create a store using Zustand (https://github.com/pmndrs/zustand). You can create a store using either the reactive (`create`) or vanilla (`createStore`) approach.
- Wrap the store with `wrapStore` from `webext-zustand`.
  - This allows the store to be shared between the background script and other parts of the extension.
- In the background script (`background.ts`):
  - You must either subscribe to the store using `store.subscribe()` or wait for the `applicationStoreReadyPromise` to resolve before accessing the store.
  - This ensures that the store is properly initialized and ready for use.
- In the popup or content script (using ReactDOM):
  - Import the store and the `storeReadyPromise` from the store module.
  - Await the `storeReadyPromise` using `.then()` before rendering your React component.
  - This ensures that the store is connected to the background and ready to be used in your component.


That's it! Now your store is available from everywhere.

> You can try out the basic example app in the `examples` folder.

`store.ts`

```js
import { create } from 'zustand'
// or import { createStore } from 'zustand/vanilla'
import { wrapStore } from 'webext-zustand'

interface BearState {
  bears: number
  increase: (by: number) => void
}

export const useBearStore = create<BearState>()((set) => ({
  bears: 0,
  increase: (by) => set((state) => ({ bears: state.bears + by })),
}))

export const storeReadyPromise = wrapStore(useBearStore);

export default useBearStore;
```

`background.ts`

```js
import store from "./store";
// IMPORTANT: In the background script, you must either subscribe to the store
// or wait for the applicationStoreReadyPromise before accessing the store.
// Choose one of the following options:
// Option 1: Subscribe to the store
store.subscribe((state) => {
  // console.log(state);
});
// Option 2: Wait for the applicationStoreReadyPromise
// applicationStoreReadyPromise.then(() => {
//   console.log("ready");
// });
// After choosing one of the above options, you can safely access the store
// and dispatch actions, for example:
// store.getState().increase(2);
```

`popup.tsx`

```js
import React from "react";
import { createRoot } from "react-dom/client";

import { useBearStore, storeReadyPromise } from "./store";

const Popup = () => {
  const bears = useBearStore((state) => state.bears);
  const increase = useBearStore((state) => state.increase);

  return (
    <div>
      Popup
      <div>
        <span>Bears: {bears}</span>
        <br />
        <button onClick={() => increase(1)}>Increment +</button>
      </div>
    </div>
  );
};

storeReadyPromise.then(() => {
  createRoot(document.getElementById("root") as HTMLElement).render(
    <React.StrictMode>
      <Popup />
    </React.StrictMode>
  );
});
```

`content-script.tsx`

```js
import React from "react";
import { createRoot } from "react-dom/client";
import { useBearStore, storeReadyPromise } from "./store";

const Content = () => {
  const bears = useBearStore((state) => state.bears);
  const increase = useBearStore((state) => state.increase);

  return (
    <div>
      Content
      <div>
        <span>Bears: {bears}</span>
        <br />
        <button onClick={() => increase(1)}>Increment +</button>
      </div>
    </div>
  );
};

storeReadyPromise.then(() => {
  const root = document.createElement("div");
  document.body.prepend(root);

  createRoot(root).render(
    <React.StrictMode>
      <Content />
    </React.StrictMode>
  );
});
```

## Architecture

Current implementation is based on the
https://github.com/eduardoacskimlinks/webext-redux (which is a fork of `tshaddix/webext-redux`)

This library adds a minimal layer to support `zustand` and automatic runtime environment detection.

You can find more information on the `webext-redux` package.

## License

[MIT](./LICENSE) License © 2023 [Sinan Bekar](https://github.com/sinanbekar)
