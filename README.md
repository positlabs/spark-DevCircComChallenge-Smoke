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

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/Getting%20set%20up.png" alt="Patches - step 1" width="600"/>

With this you have all you need to get started, let's move on!

### Part 2 - _Creating the loop_

The basic idea thus is to modify the delay frame. In order to do this, we'll be making use of a Texture Transform patch, in tandem with a 2D Transform patch. These will allow us to modify the delay frames each time they pass through. From here we can't plug this into the delay frame, we'll need to go through a Shader Render Pass patch first, then output that to our Delay Frame patch, as shown below :

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/Texture%20Transform.png" alt="Patches - step 2" width="600"/>

What do we feed in the texture slot you might ask? 
We could just feed the delay frame in it using the receiver patch we set up earlier, however we wanted our effect to apply on the user. In other words, we don't want the effect to apply inside of the person segmentation, rather only on the background. Given that we also want it to draw into the delay frame, we'll make use of a Blend patch set to Normal, and overlay our person segmentation setup with the delay frame, as shown below :

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/Blend.png" alt="Patches - step 3" width="600"/>

At this point we can connect the output of our blend to the device output and finally see some preliminary results, as it's drawing with the transparency. We can change the background color by touching the color in the delay frame. It's also here that we see our initial loop, that starts from our cameraTexture based setup and goes into the texture transform. From there it enters the cycle, getting modified, sent into the delay frame, which in turn is fed back into our blend and sent through to cycle once more.

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/First%20Loop.png" alt="First Loop" width="600"/>

Modifying the values within the 2d transform pack will directly impact the effect. For example affecting the scale even by just 0,01 will make the frames scale off in a direction. For our purposes, we want the transformations to be centered on the user segmentation, so we'll also set Pivots to 0,5. You can play around with this to get wild results, from changing the values to using the facetracker position as a driving force, have fun!

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/Values.png" alt="Basic Results" width="400"/>

This is fundamentally what makes the smoke effect, from here we're going to use a technique called distortion, so every frame gets distorted a little bit more based on the previous distortion.

### Part 3 - _Texture Distortion Shader_

To get started on the distortion, we're going to need two things :
1. a noise texture
2. the Texture Distortion Shader from the spark AR library

A sample noise texture, and the one we'll be using in this example, can be found in the downloads section on this project's github page. Download it and drag it into your assets panel. Next, go into the SparkAR library and search for the Texture Distortion Shader. Once found, Import it into your project and drag it into your patch editor.

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/importDistortion.gif" alt="Import Texture Distortion Shader from Spark AR Patch Library" width="600"/>

This patch uses a grayscale texture to distort another texture. It can be useful as-is, but there are a few adjustments that can be made to make it more flexible. We’ll expose one value to control the strength of the distortion (it’s already there, just not exposed), and another to control the direction (by default, the distortion moves diagonally from the top left to bottom right). 

Click the “expand” or master link button to enter the patch group. You’ll see a find the multiply patch toward the beginning of the graph and click on the input. It will show a button that allows you to expose that input as a parameter on the parent group.

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/expandPatch.gif" alt="Import Texture Distortion Shader from Spark AR Patch Library" width="600"/>

This will be the strength parameter. You can name it appropriately, and set some default values and constraints by clicking the cog and selecting “edit properties”

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/editProperties.png" alt="Import Texture Distortion Shader from Spark AR Patch Library" width="500"/><img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/strengthParam.png" alt="Import Texture Distortion Shader from Spark AR Patch Library" width="500"/>
