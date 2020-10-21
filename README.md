## _How to create a smoke effect with render passes in SparkAR_

### Introduction

This tutorial was made as an entry for the [2020 Developer Circles Community Challenge](https://developercircles2020.devpost.com/) hosted by Facebook.

We'll be showing you how to achieve a smokey shader effect to be applied on any scene item thanks to SparkAR's Render Pass feature as well as a slightly modified patch from the SparkAR library.

_This is what we're building :_
<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/demo.gif" width="300"/>


### Before you start

This is everything you will need to get started :
1. A running version of SparkAR v90+ (latest can be downloaded from : [SparkAR](https://sparkar.facebook.com/ar-studio/download/))
2. Any noise texture (one can be found in the downloads of this project)

That's it!

### Part 1 - _Getting set up_

The idea behind this project is creating a feedback loop effect with the Render Pass feature that we'll use to apply distortion on multiple passes. We'll be using delay frames and modifying them on each successive run through said loop. Each iteration will affect the frame, giving it a continuous effect.

The first thing we'll want to do is start from a blank template on SparkAR. From here, we'll set up a few patches and briefly go over why them specifically.
We'll start by bringing in the device output patch, not the default render pass pipeline :

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/Device%20output.gif" width="250"/>

It's important to keep in mind that this is the end of the chain, where you'll plug in the result of all the previous operations in the patch editor. 
To complement this we'll be adding in a Delay Frame patch as well as a receiver linked to it. Finally, in this first iteration of the effect we want to apply it to our user, so we'll need to use the person segmentation feature. Select the camera in your Scene tab and navigate to the right :

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/Extract%20Textures.gif" width="250"/>

This will have brought in both cameraTexture0 and personSegmentationMaskTexture0 in your assets panel, drag them in the patch editor and plug them both into a Pack patch like shown below :

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/Getting%20set%20up.png" width="500"/>

With this you have all you need to get started, let's move on!

### Part 2 - _Creating the loop_

The basic idea thus is to modify the delay frame. In order to do this, we'll be making use of a Texture Transform patch, in tandem with a 2D Transform patch. These will allow us to modify the delay frames each time they pass through. From here we can't plug this into the delay frame, we'll need to go through a Shader Render Pass patch first, then output that to our Delay Frame patch, as shown below :

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/Texture%20Transform.png" width="500"/>

What do we feed in the texture slot you might ask? 
We could just feed the delay frame in it using the receiver patch we set up earlier, however we wanted our effect to apply on the user. In other words, we don't want the effect to apply inside of the person segmentation, rather only on the background. Given that we also want it to draw into the delay frame, we'll make use of a Blend patch set to Normal, and overlay our person segmentation setup with the delay frame, as shown below :

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/Blend.png" width="500"/>

At this point we can connect the output of our blend to the device output and finally see some preliminary results, as it's drawing with the transparency. We can change the background color by touching the color in the delay frame. It's also here that we see our initial loop, that starts from our cameraTexture based setup and goes into the texture transform. From there it enters the cycle, getting modified, sent into the delay frame, which in turn is fed back into our blend and sent through to cycle ad infinitum.

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/First%20Loop.png" width="600"/>

Modifying the values within the 2d transform pack will directly impact the effect. For example affecting the scale even by just 0,01 will make the frames scale off in a direction. For our purposes, we want the transformations to be centered, so we'll also set Pivots to 0,5 (they are relative to the screen). You can play around with this to get wild results, from changing the values to using the facetracker position as a driving force, have fun!

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/Values.png" width="400"/>

This is fundamentally what makes the smoke effect, from here we're going to use a technique called distortion, so every frame gets distorted a little bit more based on the previous distortion.

### Part 3 - _Texture Distortion Shader_

To get started on the distortion, we're going to need two things :
1. a noise texture
2. the Texture Distortion Shader from the spark AR library

A sample noise texture, and the one we'll be using in this example, can be found in the downloads section on this project's github page. Download it and drag it into your assets panel. Next, go into the SparkAR library and search for the Texture Distortion Shader. Once found, Import it into your project and drag it into your patch editor.

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/importDistortion.gif" width="500"/>

This patch uses a grayscale texture to distort another texture. It can be useful as-is, but there are a few adjustments that can be made to make it more flexible. We’ll expose one value to control the strength of the distortion (it’s already there, just not exposed), and another to control the direction (by default, the distortion moves diagonally from the top left to bottom right). 

Click the “expand” or master link button to enter the patch group. You’ll see a find the multiply patch toward the beginning of the graph and click on the input. It will show a button that allows you to expose that input as a parameter on the parent group.

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/expandPatch.gif" width="500"/>

This will be the strength parameter. You can name it appropriately, and set some default values and constraints by clicking the cog and selecting “edit properties”

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/editProperties.png" width="300"/><img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/strengthParam.png" width="300"/>

This strength value is affecting the brightness of the distortion texture, so anything less than 1 will be darkening and anything greater than 1 will be brightening. The pixel brightness of the distortion texture is used to determine how far to move the pixels in the main texture. 

_The pictures below use the final version of the effect so it's easier to understand_

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/strengthEffect.png" width="500"/>

The distortion texture gets sampled and swizzled into a vector 2, but because this is a grayscale image, the x and y values are both the same, meaning the distortion will always be in a diagonal direction (e.g. one pixel right, one pixel down). We can easily change the direction of the vector by multiplying it by another vec2. 

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/rotationEffect.gif" width="500"/>

Now you can expose that value as a parameter called “direction”! Your new super-powered texture distortion patch should look like this: 

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/superDistortion.png" width="300"/>

### Part 4 - _Smoke?_

To set the records straight, this is what you should be seeing in your preview if you've followed accordingly and set the strength parameter to 0,02 as well as the direction to (0,1).

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/preview.gif" width="300"/>

We now have a nice flowy movement that goes upwards. This however is not smoke, as it not only isn't the right color, but it's flowing to infinity and beyond, reaching all the way to the borders of the screen.

We'll tackle the issue of limiting it's reach first. A simple way to do this is to place a Multiply patch in between the delay frame Receiver and our newly-built Super Texture Distortion Shader. Given that the receiver is transmitting a texture, we can easily affect it's opacity by setting the multiplier to anything below 1.

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/alpha.png" width="500"/>

Now that we've succesfully limited the reach of the smoke, let's quickly make the background more visible, partly because smoke will stand out more on a black background, partly because we've done this set-up before, and without this little addition things are hard to see...

Add a Blend patch between your newly added Multiply patch and the STDP (your super-powered patch) like so :

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/backgroundChanged.png" width="500"/>

This also adresses an issue we've kept silent about that you may have noticed : you haven't been able to change the background color for a while... This blend patch will mainly allow us to do just that!

Now that we have the background color set, we need to attack the actual smoke's color, which ideally we want to be white. The trick here is rethinking what we initially feed into the loop. As you can see in the preview, we currently have the segmentation in use, however showing the cameraTexture through. We'd want the same, but filled in with a flat color. You guessed it (or not), that's exactly what the personSegmentationMaskTexture0's Alpha channel is, a white person segmentation mask. So we'll feed that in!

Concretely, bring the Blend patch connected to the Device Output upwards and add another one underneath. This new patch will take the Alpha output from the personSegmentationMaskTexture patch as a source, and, since we want it in the loop, will take your Super Texture Distortion Shader as destination input. This way the alpha channel is thrown into the delay frame, distorted, cycled back etc...

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/newBlend.png" width="500"/>

Finally, we'll quickly just smoothen our results by retouching the person segmentation alpha a tiny bit before we plug it in the rest. A quick swizzle set on 111x will allow us to focus on getting the alpha information without bothering about any RGB changes.

This leads us to the final project setup, featuring a nice smokin' Dolapo. The effect can of course be applied to a variety of things as well as tweaked and tuned to stray very far from our present use case, as we'll demonstrate with a few examples.

<img src="https://github.com/The-AR-Company/spark-DevCircComChallenge-Smoke/blob/main/images/finalSetup.png" width="500"/>

### What you've learned
