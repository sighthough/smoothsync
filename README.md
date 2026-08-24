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
