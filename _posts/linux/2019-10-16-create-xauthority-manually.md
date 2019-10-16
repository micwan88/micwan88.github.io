---
title: How to create .Xauthority manually
categories: linux xauth
---

If .Xauthority does not exist in user home directory, it will show below error:

``` bash
xauth
xauth:  file /root/.Xauthority does not exist
Using authority file /root/.Xauthority
```

Below are the steps to manually create the `.Xauthority` under user home directory

``` bash
touch ~/.Xauthority

# Generate the magic cookie with 128 bit hex encoding
xauth add ${HOST}:0 . $(xxd -l 16 -p /dev/urandom)

# Verify the result and it shouldn't show any error
xauth list
```

References:
- [xauth not creating .Xauthority file](https://superuser.com/questions/806637/xauth-not-creating-xauthority-file)
