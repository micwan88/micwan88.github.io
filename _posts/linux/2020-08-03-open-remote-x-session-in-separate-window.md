---
title: Open remote X session in separated window
categories: linux X11
---

Sometimes you would like to open remote X app / session in a separated window with specific size rather than the primary one. For this, you can use `xnest` to create a separated window like below:

``` bash
# xnest :display_number -geometry sizexsize
# Create a new display :2 with resolution size 800 x 600
xnest :2 -geometry 800x600

# Or you can use xinit to call xnest
#xinit -- `which xnest` :2 -geometry 800x600
```

After that, you can change `$DISPLAY` to point to your newly created window and that's it.

``` bash
export DISPLAY=:2
```

You can try to open simple X app (`xeyes`, `xclock`) to test it.
