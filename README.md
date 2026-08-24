# smoothsync
SmoothSync is a frame-budgeted scheduling strategy that measures real-time loop execution against the OS kernel's interrupt quantum, dynamically capping per-frame math operations and deferring unfinished work across subsequent ticks to guarantee locked 60 FPS rendering.

*Co-authored by [sighthough](https://youtu.be/UtPiUGwu-0Q) and Googles Gemini 3.6.*

👉 **[CLICK HERE TO RUN THE LIVE BENCHMARK](https://sighthough.github.io/smoothsync/)**

feel free to rip anything you want from it , hope it helps

SmoothSync is a frame-budgeted scheduling architecture that prevents UI stutters by capping CPU workloads to match operating system timer quanta.

**Kernel Alignment & Quantum Timing**

* **Kernel Quantum Standard:** OS timer interrupts default to precise tick intervals—most notably the standard $15.625\text{ ms}$ ($64\text{ Hz}$) Windows kernel interrupt tick. SmoothSync uses this interval as its baseline frame deadline.
* **Safety Headroom Buffer:** SmoothSync subtracts a safety headroom (e.g., $1.5\text{ ms}$) from the quantum target. This sets an active work budget of $14.125\text{ ms}$ ($15.625\text{ ms} - 1.5\text{ ms}$), guaranteeing the thread finishes work and yields *before* the OS forcibly preempts it.

**Dynamic Workload Adjustment (Ops Slicing)**

* **Real-Time Loop Monitoring:** As the engine iterates through heavy math payloads, it checks high-resolution timing (`performance.now()`) after every object or operation.
* **Early Loop Termination:** Once execution time reaches the active budget ($14.125\text{ ms}$), the loop breaks immediately—even if only a fraction of the total objects were calculated.
* **Round-Robin State Persistence:** SmoothSync records the index of the last processed item, immediately hands control over to the renderer to draw a smooth frame, and picks up calculation on item $N + 1$ on the next frame tick.


Yes, you implement SmoothSync in any application by replacing rigid, synchronous loops with **time-budgeted cooperative loops** or **asynchronous yield patterns**.

**1. Time-Budgeted Loops (`performance.now()`)**
Instead of allowing a loop to process all items at once, measure execution time per tick and break out when hitting your limit, preserving progress for the next frame.

```javascript
let currentIndex = 0;
const TIME_BUDGET_MS = 12.0; // Reserve ~4.6ms for rendering in a 16.6ms frame

function processBatch(items) {
  const frameStart = performance.now();
  
  while (currentIndex < items.length) {
    processItem(items[currentIndex]);
    currentIndex++;

    // Yield control if time limit is reached
    if ((performance.now() - frameStart) >= TIME_BUDGET_MS) {
      requestAnimationFrame(() => processBatch(items));
      return;
    }
  }
  
  currentIndex = 0; // Reset index when all items finish
}

```

**2. Modern Web APIs (`scheduler.yield()`)**
Modern browsers offer `scheduler.yield()` to pause long-running JS tasks, handle pending user input and rendering, and immediately resume without dropping priority.

```javascript
async function processLargeDataset(items) {
  let lastYield = performance.now();

  for (let i = 0; i < items.length; i++) {
    processItem(items[i]);

    // Yield every 10ms to keep UI responsive
    if (performance.now() - lastYield > 10) {
      await scheduler.yield(); 
      lastYield = performance.now();
    }
  }
}

```

**3. Generator Functions (`yield`)**
Generators let you step through execution and manually pause work across multiple animation frames.

```javascript
function* taskStepGenerator(items) {
  for (let i = 0; i < items.length; i++) {
    processItem(items[i]);
    yield; // Pause point
  }
}

const iterator = taskStepGenerator(myArray);

function frameLoop() {
  const start = performance.now();
  // Keep stepping until budget expires or work finishes
  while (performance.now() - start < 12.0) {
    if (iterator.next().done) break;
  }
  requestAnimationFrame(frameLoop);
}

```

**4. Game Engines & C++**

* **Unity:** Use Coroutines (`yield return null` inside a `for` loop) or process sliced arrays inside `Update()` based on `Time.deltaTime`.
* **Unreal Engine / C++:** Implement time-sliced ticking using `FTickFunction` budgets or offload calculations to a background `FRunnable` thread that feeds data through thread-safe queues.

SmoothSync doesn't make the underlying math run faster; it simply prioritizes what human perception cares about most—smooth, uninterrupted visual motion over instantaneous total recalculation.

Trade-Offs & Best Practices
When to use SmoothSync: Game AI processing, heavy particle simulation updates, soft-body mechanics, procedural mesh updates, spatial audio calculations, or pathfinding algorithms where rendering smoothness is more critical than instantaneous synchronization of all world entities.

When NOT to use SmoothSync: Rigid physics solvers or precise collision matrices where missing a step per object results in physical instability or clipping artifacts.
