# Object Pooling

Reusing a fixed set of objects instead of constantly creating and destroying them. A generic concept (comes from game dev), but applies directly to any high-frequency loop in frontend: animations, canvas, particle systems.

## Why it matters

Creating a new object on every frame of an animation (60 times a second) generates garbage the garbage collector has to clean up — and that cleanup can pause the main thread right in the middle of an animation, causing the *jank* (visible stutter) that shows up as dropped frames.

```js
// ❌ a new particle every frame, thousands of discarded objects per second
function spawnParticle() {
  return { x: 0, y: 0, vx: Math.random(), vy: Math.random() };
}
function tick() {
  particles.push(spawnParticle()); // the GC has to clean up the discarded old ones
}
```

## The pattern

A fixed set of objects is pre-created at startup, and instead of `new`/discard, one inactive object is "borrowed" from the pool and "returned" when no longer needed.

```js
class ParticlePool {
  constructor(size) {
    this.pool = Array.from({ length: size }, () => ({ x: 0, y: 0, vx: 0, vy: 0, active: false }));
  }

  acquire() {
    const p = this.pool.find(p => !p.active);
    if (p) p.active = true;
    return p; // reuses an existing object, doesn't create a new one
  }

  release(p) {
    p.active = false; // goes back to the pool, ready for the next particle
  }
}

const pool = new ParticlePool(500);
const particle = pool.acquire();
// ... use the particle ...
pool.release(particle); // not discarded, recycled
```

Zero new allocations in the hot loop — the pool already has everything it's going to need reserved upfront.

## The same principle in virtualized lists

[Virtualization / Windowing](../diagnostics/frontend.md#missing-pagination--virtualization) applies the same idea at the DOM node level: instead of mounting and unmounting a `<div>` for every row entering/leaving the viewport while scrolling, virtualization libraries recycle a fixed handful of DOM nodes and just change their content — the DOM is also "expensive" to create/destroy, just like objects in a game loop's memory.
