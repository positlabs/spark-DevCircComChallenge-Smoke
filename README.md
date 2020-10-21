# _How to create a smoke effect with render passes in SparkAR_

## **Introduction**

This tutorial was made an as entry for the 2020 Developer Circles Community Challenge hosted by Facebook.
We'll be showing you how to achieve a smokey shader effect to be applied on any scene item thanks SparkAR's Render Pass feature as well as a slightly modified patch from the SparkAR library.

## **Before you start**

This is everything you will need to get started :
1. A running version of SparkAR v90+ (latest can be downloaded from : [SparkAR](https://sparkar.facebook.com/ar-studio/download/)
2. Any noise texture (one can be found in the downloads of this project)
That's it!

## **Part 1** - Getting set up

The first thing we'll want to do is start from a blank template on SparkAR. From here, we'll bring in the initial setup and briefly go over why we're bringing these in.
We'll start by bringing in the device output patch, not the default render pass pipeline
![alt text](https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/Getting%20set%20up.png "Patches - step 1")

The idea behind this project is creating a feedback loop effect with the Render Pass feature. We'll be using delay frames and modifying them on each successive run through said loop. Each iteration will affect the frame, giving it a continuous effect.