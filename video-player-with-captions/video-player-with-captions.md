#Video Player with Captions

Previously we looked at how to [build a cross browser video player](https://developer.mozilla.org/en-US/Apps/Build/Manipulating_media/cross_browser_video_player) using the Media and Fullscreen APIs, and also at how to [style the player](https://developer.mozilla.org/en-US/Apps/Build/Manipulating_media/Video_player_styling_basics). This article will take the same player and look at how captions and/or subtitles could be added.

##The example in action

ENTER SCREENSHOT HERE

You can find the code for the [video player with captions](https://github.com/iandevlin/iandevlin.github.io/tree/master/mdn/video-player-with-captions) on GitHub, and you can also [view it live](http://iandevlin.github.io/mdn/video-player-with-captions/).

##HTML5 and Video Captions

Before diving into how to add captions to the video player, there are a number of things that need to be cleared up and mentioned before use.

First of all, [captions and subtitles are not the same thing](http://screenfont.ca/learn/), but for this article we will refer to them as captions, as the content of the files that are used have more in common with that type of content.

###The `<track>` element
HTML5 allows us to specify captions for a video by way of the [`<track>` element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/track). The various attributes in this element allow us to specify such things as the type of content that we're adding, the language it's in and of course a reference to the text file that contains the actual caption information.

###WebVTT
The text files that contain the actual caption data are simple text files that follow a specified format, in this case the Web Video Text Tracks or [WebVTT format.](https://developer.mozilla.org/en-US/docs/HTML/WebVTT) The [WebVTT specification](http://dev.w3.org/html5/webvtt/) is still being worked on, but major parts of it are stable so we can use it today.

Video providers (such as the [Blender Foundation](http://www.blender.org/foundation/)) provide caption or subtitles in a text format with their videos, but they're usually in the SRT (SubRip Text) format. These can be easily converted to WebVTT using an online converter such as [srt2vtt](https://atelier.u-sub.net/srt2vtt/).

##Modifications

This section summarises the modifications made to the previous article's code in order to facilitate the addition of subtitles to the video.

In this example we are using a different video, [Sintel](http://www.sintel.org/), to the one used in the previous articles as it actually has some speech in it and therefore is better for illustrating how captions work!

###HTML Markup

As mentioned above, we need to make use of the new HTML5 `<track>` element to add our caption files to the HTML5 video. We actually have our captions in three different languages, English, German, and Spanish, so we will add all three of these to our HTML5 `<video>` element:

```html
<video id="video" controls preload="metadata">
   <source src="video/sintel-short.mp4" type="video/mp4">
   <source src="video/sintel-short.webm" type="video/webm">
   <track label="English" kind="captions" srclang="en" src="captions/vtt/sintel-en.vtt" default>
   <track label="Deutsch" kind="captions" srclang="de" src="captions/vtt/sintel-de.vtt">
   <track label="Español" kind="captions" srclang="es" src="captions/vtt/sintel-es.vtt">
</video>
```

As you can see, each caption file has its own `<track>` element with the following attributes set:
* `kind` is set to `'captions'` indicating the type of content the files contain
* `label` is set to something relevant such as 'English' or 'Deutsch'
* `src` is assigned a valid URL to the relevant WebVTT captions file
* `srclang` indicates what language the captions file contents are in
* `default` is set on the English `<track>` element, indicating that this is the default caption file definition to use

In addition to adding the `<track>` elements, we also add a new button to control the captions menu that we will build. As a consequence, the video controls now looks as follows:

```html
<div id="video-controls" class="controls" data-state="hidden">
   <button id="playpause" type="button" data-state="play">Play/Pause</button>
   <button id="stop" type="button" data-state="stop">Stop</button>
   <div class="progress">
      <progress id="progress" value="0" min="0">
         <span id="progress-bar"></span>
      </progress>
   </div>
   <button id="mute" type="button" data-state="mute">Mute/Unmute</button>
   <button id="volinc" type="button" data-state="volup">Vol+</button>
   <button id="voldec" type="button" data-state="voldown">Vol-</button>
   <button id="fs" type="button" data-state="go-fullscreen">Fullscreen</button>
   <button id="captions" type="button" data-state="captions">CC</button>
</div>
```

###CSS Changes

The video controls undergo some minor changes in order to facilitate the extra button, but these are relatively straightforward. There will also be other CSS changes that are specific to some extra JavaScript implementation, but these will be mentioned at the appropriate place below.

##JavaScript Implementation

A lot of what we will do to access the video captions revolves around JavaScript. Similar to the video controls, if a browser supports HTML5 video captions, there will be a native button within the control set but since we have defined our own video control set, this button is hidden.

Browsers do vary as to what they support, so we will be attempting to bring a more unified UI to each browser where possible. There's more on browser compatibility and issues later on.

As with all the other buttons, one of the first things we need to do is to store a handle to the captions button:
```javascript
var captions = document.getElementById('captions');
```
We also initially turn off all captions, in case the browser turns any of them on by default:
```javascript
for (var i = 0; i < video.textTracks.length; i++) {
   video.textTracks[i].mode = 'hidden';
}
```
As you can see, we loop through each `textTrack` within the video and set its `mode` to `'hidden'`. Wait, `textTrack`? `mode`? Similar to the Media API, there is also a (WebVTT API)[http://dev.w3.org/html5/webvtt/#api] which gives us access to all the text tracks that are defined for a HTML5 video using the `<track>` element.

