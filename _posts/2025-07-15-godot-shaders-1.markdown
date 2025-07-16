---
layout: post
title:  "Godot Shaders 1"
date:   2025-07-15 17:52:02 -0400
categories: godot shaders game development
---
I've been learning about the game engine [Godot](https://godotengine.org/) lately. It's considered to be a leading freely available open source engine. In the spirit of the [feynman technique](https://fs.blog/feynman-technique/),I find it helpful to write explanations about the things I'm learning. What follows is a problem driven introduction to programming shaders in Godot. It assumes a mild acquaintance with game engines. 

#### Problem 1: Write a shader that fades a *Sprite2D* to zero transparency.

It should be said upfront that this is not the best method for achieving this effect. Creating an animation in the *AnimationPlayer* that creates a tween on the modulate:a property is much more user friendly than writing a shader. 

That being said, let's explore how to solve this problem with a shader.

I have in my new Godot 4.4 project a scene with *Node2D* as its root. I have dragged the Godot icon.svg into the 2D window, making it a *Sprite2D* child of the root *Node2D*. Incidentally, I also have an *AnimationPlayer* with a fade out animation in the scene tree. 

![Workspace 1](/assets/shaders_1/1_workspace_screenshot.JPG)

When I select the *Sprite2D* Icon, the inspector panel on the left shows its properties. The properties are subdivided by the classes *Sprite2D* inherits--as *Sprite2D* is a class itself. The screen shot above shows properties for *Node2D* and *CanvasItem* because *Sprite2D* inherits them. What is not shown in the screenshot is the *Material* property of *CanvasItem*. This is where we will begin using a shader to solve our problem. 

First, we assign a new *ShaderMaterial* to the *Material* property. Then we add a new shader to that material. It opens up a dialog box called "Create Shader." We use it to create a new shader file that ends in *.gdshader*. I will name mine fade_out_1.gdshader. 

Having created the new shader file and assigned it to the new shader material of the 2D sprite we are working with, the shader editor pops up in the bottom panel. 

![Workspace 2](/assets/shaders_1/2_workspace_screenshot.JPG)

There's only one shader function I'm interested in: *fragment()*. It is called on every pixel the material is visible on, which is the entirety of the sprite. 

There are [several functions and data](https://docs.godotengine.org/en/4.4/tutorials/shaders/shader_reference/canvas_item_shader.html#doc-canvas-item-shader) that are accessible within the fragment shader function. We will need to use *TIME* and *COLOR* to solve our problem.

Suppose I write the following code in the *fade_out_1.gdshader*:

```C++
void fragment() {
	// Called for every pixel the material is visible on.
	COLOR.a -= 0.5;
}
```

It will immediately fade the Icon in the Godot 2D preview, but it's static. Before we animate a fade, let's discuss what this code does. *Color.a* references the alpha channel for the pixel it is called on--remember this function is being called on every pixel of the sprite. It simply takes the color of the sprite texture and renders it as the same color, but with half the opacity. If we changed 0.5 to 1.0, the sprite would disappear from the 2D preview window because we have written a shader to render every pixel of the sprite texture with full transparency. 

Our problem calls for a *fade*, which is an animation. Animations are functions of time. *TIME* is exposed for our use in the *fragment* function. We will use it to create the fade. *TIME* is a global variable that keeps track, in seconds, of how long the engine has been running. It repeats after every 3,600 seconds, but this doesn't matter for our purpose.

Suppose we would like our fade to last *x* seconds. This means we will reduce *Color.a* from 1 to 0 in *x* seconds. This can be achieved with the following code:

```C++
void fragment() {
	// Called for every pixel the material is visible on.
	COLOR.a = 1 - TIME/x;
}
```

Where *x* is simply a placeholder for the duration of our fade. If we replaced *x* with 3.0, the fade would last three seconds. Note: since TIME resets after an hour, this isn't a perfect solution, as the sprite would magically appear again after an hour. We may explore a better solution later, but first let's unpack this code. 

If *TIME* < x, *TIME/x* is less than one. As TIME increases, it approaches X. As TIME/x approaches 1, 1-TIME/x approaches 0. If *TIME > x* then COLOR.a < 0. If COLOR.a < 0, the pixel has a negative alpha chanel value, which means it is transparent. 

When x is a constant, 1 - Time/x is a linear function. This means *Color.a* decreases at a constant rate.  


[test](/assets/exp/learning_shaders.html)