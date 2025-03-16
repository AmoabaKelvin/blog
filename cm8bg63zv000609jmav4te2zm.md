---
title: "Monotonic and Wall Clock Time"
seoTitle: "Monotonic and Wall Clock Time"
datePublished: Sun Mar 16 2025 09:42:35 GMT+0000 (Coordinated Universal Time)
cuid: cm8bg63zv000609jmav4te2zm
slug: monotonic-and-wall-clock-time
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/AMzC2RVurO4/upload/85f1d90a36312c07f4685706143a03b8.jpeg
tags: go, software-engineering

---

**Monotonic** and **wall clock** time are two ways we measure time.

Wall clock time is the time we see on our clocks. It's the time we are all used to and respond with when someone asks "what time it is?". Monotonic time on the other hand, is one that simply continues to increase since the computer started. When used in programming, it's the time that has passed from the point the timer was started. Monotonic time is mostly used to measure the time that has elapsed between two events.

Monotonic times have two characteristics:

1. They tick forward at a regular rate
    
2. They are detached from current time of day
    

The value you get from reading a monotonic clock is typically a long integer. It's usefulness is greatly seen when it's compared with other monotonic times.

The reason why we need to keep track of the two types of time is because, *wall clock time does not uniformly increase*. It can be affected by the user's manual adjustments to the clock, daylight saving time, and other factors like network time changes. Since it can be affected by these factors, it's not a good idea to use it to measure the time that has elapsed between two events.

Using wall clock time in programming can cause weird bugs where your programs might be thinking a certain duration of time has passed, when in reality, it's been longer or shorter. One major assumptions people make is that "Time difference cannot be negative". Going back to our reason why we need to keep track of the two types of time, we can see why this might be the case. Assuming we have two times, `t1` and `t2`, and we want to find the difference between them, we can do so by subtracting `t1` from `t2`. Should time `t1` be before time `t2`, the result will be negative, which breaks our assumption. Time `t1` can always be before time `t2` because we learnt that wall clock time can be affected by the user's manual adjustments to the clock, daylight saving time, and other factors like network time changes.

Time difference being negative actually caused a big issue one time for Cloudflare DNS.

> In 2017, Cloudflare experienced a significant outage during a leap second adjustment. Their RRDNS software compared timestamps before and after the leap second was inserted, resulting in negative time differences. This violated the code's assumptions and caused the software to crash repeatedly. You can read more about it [here](https://blog.cloudflare.com/how-and-why-the-leap-second-affected-cloudflare-dns/)

When dealing with time, you should never ever think that time difference cannot be negative, never think that time cannot go backwards. Having these ideas in your mind will help you build programs that are more robust and reliable. Wall clock time is suitable for displaying time to users and scheduling based on the time of day, but it's inappropriate for measuring elapsed time due to its susceptibility to adjustments. Monotonic time, with its steady forward progression, is the correct choice for duration measurements, timeouts, and performance profiling.