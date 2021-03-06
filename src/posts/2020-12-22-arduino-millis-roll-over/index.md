---
title: Arduino millis roll-over, and why it can ruin your project
description: In Arduino controllers, millis() rolls-over back around to zero after 49 days. How can you avoid the problems this causes?
date: 2020-12-22
tags:
- Electronics
- Coding
- Solution
eleventyExcludeFromCollections: false
---

So you've got an Arduino that flashes some lights, moves some motors, and reads some sensors. Chances are that you don't want it to do whatever it's doing as many times as it can as quickly as it can. You don't care what the temperature is every 0.001 seconds. 

![Work without wait](./work-without-wait.png)

So you add delays to things e.g. read the sensor every 5 minutes.

![Work with wait](./work-with-wait.png)

The easy way is to use `delay()` which puts things to sleep, but this is usually a bad idea because you can't do anything else while it's `delay()`-ing.

The next thing you try is to check if the desired amount of time has passed before doing the activity again.
```cpp
void readTemp() {
	If (millis() > timeToReadTemp) {
		// Enough time has passed since the temperature was last read
		// Read the temperature again
	}
}
```

`millis()` returns the number of milliseconds that the controller has been active for. If you plugged in your USB cable and then called `millis()` an exact second later, you'd get the number 1,000 which means the controller has been running for 1,000 milliseconds or 1 second.

## The problem
The problem is that `millis()` is an unsigned long which goes from 0 up to 4,294,967,295 milliseconds, or about 49 days. When the maximum number is reached (`0xFFFFFFFF`) and more time passes, it will roll-over back to 0 (`0x00000000`) and start again. It won't cause the Arduino to crash, lock-up, or anything like that, it'll just happen. 

![Looping numbers](./looping-numbers-2.jpg)

Remember what happened with the Warlord Ghandi bug in the game Civilizations? The leader Ghandi was already pretty peaceful. When extra bonuses and policies were applied, he would get more and more peaceful until it went below 0. However, the "peaceful" number was stored as an unsigned integer and couldn't go negative, so it rolled-over up to the other end of the scale and Gandhi became incredibly aggressive and warlike because of it.

![Warlike Gandhi](./Civ-1-Gandhi.jpg)

Roll-over becomes a problem with Arduino's when checking if the current time is after when you next want to do something. This results in the task never being run once `millis()` has rolled over, or once the controller has been on for more than 49 days. 

Let's work this out.

```
Interval = 100
Millis = 5000
NextRead = 4500
Millis - NextRead = 500 which is greater than the interval, run the task
```
This example shows the logic working correctly.

```
Interval = 100
Millis = 5
NextRead = 4000000000
Millis - NextRead = -3999999999 which is a negative number less than the interval, do not run the task
```

This example shows where the `LastRead` is still up around the end of the number scale, but `millis()` has rolled back around to the bottom end. This results in the task never being run once `millis()` has rolled over.

## The solution
The solution to this problem is to instead compare the **duration**, not the **timestamps**. Take the current time, subtract the last time the task was performed, and then check if that number is greater than the interval. Because of the way that the math is performed, the 2 timestamps roll-over together in the same way so which means that they're not out of sync.

The code for this is:
```cpp
unsigned long lastThink = 0;
unsigned long interval = 250; // in milliseconds
unsigned long now = millis();
if ( ( now - lastThink) >= interval) { 
	lastThink = now;
	doTheThing();
}
```

I like to use this logic at the start of the task functions themselves. Instead of wrapping the entire function in an `if` block, I reverse the logic and `return` out of the function if not enough time has passed.

```cpp
void doTheThing() {
	if ( millis() - lastThink < interval ) return; // Not enough time has passed, return
	
	// Do the thing
}
```
