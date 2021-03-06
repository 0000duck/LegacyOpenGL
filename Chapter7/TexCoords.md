# Texture Coordinates

Now that we can load and unload textures it's finally time to texture something! In order to apply a texture to a 3D model, you must first specify some texture coordinates for each vertex of that model. So far we've given a vertex a position, a color, a normal; now we add texture coordinates to the list of attributes a vertex can have.

Texture coordinates have some special vocabulary. We call the texture coordinates __UV coordinates__. This is because a texture is not read on an X-Y axis like you might expect, rather a texture is read using a U-V axis. The U axis is horizontal, the V axis is vertical. 0, 0 is in the bottom left. When we use subscript notation, U comes first: (U, V)

The other thing to know about U-V space is that it is normalized. It doesn't matter if a texture is 124x124, 256x256, 512x1024 or whatever. The U-V coordinate space always extends from 0 to 1. The center pixel of an image is always at point (0.5f, 0.5f).

In this image, the left side shows a png picture, the right side shows the picture being applyed onto a triangle. The triangle is also outlined on top of the image. The UV coordinates of this triangles vertices are (0, 0), (1, 1), (1, 0). 

![UV](gl_uv.png)

What happens when your texture isn't a square? Let's say you have 512x1024. Then the UV coordinate (0.5f, 0.5f) maps to the pixel at  256x512. I can't stress this enough, __images should always be square and a power of two__. This will help keep coordinates simple, and will run faster on your graphics card.

While we're talking about textures, let's review why a power of two textue is faster on a graphics card. A U-V coordinate doesn't map to a pixel, it maps to a __texel__. As such, each pixel on an image might take up one or more pixels on the actual render target, or no pixels at all. In order to convert texels to pixels, a graphics card must divide by 2.

If you remember WAY back to when we started programming, we briefley talked about bitshifting. Let's take the binary number for 4 for example: ```0000 0100```. If we right shift this by 1 ```>> 1``` the result is two: ```000 0010```. We just achived the same thing as a division by two, except this is about 500 times less expensive than division! This trick only works with powers of two.

## Tex Coords

Alright, enough theory. You can specify texture coordinates for vertices using the ```GL.TexCoord2``` function. This function has the following signature:

```
GL.TexCoord2(float s, float t);
```

Where S is the U axis and T is the V axis. So, if we wanted for example to draw a textured quad on screen, we would specify UV coordinates for each vertex, like so:

```
GL.Begin(PrimitiveType.Quads);
    GL.TexCoord2(0, 1);             // What part of the texture to draw
    GL.Vertex3(left, bottom, 0.0f); // Where on screen to draw it
    
    GL.TexCoord2(1, 1);             // What part of the texture to draw
    GL.Vertex3(right, bottom, 0.0f);// Where on screen to draw it
    
    GL.TexCoord2(1, 0);             // What part of the texture to draw
    GL.Vertex3(right, top, 0.0f);   // Where on screen to draw it
    
    GL.TexCoord2(0, 0);             // What part of the texture to draw
    GL.Vertex3(left, top, 0.0f);    // Where on screen to draw it
GL.End();
```

Notice how i define the UV coordinate before the vertex. You specify whatever attributes a vertex needs, you could even specify all the attributes we've used until now for a single vertex!

```
GL.Begin(PrimitiveType.Triangles);
    GL.TexCoord2(1f, 1f);
    GL.Normal3(0f, 1f, 0f);
    GL.Color3(0f, 1f, 0f);
    GL.Vertex3(0f, 1f, 0f);
    
    GL.TexCoord2(1f, 0f);
    GL.Normal3(0f, 1f, 0f);
    GL.Color3(1f, 0f, 0f);
    GL.Vertex3(1f, 0f, 0.0f);
    
    GL.TexCoord2(0f, 0f);
    GL.Normal3(0f, 1f, 0f);
    GL.Color3(0f, 0f, 1f)
    GL.Vertex3(-1f, 0f, 0f);
GL.End();
```

The order in which you define the vertex attributes does not matter so long as A) you are consistent within the Begin / End calls and B) the ```GL.Vertex3``` call specifying position comes last. 

Most of the time when you specify a texture you will not specify a color. But position and normal will usually be specified for all 3D models. UI and 2D games tend to only use UV-Coords and position.

## So far
It's not enough to just specify texture coordinates. Texturing must be enabled, and a valid texture must be bound. Let's see some code as to what it takes to draw a textured quad:

