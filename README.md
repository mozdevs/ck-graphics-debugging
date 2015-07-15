# Graphics debugging with Firefox DevTools

> Current as of July 2015

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





## Demoing: Things that are Broken

- [ ] Sites that do *not* use `requestAnimationFrame` are not intercepted yet
- [ ] Canvas in iframes (not in the current document) are not detected

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
