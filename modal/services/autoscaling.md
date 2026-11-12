---
url: https://alchemy.run/modal/services/autoscaling
title: "Autoscaling"
description: "Keep containers warm and cap concurrency for a deployed Modal function."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Modal scales a function’s containers with its calls, down to none when idle. A `Modal.FunctionAutoscaler` sets the bounds: containers kept warm, spare ones ready for the next spike, and a cap on how many may run.

## Warm containers skip the cold start

With `minContainers: 2` and `bufferContainers: 1`, two containers wait for calls and a third stands ready while they’re busy:

<svg viewBox="0 0 677.4 168" style="max-width: 779px" role="img" aria-label="The two minimum containers never stop. When both are busy, Modal starts a buffer container, so the next call doesn't wait for a cold start."><line stroke-opacity="0.2" stroke="currentColor" x1="273.4" y1="24" x2="273.4" y2="162"></line><g><text fill="currentColor" x="4" y="50">container 1</text> <text fill="currentColor" x="4" y="64">min</text> <g><rect stroke="currentColor" fill="none" x="105.4" y="40" width="112" height="24" rx="5"></rect><text fill="currentColor" x="161.4" y="56" text-anchor="middle">idle</text></g> <g><rect stroke="currentColor" fill="none" x="217.4" y="40" width="223.99999999999997" height="24" rx="5"></rect><text fill="currentColor" x="329.4" y="56" text-anchor="middle">call</text></g> <g><rect stroke="currentColor" fill="none" x="441.4" y="40" width="224" height="24" rx="5"></rect><text fill="currentColor" x="553.4" y="56" text-anchor="middle">idle</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="677.4" y2="74"></line><text fill="currentColor" x="4" y="94">container 2</text> <text fill="currentColor" x="4" y="108">min</text> <g><rect stroke="currentColor" fill="none" x="105.4" y="84" width="167.99999999999997" height="24" rx="5"></rect><text fill="currentColor" x="189.39999999999998" y="100" text-anchor="middle">idle</text></g> <g><rect stroke="currentColor" fill="none" x="273.4" y="84" width="224" height="24" rx="5"></rect><text fill="currentColor" x="385.4" y="100" text-anchor="middle">call</text></g> <g><rect stroke="currentColor" fill="none" x="497.4" y="84" width="168" height="24" rx="5"></rect><text fill="currentColor" x="581.4" y="100" text-anchor="middle">idle</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="118" x2="677.4" y2="118"></line><text fill="currentColor" x="4" y="138">container 3</text> <text fill="currentColor" x="4" y="152">buffer</text> <g><rect stroke="currentColor" fill="none" x="273.4" y="128" width="56" height="24" rx="5"></rect><text fill="currentColor" x="301.4" y="144" text-anchor="middle">idle</text></g> <g><rect stroke="currentColor" fill="none" x="329.4" y="128" width="140" height="24" rx="5"></rect><text fill="currentColor" x="399.4" y="144" text-anchor="middle">call</text></g></g> <text fill="currentColor" x="273.4" y="18" text-anchor="middle">both busy</text></svg> <svg viewBox="0 0 330 213" style="max-width: 379px" role="img" aria-label="The two minimum containers never stop. When both are busy, Modal starts a buffer container, so the next call doesn't wait for a cold start."><line stroke-opacity="0.2" stroke="currentColor" x1="95.39999999999999" y1="24" x2="95.39999999999999" y2="207"></line><g><text fill="currentColor" x="0" y="44">container 1 <tspan fill="currentColor" dx="8">min</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="63.6" height="24" rx="5"></rect><text fill="currentColor" x="31.8" y="70" text-anchor="middle">idle</text></g> <g><rect stroke="currentColor" fill="none" x="63.6" y="54" width="127.19999999999999" height="24" rx="5"></rect><text fill="currentColor" x="127.19999999999999" y="70" text-anchor="middle">call</text></g> <g><rect stroke="currentColor" fill="none" x="190.79999999999998" y="54" width="127.20000000000002" height="24" rx="5"></rect><text fill="currentColor" x="254.39999999999998" y="70" text-anchor="middle">idle</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="84" x2="330" y2="84"></line><text fill="currentColor" x="0" y="98">container 2 <tspan fill="currentColor" dx="8">min</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="108" width="95.39999999999999" height="24" rx="5"></rect><text fill="currentColor" x="47.699999999999996" y="124" text-anchor="middle">idle</text></g> <g><rect stroke="currentColor" fill="none" x="95.39999999999999" y="108" width="127.2" height="24" rx="5"></rect><text fill="currentColor" x="159" y="124" text-anchor="middle">call</text></g> <g><rect stroke="currentColor" fill="none" x="222.6" y="108" width="95.4" height="24" rx="5"></rect><text fill="currentColor" x="270.3" y="124" text-anchor="middle">idle</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="138" x2="330" y2="138"></line><text fill="currentColor" x="0" y="152">container 3 <tspan fill="currentColor" dx="8">buffer</tspan></text> <g><rect stroke="currentColor" fill="none" x="95.39999999999999" y="162" width="31.80000000000001" height="24" rx="5"></rect><text fill="currentColor" x="95.39999999999999" y="198" text-anchor="start">idle</text></g> <g><rect stroke="currentColor" fill="none" x="127.2" y="162" width="79.50000000000001" height="24" rx="5"></rect><text fill="currentColor" x="166.95000000000002" y="178" text-anchor="middle">call</text></g></g> <text fill="currentColor" x="95.39999999999999" y="18" text-anchor="middle">both busy</text></svg>

callwarm, waiting

The two minimum containers never stop. When both are busy, Modal starts a buffer container, so the next call doesn't wait for a cold start.

## Keep containers warm

```typescript
import * as Modal from "@alchemy.run/modal";

export const Warm = Modal.FunctionAutoscaler("Warm", {
  app: "inference",
  function: "Model.*",
  minContainers: 2,
  bufferContainers: 1,
  scaledownWindow: "5 minutes",
});
```

`scaledownWindow` is how long an idle container above the minimum waits before it stops.

## Cap concurrency

```typescript
export const Capped = Modal.FunctionAutoscaler("Capped", {
  app: "inference",
  function: "embed",
  maxContainers: 10,
});
```

## Settings return when you destroy it

Destroying the resource restores the settings it found. `modal deploy` resets them, so deploy your Stack after redeploying the app.