```
int textureHandle = -1;
int textureWidth = -1;
int textureHeight = -1;

void Initialize() {
    // Enable Texturing
    GL.Enable(EnableCap.Texture2D);
    
    // Take note, we store the width and height!
    textureHandle = LoadGLTexture("file.png", out textureWidth, out textureHeight, true);
}

void Render() {
    // World coordinates of the quad we are drawing
    float left = 0f;
    float right = 20f;
    float top = 0f;
    float bottom = 10f;
    
    // We have to bind a valid texture to draw
    GL.BindTexture(TextureTarget.Texture2D, textureHandle);
    
    GL.Begin(PrimitiveType.Quads);
        GL.TexCoord2(0, 1);
        GL.Vertex3(left, bottom, 0.0f);
        
        GL.TexCoord2(1, 1); 
        GL.Vertex3(right, bottom, 0.0f);
        
        GL.TexCoord2(1, 0); 
        GL.Vertex3(right, top, 0.0f); 
        
        GL.TexCoord2(0, 0); 
        GL.Vertex3(left, top, 0.0f);
    GL.End();
    
    // You don't have to do this, but i don't like leaving bound textures
    GL.BindTexture(TextureTarget.Texture2D, 0);
}

void Shutdown() {
    // Now that we are done, delete the texture handles!
    GL.DeleteTexture(textureHandle);
    
    // Invalidate references
    textureHandle = -1;
}
```

That's a lot of code, and it's even more when you consider that all our texture loading code is nicley wrapped up in the ```LoadGLTexture``` function, not shown here. But that's what it takes to draw a textured primitive.

So, let's pretend that the quad we drew is a health bar. And the health bar is on an atlas. Our artist tells us that the UV-Coordinates for this health bar are at top-left:125x34, bottom-right(150, 50). How do we take those pixel coordinates and convert them into UV-coords? 

By __normalizing__ them. That is, divide the X pixel by the width or the imge, and the Y pixel by the height of the image. This will put the pixel coordinates into a normal space. Let's take a look at how we might change the render function to do this:

```
void Render() {
    // Normalized uv coordinates, remember we kept a reference
    // to the width and height of the image, returned 
    // from the LoadGLTexture function.
    float uv_left = 125f / textureWidth;
    float uv_right = 150f / textureWidth;
    float uv_top = 34f / textureHeight;
    float uv_bottom = 50f / textureHeight;
    
    // World coordinates of the quad we are drawing
    float left = 0f;
    float right = 20f;
    float top = 0f;
    float bottom = 10f;
    
    // We have to bind a valid texture to draw
    GL.BindTexture(TextureTarget.Texture2D, textureHandle);
    
    GL.Begin(PrimitiveType.Quads);
        GL.TexCoord2(uv_left, uv_bottom);
        GL.Vertex3(left, bottom, 0f);
        
        GL.TexCoord2(uv_right, uv_bottom); 
        GL.Vertex3(right, bottom, 0f);
        
        GL.TexCoord2(uv_right, uv_top); 
        GL.Vertex3(right, top, 0f); 
        
        GL.TexCoord2(uv_left, uv_top); 
        GL.Vertex3(left, top, 0f);
    GL.End();
    
    // You don't have to do this, but i don't like leaving bound textures
    GL.BindTexture(TextureTarget.Texture2D, 0);
}
```

You will notice in my code that i map the Y axis of my UV-Coordinates in reverse. This is because having (0, 0) on the bottom left makes no sense to me. So i map the UV's in reverse, effectivley putting (0, 0) at the top left of uv-space.

One trick i like to use to avoid dividing twice is reciprical multiplication. This code:

```
float uv_top = 34f / textureHeight;
float uv_bottom = 50f / textureHeight;
```

can be written as

```
float inverseHeight = 1f / textureHeight;
float uv_top = 34f * inverseHeight;
float uv_bottom = 50f * inverseHeight;
```

The final values of ```uv_top``` and ```uv_bottom``` will be the same, but instead of doing two divisions we now do 1 division and two multiplications. Which is actually faster. Not by a lot, but still faster.

## What's next
Before we move on to the "putting it all together" section where we actually write code i want you to take a peek into the 2DOpenTKFramework we've been using to make 2D games. Specifically, i want you to check out the [TextureManager.cs](https://github.com/gszauer/2DOpenTKFramework/blob/master/2DFramework/Framework/TextureManager.cs) file, it's the one with all the texture goodies. 

There is a specific function in there, it takes a texture-id and a screen rectangle. It draws the texture in it's entirety to the screen. The signature of this function is:

```
public void Draw(int textureId, Point screenPosition)
```

It's heavily commented, it should be easy to see how it works. If you have a pretty good grasp on how that function works, and feel up to a challenge, take a look at it's cousin with the overrides:

```
public void Draw(int textureId, Point screenPosition, PointF scale, Rectangle sourceSection)
```

This function does a bunch of math (mostly multiplication) to convert pixel coordinates on the source texture into normalized texture coordinates.