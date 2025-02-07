---
description: >-
  All of the content below is specific to x86-64  (also known as AMD64 or Intel
  64) unless shown otherwise.
---

# Refreshers

{% hint style="info" %}
This is not a reference for starting with assembly, instead the content below is a form of refreshers to basic and fundamental concepts of assembly.&#x20;

It happens that we could forget few things about calling conventions, stack frames. Etc , So this should be helpful and self explanatory.
{% endhint %}

## Calling Conventions

Where are the function arguments are stored, how are they stored? and in which case are they stored that way?

Let's start with this simple C code:

```c
#include <stdio.h>

int func(int a, int b, int c, int d, int e, int f, int g, int h, int i){
        return 1;
}
int main(int argc, int *argv[]){
        func(1,2,3,4,5,6,7,8,9);
        return 1;
}
```

Simple C code that defines func with 9 arguments and the main func.&#x20;

<figure><img src="../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

