# Module Federation

**Micro-frontends** architecture: splitting a large frontend app into independent pieces, each with its own build and deploy, that only get composed at **runtime** instead of all together in a single bundle.

## The problem it solves

A traditional SPA is a frontend monolith: all the code lives in one repo, one build, one deploy — if ten teams work on the same app, they all share the same pipeline and any release waits for everything to be ready. Micro-frontends apply to the UI the same idea microservices apply to the backend: each team owns a part of the screen, with its own development and deploy cycle.

## How it works (e.g. with Webpack Module Federation)

An app (`host`) can load a module served by another app (`remote`) at runtime, as if it were a normal import — but that code didn't even exist in the host's bundle when it was built.

```js
// remote's webpack.config.js (exposes a component)
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'checkout',
      filename: 'remoteEntry.js',
      exposes: { './CheckoutButton': './src/CheckoutButton' },
    }),
  ],
};

// host's webpack.config.js (consumes the remote)
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      remotes: { checkout: 'checkout@https://checkout.myapp.com/remoteEntry.js' },
    }),
  ],
};
```

```jsx
// in the host's code, it's imported as if it were local —
// but at runtime it's fetched from the checkout team's server
const CheckoutButton = React.lazy(() => import('checkout/CheckoutButton'));
```

## Trade-offs

It's not free: it adds **runtime coupling** (if the remote is down or changed its contract, the host breaks in production, not at build time), *version skew* between teams (the host might be running against a different remote version than the one it tested), and extra testing complexity (testing the full system requires spinning up several remotes at once). For small teams, or when the problem is just "sharing components" and not "independent deploys," a shared library in a monorepo is usually enough without paying that cost.
