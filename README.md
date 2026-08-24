# smoothsync
SmoothSync is a frame-budgeted scheduling strategy that measures real-time loop execution against the OS kernel's interrupt quantum, dynamically capping per-frame math operations and deferring unfinished work across subsequent ticks to guarantee locked 60 FPS rendering.
