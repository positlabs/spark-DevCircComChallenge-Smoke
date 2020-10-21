## _How to create a smoke effect with render passes in SparkAR_

### Introduction

This tutorial was made as an entry for the 2020 Developer Circles Community Challenge hosted by Facebook.
We'll be showing you how to achieve a smokey shader effect to be applied on any scene item thanks SparkAR's Render Pass feature as well as a slightly modified patch from the SparkAR library.

### Before you start

This is everything you will need to get started :
1. A running version of SparkAR v90+ (latest can be downloaded from : [SparkAR](https://sparkar.facebook.com/ar-studio/download/))
2. Any noise texture (one can be found in the downloads of this project)
That's it!

### Part 1 - _Getting set up_

The idea behind this project is creating a feedback loop effect with the Render Pass feature that we'll use to apply distortion on multiple passes. We'll be using delay frames and modifying them on each successive run through said loop. Each iteration will affect the frame, giving it a continuous effect.

The first thing we'll want to do is start from a blank template on SparkAR. From here, we'll bring in the initial setup and briefly go over why we're bringing these in.
We'll start by bringing in the device output patch, not the default render pass pipeline :

![alt text](https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/Device%20output.gif "Device output")

It's important to keep in mind that this is the end of the chain, where you'll plug in the result of all the previous operations in the patch editor. 
To complement this we'll be adding in a Delay Frame patch as well as a receiver linked to it. Finally, in this first iteration of the effect we want to apply it to our user, so we'll need to use the person segmentation feature. Select the camera in your Scene tab and navigate to the right :

![alt text](https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/Extract%20Textures.gif "Device output")

This will have brought in both cameraTexture0 and personSegmentationMaskTexture0 in your assets panel, drag them in the patch editor and plug them both into a Pack patch like shown below :

![alt text](https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/Getting%20set%20up.png "Patches - step 1")

With this you have all you need to get started, let's move on!

### Part 2 - _Creating the loop_

The basic idea thus is to modify the delay frame. In order to do this, we'll be making use of a Texture Transform patch, in tandem with a 2D Transform patch. These will allow us to modify the delay frames each time they pass through. From here we can't plug this into the delay frame, we'll need to go through a Shader Render Pass patch first, then output that to our Delay Frame patch, as shown below :

![alt text](https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/Texture%20Transform.png "Patches - step 2")