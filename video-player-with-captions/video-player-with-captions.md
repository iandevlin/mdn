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

The video controls undergo some minor changes in order to facilitate the extra button, but these are relatively straightforward.

No image is used for the captions button, so it is simply styled as:
```css
.controls button[data-state="captions"] {
	height:85%;
	text-indent:0;
	font-size:16px;
	font-size:1rem;
	font-weight:bold;
	color:#666;
	background:#000;
	-moz-border-radius:2px;
	-webkit-border-radius:2px;
	border-radius:2px;
}
```

There will also be other CSS changes that are specific to some extra JavaScript implementation, but these will be mentioned at the appropriate place below.

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
As you can see, we loop through each `textTracks` within the video and set its `mode` to `'hidden'`. We can does this via a handy API, which, similar to the Media API, the [WebVTT API](http://dev.w3.org/html5/webvtt/#api) gives us access to all the text tracks that are defined for a HTML5 video using the `<track>` element.

###Building a caption menu

Our aim, is to use the captions button we added earlier to display a menu to the user that allows them to choose which language they want the captions displayed in, or to turn them off entirely.

We have added the button, but before we make it do anything, we need to build the menu that goes with it. This menu is built dynamically, so that languages can be added or removed to/from it by simply editing the `<track>` elements within the video's markup.

All we need to do is to go through the video's `textTracks`, reading its properties and building the menu up from there:
```javascript
var captionsMenu;
if (video.textTracks) {
   var df = document.createDocumentFragment();
   var captionsMenu = df.appendChild(document.createElement('ul'));
   captionsMenu.className = 'captions-menu';
   captionsMenu.appendChild(createMenuItem('captions-off', '', 'Off'));
   for (var i = 0; i < video.textTracks.length; i++) {
      captionsMenu.appendChild(createMenuItem('captions-' + video.textTracks[i].language, video.textTracks[i].language,         video.textTracks[i].label));
   }
   videoContainer.appendChild(captionsMenu);
}
```
This code creates a `documentFragment` which is then used to hold an unordered list containing our captions menu. First of all button is added to allow the user to switch all captions off, and then buttons are added for each `textTracks`, reading the language and label from each one.

The building of each list item and button is contained within the `createMenuItem()` function which is defined as follows:
```javascript
var captionMenuButtons = [];
var createMenuItem = function(id, lang, label) {
   var listItem = document.createElement('li');
   var button = listItem.appendChild(document.createElement('button'));
   button.setAttribute('id', id);
   button.className = 'captions-button';
   if (lang.length > 0) button.setAttribute('lang', lang);
   button.value = label;
   button.setAttribute('data-state', 'inactive');
   button.appendChild(document.createTextNode(label));
   button.addEventListener('click', function(e) {
      // Set all buttons to inactive
      captionMenuButtons.map(function(v, i, a) {
         captionMenuButtons[i].setAttribute('data-state', 'inactive');
      });
      // Find the language to activate
      var lang = this.getAttribute('lang');
      for (var i = 0; i < video.textTracks.length; i++) {
         // For the 'captions-off' button, the first condition will never match so all will captions be turned off
         if (video.textTracks[i].language == lang) {
            video.textTracks[i].mode = 'showing';
            this.setAttribute('data-state', 'active');
         }
         else {
            video.textTracks[i].mode = 'hidden';
         }
      }
      captionsMenu.style.display = 'none';
   });
   captionMenuButtons.push(button);
   return listItem;
}
```
This function builds the required `<li>` and `<button>` and returns them to be added to the captions menu list. It also sets up the required event listeners on the button which actually toggles the relevant caption set on or off. This is done by simply setting the required caption to `showing` via the `mode` attribute and setting the others to `hidden`.

Once the menu is built, it is then inserted into the DOM at the bottom of the `videoContainer` (see code snippet above).

Initially the menu is hidden by default, so an event listener needs to be added to our captions button to toggle it:
```javascript
captions.addEventListener('click', function(e) {
   if (captionsMenu) {
      captionsMenu.style.display = (captionsMenu.style.display == 'block' ? 'none' : 'block');
   }
});
```

###CSS

We also add some rudimentary styling for the newly created captions menu is also required:
```css
captions-menu {
	display:none;
	position:absolute;
	bottom:14.8%;
	right:20px;
	background:#666;
	list-style-type:none;
	margin:0;
	padding:0;
	width:100px;
	padding:10px;
}
.captions-menu li {
	padding:0;
	text-align:center;
}
.captions-menu li button {
	border:none;
	background:#000;
	color:#fff;
	cursor:pointer;
	width:90%;
	padding:2px 5px;
	-moz-border-radius:2px;
	-webkit-border-radius:2px;
	border-radius:2px;
}
```

That's pretty much all that's required to get the captions working on most browsers. Most.

##Browser Compatibility

Browser support for [WebVTT and the `<track>` element](http://caniuse.com/webvtt) is fairly good, although some browsers differ slightly in their implementation.

###Internet Explorer 11
Captions are enabled by default, and the default controls contains a button and a menu that offers the same functionality as the menu we just built. The `default` attribute is honoured.

NOTE: IE will completely ignore WebVTT files unless you setup the MIME type. This can easily be done by adding a `.htaccess` file to the appropriate directory that contains: `AddType text/vtt .vtt`.

###Chrome and Opera
Since they both run on WebKit, these browsers have similar implementations of video captions. Captions are enabled by default, and the default control set contains a 'cc' button which simply turns captions on and off. It ignores the `default` attribute on the `<track>` element and will try to match the browser's language to the caption's language.

###Safari

TODO!

###Firefox
Firefox's implementation is completely broken due to a [bug](https://bugzilla.mozilla.org/show_bug.cgi?id=981280), and this has led Mozilla to take the decision to turn off WebVTT support by default but you can turn it on via the `media.webvtt.enabled` flag.

Once you have enabled WebVTT, if you only provide one set of captions, Firefox works ok, but there is no way to turn them off. If you provide more than one set of captions, it just displays them on top of one another, and none of the JavaScript magic mentioned above has any effect, it simply ignores any manual setting of the `textTracks` `mode` attribute.

##Plugins

