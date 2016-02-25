# Graphics debugging with Firefox DevTools

> Current as of February 2016

## Introduction

* Firefox DevTools has two tools for debugging and editing graphics content in place: the Canvas Inspector and the WebGL Shader editor
* They are specially useful for building games, infographics, visualisations, and any other type of content that uses the `<canvas>` element or WebGL APIs to display graphics in an efficient way, such as WebVR

## Key Points

1. The Canvas Inspector lets you intercept all the calls happening on a `Canvas` context, be it 2D (traditional "canvas" operations such as `fillRect`) or 3D (for WebGL content). It complements traditional step by step debugging with a filmstrip view where each frame is a snapshot of the canvas element per call, and a slider to be able to move back and forth to diagnose glitches or issues.
2. The WebGL Shader editor lists all the shaders (programs) in the inspected WebGL context, and also allows the developer to live edit or hide them, to help diagnosing underperformant or troublesome shaders.
3. Both need to be manually enabled in the DevTools settings menu--they are not enabled by default on a standard Firefox installation
4. They work best when the code uses `requestAnimationFrame` and *not* `setInterval` or `setTimeout`

## Presentation Setup and Materials

For __any presentation__:

- [ ] Firefox (Stable, Developer Edition or Nightly--all are fine, but starting with DevEdition the Shader Editor has a new and better icon) installed on the host computer
- [ ] the Canvas Inspector or WebGL Shader editor enabled in DevTools' settings (open DevTools - click on the 'cog' icon on the top right side, then under *Default Firefox Developer Tools* make sure the *Shader Editor* and *Canvas* entries are checked).

## Demoing / Script

[Launcher with 'tips and tricks'](https://mozdevs.github.io/ck-graphics-debugging/)

Let's visit [Hello Racer](http://helloracer.com/webgl/).

### Canvas inspector

1. Open DevTools.
2. Select the *Canvas* tab
3. You'll get a "Reload the page to be able to debug `<canvas>` contexts" message.
4. Click `Reload`.
5. When the site reloads and the canvas context is created anew, the calls to this one can be *captured* by DevTools by clicking on the little icon to the left of the `Import...` button.

Let's capture and debug a frame:

1. Press the capture button. It will take a little longer to render the frame than usual.
2. When it's done, you'll see a thumbnail labeled *Snapshot #1*, plus the number of draws and calls used to render the frame on the column on the left, and the full stack of drawing calls that happened on the canvas context.
3. We can go through the list of calls and replay how the frame was built, step by step. Click on the first call to select it, and notice how a black picture is rendered on the right--it's the initial state for the canvas!
4. We can scroll down and look at the calls that have been performed. Each time there's a *drawing* call (e.g. `gl.drawElements`) a new *frame* will be rendered on the film-strip like representation on the bottom area of the inspector.
5. We can also scroll up-to compare differences between frames.
6. Notice there's a slider on the top that lets us scroll faster and jump to random points in the call stack.

You can also double click on a function call in the stack to unfold the JavaScript stack. If you click on the line number, you'll be brought to the Debugger view where you can debug JS *the traditional way*.

At this point, you can of course do useful things such as beautify the code (to work with minified/compressed versions) by clicking the brackets icon on the bottom left side of the Debugger (`{}`), and maybe set a breakpoint on the *conflictive* piece of code, and the animation will stop.

Another thing you can do is export the recording to give it to another developer who can use it to reproduce the rendering--this is very useful when working on a team, and way better than trying to describe what you're seeing! After it's been exported, you can `Import...` it into the inspector even if you don't have access to the same website.


### WebGL Shader Editor

**NOTE**: If you set up a debugger breakpoint, make sure to delete or disable it, and resume execution, or just reload the site if things aren't *moving*.

1. Change to the *Shader editor* tab.
2. You will need to `Reload` the page for the context calls that load shader code to be intercepted.
3. You'll see a series of entries being added to the left side column, labeled `Program 0`, `Program 1`, and so on...
4. Each entry corresponds to a WebGL shader loaded by this context.

Let's select a shader by clicking on its entry:

1. Hovering an entry highlights it in red when rendering, so you can determine which program exactly you want to debug (useful when there are *many* programs as in this case). Try hovering over a few entries and notice how different parts of the scene are rendered red.
2. Clicking on the little eye icon lets you hide or *fake blackbox* programs to toggle their visibility (although what it actually does is hiding their geometry), so you can get things out of the way but still inspect or edit the rest of programs in place, or if you want to find out what is making your rendering slow you can start hiding things until you come up with the culprit. Try toggling a few programs and seeing the effect.
3. Clicking over the `Program` name selects the entry.
4. You'll see two panes on the right, corresponding to the *Vertex* and *Fragment* shaders respectively.
5. Both can be edited and you'll see the result immediately on the screen as the shader is live reloaded as you type.
6. But this site is using three.js so the shaders are programmatically generated and are full of `#ifdefs` that make demoing the editors a bit too complex, so let's go to a simpler example.

Let's navigate to the [3D objects](http://learningwebgl.com/lessons/lesson04/index.html) demo from the [Learning WebGL](http://learningwebgl.com) site.

1. Notice there is only one (very simple) shader in use now.
2. Let's edit the fragment shader so we can play with colours. Instead of having `gl_FragColor = vColor;` we can try some of these variations:

```c
// alpha channel at 0.5
gl_FragColor = vec4( vColor.x, vColor.y, vColor.z, 0.5);

// saturate the blue channel
gl_FragColor = vec4( vColor.x, vColor.y, 1.0, 1.0);

// invert the colours
gl_FragColor = vec4( 1.0 - vColor.x, 1.0 - vColor.y, 1.0 - vColor.z, 1.0);
```

3. Let's now edit the vertex shader. We can add a new line after line 11, where we modify the values of `gl_Position` before we return from the shader:

```c
// make everything narrower on the X axis
gl_Position.x *= 0.5;

// make it ... interesting
gl_Position.x -= 1.0*sin(gl_Position.y);
```

Now notice the declaration for the varying `vColor` that we later use in the fragment shader? We can play with its value too. Let's add this line after the `vColor = aVertexColor;` declaration:

```glsl
vColor.x = 0.5*sin(gl_Position.x);
```

this way we make the colour of each pixel depend on its x position as well, not just the standard interpolation between colours.

## Demoing: Things that are Broken

- [ ] Sites that do *not* use `requestAnimationFrame` are not intercepted yet
- [ ] Canvas in iframes (not in the current document) are not detected

## What's next for these tools?

* A better way to identify slowness or performance bottlenecks
* Consolidating everything in just one "game" tool, instead of different panels.

## Reference Materials

### Canvas inspector

* [Mozilla Hacks post, March 2014](https://hacks.mozilla.org/2014/03/introducing-the-canvas-debugger-in-firefox-developer-tools/)
* [Mailing list announcement](https://groups.google.com/forum/#!msg/mozilla.dev.developer-tools/sFbzSTRN5vw/AOtGO6FQ8X0J)

### WebGL Editor

* [Mozilla Hacks post, November 2013](https://hacks.mozilla.org/2013/11/live-editing-webgl-shaders-with-firefox-developer-tools/)
* [Video](https://www.youtube.com/watch?v=hnoKqFuJhu0)
* [Documentation in MDN](https://developer.mozilla.org/en-US/docs/Tools/Shader_Editor)

### Developer Tools:

* Docs: https://developer.mozilla.org/docs/Tools
* Wiki: https://wiki.mozilla.org/DevTools
* Feedback: https://ffdevtools.uservoice.com/ or http://mzl.la/devtools
