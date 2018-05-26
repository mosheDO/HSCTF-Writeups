# Alarm (Binary Exploitation, 400 points)

We are given a binary and no source code this time! Let's try running it and testing some inputs, much like we did for [Review](../review-100/README.md).

![term1.png](term1.png)

Hmm...that "Alarm name: " looks rather suspicious at the bottom. Let's examine the objdump.

Right away, we identify `00000000000011b0 <what>:` as our probable flag function. 