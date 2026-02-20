# 0. The big picture

When you boot Linux, nothing graphical exists.

You must build this full pipeline:

```
Hardware (GPU / framebuffer)  
        ↓  
Kernel driver  
        ↓  
Userspace display server (you write this)  
        ↓  
Window manager / compositor (you write this)  
        ↓  
GUI toolkit / apps (you write or port)
```

You are basically recreating a simplified version of:

- X.Org Foundation (`X11` world)
- `Wayland` (modern Linux graphics)
- Microsoft Windows NT graphics stack

But tiny.

# Applications (client programs)

Apps cannot access framebuffer.
So you design IPC:
Options:
- Unix domain sockets
- shared memory (shm_open)
- pipes

Protocol example:
```
CREATE_WINDOW 400 300  
DRAW_RECT 10 10 50 50 RED  
TEXT "hello"  
FLUSH
```
Your server renders it.
This = mini Wayland protocol.

# What problem X.Org / Wayland actually solve

Linux kernel can:
- talk to GPU
- expose framebuffer (/dev/fb0 or DRM)

But **applications must NOT draw directly to screen** because:
- apps would overwrite each other
- no z-order
- no mouse focus
- no resizing
- no safety

So Linux invented a middleman:

> a **display server**

That server owns the screen.  
Programs only request drawing.
# The classic system — X.Org Foundation (X11)

Very old (1987 design)
## Architecture

```
App → X protocol → X Server → GPU → Monitor
```

Important idea:

> apps draw themselves  
> server only puts them on screen

So buttons, borders, etc are NOT inside X — they belong to a separate program:

```
**Window Manager**
```

Example tasks:
- draw window borders
- move windows
- title bar
- focus

So in X world:

```
X Server  = owns pixels  
Window Manager = decorations  
Toolkit (GTK/Qt) = widgets
```

That’s why Linux GUI feels layered and complex.
# The modern system — Wayland

Created because X11 became impossible to maintain.
## Architecture

```
App → buffer → compositor → GPU
```

Here the server itself moves windows and composes.

No separate window manager.

> Wayland is basically:  
> “the compositor IS the window manager”

Much simpler.

# What YOU want

You want something even simpler:

```
your programs → shared memory → your compositor → framebuffer
```

You are basically making a **tiny Wayland-like compositor** but without protocol complexity.

So ignore GTK, Qt, Xlib, etc.

# Minimal stack YOU should build

## Level 1 — screen control

Read:  
👉 [https://docs.kernel.org/fb/framebuffer.html](https://docs.kernel.org/fb/framebuffer.html)

Goal:
- draw pixels
- load wallpaper    
## Level 2 — input

Read:  
👉 [https://docs.kernel.org/input/input.html](https://docs.kernel.org/input/input.html)

Goal:
- mouse move
- click detection

## Level 3 — DRM/KMS (modern graphics, recommended)

Read:  
👉 [https://docs.kernel.org/gpu/drm-kms.html](https://docs.kernel.org/gpu/drm-kms.html)

This replaces framebuffer and works on modern GPUs.

## Level 4 — understand compositor design

Read Wayland internals (not to use, just to learn design):  
👉 [https://wayland.freedesktop.org/docs/html/ch03.html](https://wayland.freedesktop.org/docs/html/ch03.html)

This teaches window composition logic.

# What your final system will look like

Boot → start your program:

```
init  
 └── mydisplayd  
        ├ background image  
        ├ draw borders  
        ├ move window  
        └ draw cursor
```

Apps send raw pixel buffers.

That’s it.
# Absolute minimum features to implement

### Rendering
- load BMP/PPM wallpaper
- draw rectangle border
- alpha cursor

### Windowing
- window struct
- z-order list
- dragging
### Input
- focus
- click
- move

You’ll have a graphical OS already.
# Important mindset

You are **NOT building a desktop**  
You are building:

> a compositor

Exactly what Wayland compositors do — just tiny.
# KMS = Kernel Mode Setting

**Kernel Mode Setting** means:

> the Linux **kernel itself** controls the monitor and GPU display pipeline.

Before KMS existed, a userspace program (like the old X server from X.Org Foundation) had to switch the monitor resolution, timings, refresh rate, etc.

That caused:

- screen flicker during boot
- crashes = black screen
- slow startup
- ugly tty → GUI switch

So Linux moved display control into kernel.

Now:

```
BIOS → Kernel → KMS → screen stays stable forever
```

No mode switching chaos.

# What KMS actually controls (real hardware things)

Your monitor is NOT just pixels.  
There is a pipeline inside GPU:

```
Framebuffer → Plane → CRTC → Encoder → Connector → Monitor
```
### Translate to human language

|Term|Meaning|
|---|---|
|Framebuffer|memory containing pixels|
|Plane|layer (like a window layer)|
|CRTC|scanout engine (reads pixels line-by-line)|
|Encoder|converts digital signal|
|Connector|HDMI / eDP / VGA port|

KMS lets the kernel program all of that.

# Where DRM comes in

You always hear **DRM/KMS** together.

DRM = Direct Rendering Manager  
KMS = display configuration part of DRM

So:

```
/dev/dri/card0
```

is basically:

> the real GPU interface

# Difference from framebuffer (important)

## Old framebuffer (fbdev)

```
You → write memory → kernel copies → screen
```

Simple but fake-ish.

## KMS

```
You → ask GPU to scan out buffer → hardware displays it
```

You are controlling real display hardware.

# Why modern systems use it

Everything modern (including Wayland compositors):

- no screen flicker
- vsync
- multi-monitor
- hardware planes
- smooth animations

They all talk to DRM/KMS.

# For YOUR project

You actually have 2 choices:

### Easy path (recommended first)

Use framebuffer `/dev/fb0`  
→ learn windows & composition

### Real OS path (after it works)

Switch to KMS  
→ real GPU compositor

# One-sentence explanation

**Framebuffer:** “here’s memory, draw pixels”  
**KMS:** “here’s the monitor hardware, control how pixels reach it”