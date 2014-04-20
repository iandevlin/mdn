#Cross Browser Video Player Styling
In a [previous article](https://developer.mozilla.org/en-US/Apps/Build/Manipulating_media/cross_browser_video_player) I described how to build a cross-browser HTML5 video player using the Media and Fullscreen APIs. This follow-up article describes how to style this custom player, including making it responsive.

##Differences
###HTML Markup
In order to make the styling task easier, there are a number of changes that will be made to the HTML markup shown in the previous article. The change surrounds the custom video controls themselves which, instead of remaining as an unordered list, are now contained withing a `div`, as is the `progress` element.

The markup for the custom controls now looks as follows:

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
</div>
```

The previous article simply set the `display` property of the video controls to `block` in order to display them. This has now been changed to use a [`data-state`](http://toddmotto.com/stop-toggling-classes-with-js-use-behaviour-driven-dom-manipulation-with-data-states/) variable, which this code already uses in relation to its [fullscreen implementation](https://developer.mozilla.org/en-US/Apps/Build/Manipulating_media/cross_browser_video_player#Fullscreen).

This `data-state` idea is also used for setting the current state of buttons within the video control set which allows specific state styling.

###JavaScript
There will be some changes to the JavaScript, as mentioned above, a `data-state` variable is used in various places for styling purposes and these are set with JavaScript. Any other changes will be mentioned at the appropriate place.

##Styling
The resultant video player styled used here is rather basic, which is intentional, as the purpose is to show how such a video player could be styled and be made responsive.

Note: in some cases some basic CSS is omitted from the code examples here as its use is either obvious or not specifically relevant to styling the video player.

###Basic styling
The HTML video and the controls are all contained withing a `figure` element which is given a maximum width and height (based on the dimensions of the video used) and centered within the page:
```css
figure {
   max-width:64rem;
   width:100%;
   max-height:30.875rem;
   height:100%;
   margin:1.25rem auto;
   padding:1.051%;
   background-color:#666;
}
```

The video controls container itself also needs some styling so that it's set up the correct way:
```css
.controls {
   width:100%;
   height:8.0971659919028340080971659919028%; /* of figure's height */
   position:relative;
}
```
The height of the `.controls` element is set to be (a very precise!) percentage of the enclosing `figure` element (this was worked out with experimentation based on the required button height). Its position is also specifically set to `relative` which is required for its responsiveness (more on that later).

As mentioned earlier, a `data-state` is now used to indicate whether the video controls are visible or not and these also need to be styled:
```css
.controls[data-state=hidden] {
   display:none;
}
.controls[data-state=visible] {
   display:block;
}
```

There are a number of properties that also need to be set for all elements within the video controls:
```css
.controls > * {
   float:left;
   width:3.90625%;
   height:100%;
   margin-left:0.1953125%;
   display:block;
}
.controls > *:first-child {
   margin-left:0;
}
```
All elements are floated left, as they are to be aligned next to one another, and each element is set to have a width of almost 4% (again the actual value was calculated based on the required dimensions of the buttons), and a height of 100%. A value for `margin-left` is also set, but the first element (in this case the play/pause button) is  then set to have no left margin.

The `div` container for the `progress` element also requires some specific settings, it is set to have a much wider width and its cursor value is set to be a `pointer`:
```css
.controls .progress {
   cursor:pointer;
   width:75.390625%;
}
```

####Buttons
The first major styling task to tackle is to make the video control's buttons actually look like and act like real buttons.

Each button has some basic styling:
```css
.controls button {
   border:none;
   cursor:pointer;
   background:transparent;
   background-size:contain;
   background-repeat:no-repeat;
}
```
By default, all `button` elements have a border, so this is removed. Since background images will be used to display appropriate icons, the background colour of the button is set to be transparent, non-repeated, and the element should fully contain the image.

A simple `:hover` and `:focus` state is then set for each button that simply alters the opacity of the button:
```css
.controls button:hover, .controls button:focus {
   opacity:0.5;
}
```

To obtain appropriate button images, a set of free common control set icons was downloaded from the web. EAch image was then converted into its base64 encoded string (using an online [base64 image encoder](http://www.base64-image.de/)), since the images are quite small so are the resultant encoded strings.

Since some buttons have dual functionality, e.g. play/pause, and mute/unmute, these buttons have different states that need to be styled. As mentioned earlier, a `data-state` variable is used to indicate which state such buttons are currently in.

For example, the play/pause button has the following background image definitions:
```css
.controls button[data-state="play"] {
   background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAoCAYAAACM/rhtAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAA2ZpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMy1jMDExIDY2LjE0NTY2MSwgMjAxMi8wMi8wNi0xNDo1NjoyNyAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtcE1NOk9yaWdpbmFsRG9jdW1lbnRJRD0ieG1wLmRpZDpFNDZDNDg2MEEzMjFFMjExOTBEQkQ4OEMzRUMyQjhERCIgeG1wTU06RG9jdW1lbnRJRD0ieG1wLmRpZDpCNkU0NTY5NkE0MDcxMUUyQjgwQkYzQzhCMDZBRTU1NCIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDpCNkU0NTY5NUE0MDcxMUUyQjgwQkYzQzhCMDZBRTU1NCIgeG1wOkNyZWF0b3JUb29sPSJBZG9iZSBQaG90b3Nob3AgQ1M2IChXaW5kb3dzKSI+IDx4bXBNTTpEZXJpdmVkRnJvbSBzdFJlZjppbnN0YW5jZUlEPSJ4bXAuaWlkOjQzQ0QwNDBBMDJBNEUyMTFCOTZEQzYyRDgyRUVBOUZDIiBzdFJlZjpkb2N1bWVudElEPSJ4bXAuZGlkOkU0NkM0ODYwQTMyMUUyMTE5MERCRDg4QzNFQzJCOEREIi8+IDwvcmRmOkRlc2NyaXB0aW9uPiA8L3JkZjpSREY+IDwveDp4bXBtZXRhPiA8P3hwYWNrZXQgZW5kPSJyIj8+kBUJ9AAAAXFJREFUeNrsmLtOAkEUhneQyiAdDTExGlYMBaW9oq/ge8jlUbwkthTY2EGBLehbKK0UxsQgVK7/SWbMZo3j3mbmxPAnXyi2+fIzZ3dmRBAEHucUPO6hBhUyNXAH3umxJRZgCBo/nCKCe+DVoliUN5LUCd46lFOMwk4iPCRCiDl+Ko5X3RJOm99OEcGAyVyIrFO8lEPE9jXTBNvgRq4ba6+ZuAs5nFMwy3NQdFOcRpBSBtfgk6ugykkebZoUpGyBqyxtmhZUaYFnzoKqzcukbdoUVDkGT5wFKSVwEadNV4IqR3+16VrQkxuSVRxBVzvqKija+tQl/fafyx00u7/YBxOOU0yttcEHx9fMPphy/JJQa50krdkUrIMHjruZDdBN25ppwYOsrZkSpNZ68hDFast/Bg7Bo4nDu+7g/m/Oxc6u3+YMnBY6wTEDwXvdbmYXvDi82aKrP183xZQd0LcsSktrIC9PvV+neH1HvRZ0kC8BBgADq2RhyZa7BQAAAABJRU5ErkJggg==');
}
.controls button[data-state="pause"] {
   background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAoCAYAAACM/rhtAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAA2ZpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMy1jMDExIDY2LjE0NTY2MSwgMjAxMi8wMi8wNi0xNDo1NjoyNyAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtcE1NOk9yaWdpbmFsRG9jdW1lbnRJRD0ieG1wLmRpZDpFNDZDNDg2MEEzMjFFMjExOTBEQkQ4OEMzRUMyQjhERCIgeG1wTU06RG9jdW1lbnRJRD0ieG1wLmRpZDpCNzE0QzJGQUE0MDcxMUUyQjgwQkYzQzhCMDZBRTU1NCIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDpCNzAxODM5QUE0MDcxMUUyQjgwQkYzQzhCMDZBRTU1NCIgeG1wOkNyZWF0b3JUb29sPSJBZG9iZSBQaG90b3Nob3AgQ1M2IChXaW5kb3dzKSI+IDx4bXBNTTpEZXJpdmVkRnJvbSBzdFJlZjppbnN0YW5jZUlEPSJ4bXAuaWlkOjQzQ0QwNDBBMDJBNEUyMTFCOTZEQzYyRDgyRUVBOUZDIiBzdFJlZjpkb2N1bWVudElEPSJ4bXAuZGlkOkU0NkM0ODYwQTMyMUUyMTE5MERCRDg4QzNFQzJCOEREIi8+IDwvcmRmOkRlc2NyaXB0aW9uPiA8L3JkZjpSREY+IDwveDp4bXBtZXRhPiA8P3hwYWNrZXQgZW5kPSJyIj8+r7sqzQAAANdJREFUeNrs2MEKwjAMBuDGswd9C/UdPHvy6Ft6UTyKr6RDcceawDpKHZsE2kb4Az87GOiHNLCFvPfOcs2c9ZJ/MKSrDefCaeXnQmm7M9dfpgQoDY+CsDRy9moMeKqICznGJoqHhIie/JhXvnUNmxa9KQF6I3NBfzPFANYC7uTKRtkqeyZLOyQ0dLcVPRgSAAEEEEAAAQQQwJ9ftzQ92YAHzjLKXtmT7YUVX3UA5gK+DJiaMeDNAPCaToyl9dvdTazfpMIC810QJmed3cACk7CjBrByfQQYAHwMIXlfZRgfAAAAAElFTkSuQmCC');
}
```
When the `data-state` of the button is changed, the appropriate image will also be changed. All the other buttons are treated in a similar way.

####Progress bar
The `progress` element has the following basic style set up:
```css
.controls progress {
   display:block;
   width:100%;
   height:81%;
   margin-top:0.125rem;
   border:none;
   color:#0095dd;
   -moz-border-radius:2px;
   -webkit-border-radius:2px;
   border-radius:2px;
}
```
Like the `button` elements, `progress` also has a default border which is removed here. It is also given a slight rounded corner for aesthetic reasons. The `color` is also defined here as Internet Explorer uses this defined colour for styling the progress bar's background colour as it increases.

As mentioned in the [previous article](https://developer.mozilla.org/en-US/Apps/Build/Manipulating_media/cross_browser_video_player), there is a fallback provided for browsers that do not support the `progress` element and this also needs to be styled appropriately:

```css
.controls progress[data-state="fake"] {
   background:#e6e6e6;
   height:65%;
}
.controls progress span {
   width:0%;
   height:100%;
   display:inline-block;
   background-color:#2a84cd;  
}
```
A `data-state` is also used here when a `progress` element is being "faked", and when it's in this state the background colour needs to be set. The internal `span` element that is used as the actual progressing part of the faked progress bar as its width initially set to 0% (it is updated via JavaScript) and it also has its background colour set.

There are some browser specific properties that need to be set to ensure that Firefox and Chrome use the required colour for the progress bar:
```css
.controls progress::-moz-progress-bar {
   background-color:#0095dd;
}
.controls progress::-webkit-progress-value {
   background-color:#0095dd;
}
```
Although the same properties are set to the same value, these rules need to be defined separately, otherwise Chrome ignores it.

That's really it for the basic styling, but there are a number of JavaScript changes that also need to be made to ensure that everything works as expected.

##JavaScript
###Control visibility
The first change is a simple one, as the `data-state` for showing the video controls when JavaScript is available to the browser now needs to be set:
```javascript
// Display the user defined video controls
videoControls.setAttribute('data-state', 'visible');
```
###Progress bar support
A check also needs to be made to set up the "fake" progress bar if the browser doesn't support the `progress` element:
```javascript
var supportsProgress = (document.createElement('progress').max !== undefined);
if (!supportsProgress) progress.setAttribute('data-state', 'fake');
```
###Button functionality
####Play/Pause and Mute
Now that the buttons actually look like buttons and have images that indicate what they do, some changes need to be made so that the "dual functionality" buttons (such as the play/pause button) is in the correct "state" and displays the correct image. In order to facilitate this, a new function is defined called `changeButtonState()` which accepts a `type` variable which indicates the button's functionality:
```javascript
var changeButtonState = function(type) {
   // Play/Pause button
   if (type == 'playpause') {
      if (video.paused || video.ended) {
         playpause.setAttribute('data-state', 'play');
      }
      else {
         playpause.setAttribute('data-state', 'pause');
      }
   }
   // Mute button
   else if (type == 'mute') {
      mute.setAttribute('data-state', video.muted ? 'unmute' : 'mute');
   }
}
```
This function is then called at the relevant event handlers: 
```javascript
video.addEventListener('play', function() {
   changeButtonState('playpause');
}, false);
video.addEventListener('pause', function() {
   changeButtonState('playpause');
}, false);
stop.addEventListener('click', function(e) {
   video.pause();
   video.currentTime = 0;
   progress.value = 0;
   // Update the play/pause button's 'data-state' which allows the correct button image to be set via CSS
   changeButtonState('playpause');
});
mute.addEventListener('click', function(e) {
   video.muted = !video.muted;
   changeButtonState('mute');
});
```
You might have noticed that there are new handlers where the `play` and `pause` events are being reacted to on the video. There is a reason for this! Even though the browser's default video control set may have been turned off, many browsers make them accessible by right clicking on the HTML5 video. This means that a user could play/pause the video from these controls which would then leave the custom control set's buttons out of sync. If a user uses the default controls, the defined Media API events, such as `play` and `pause`, are raised so this can be taken advantage of to ensure that the custom control buttons are kept in sync. Sue to this, a new click handler needs to be defined for the play/pause button so that it too raises the `play` and `pause` events:
```javascript
playpause.addEventListener('click', function(e) {
   if (video.paused || video.ended) video.play();
   else video.pause();
});
```
####Volume
The `alterVolume()` function which is called when the player's volume buttons are clicked also changes, as it now calls a new function called `checkVolume()`:
```javascript
var checkVolume = function(dir) {
   if (dir) {
      var currentVolume = Math.floor(video.volume * 10) / 10;
      if (dir === '+') {
         if (currentVolume < 1) video.volume += 0.1;
      }
      else if (dir === '-') {
         if (currentVolume > 0) video.volume -= 0.1;
      }
      // If the volume has been turned off, also set it as muted
      // Note: can only do this with the custom control set as when the 'volumechange' event is raised, there is no way to know if it was via a volume or a mute change
      if (currentVolume <= 0) video.muted = true;
      else video.muted = false;
   }
   changeButtonState('mute');
}
var alterVolume = function(dir) {
   checkVolume(dir);
}
```
This new `checkVolume()` function does pretty much the same thing as the `alterVolume()` but it also sets the state of the mute button, depending on the video's current volume setting. `checkVolume()` is also called when the `volumechange` event is raised:
```javascript
video.addEventListener('volumechange', function() {
   checkVolume();
}, false);
```
####Progress bar
A slight change also needs to be made to the click handler for the `progress` element. Since the enclosing `figure` element now has `position:relative`, the calculations made by this click handler are incorrect. It now also needs to take into account the offset position of the parent element:
```javascript
progress.addEventListener('click', function(e) {
   var pos = (e.pageX  - (this.offsetLeft + this.offsetParent.offsetLeft)) / this.offsetWidth;
   video.currentTime = pos * video.duration;
});
```

####Fullscreen
The FullScreen API implemention hasn't changed.

##Responsive
Now that the player is styled, some other styling changes need to be made in order to make it responsive. Obviously media queries will be used.

The player as it is works fairly well until displayed on a "medium" screen, e.g. `1024px` (`64em`). In this case, the margins and padding on the `figure` element need to be removed so that advantage is taken of all the available space, and the buttons are a bit too small, so this needs to be altered by setting a new height on the `.controls` element:
```css
@media screen and (max-width:64em) {
   figure {
      padding-left:0;
      padding-right:0;
      height:auto;
   }
   .controls {
      height:1.876rem;
   }
}
```
This works well enough up until it is viewed on a smaller screen, so it is here that another breakpoint is made, with a maximum screen width of `680px` (`42.5em`). Since the height of the `.controls` element will now vary, a fixed height is no longer required so it is set to `auto`.
```css
@media screen and (max-width:22.5em) {
   .controls {
      height:auto;
   }
}
```
THe definitions for the elements within the `.controls` element now also need to changed:
```css
@media screen and (max-width:42.5em) {
   .controls > * {
      display:block;
      width:16.6667%;
      margin-left:0;
      height:2.5rem;
      margin-top:2.5rem;
   }
   .controls .progress {
      position:absolute;
      top:0;
      width:100%;
      float:none;
      margin-top:0;
   }
   .controls .progress progress {
      width:98%;
      margin:0 auto;
   }
   .controls button {
      background-position:center center;
   }
}
```
The `.progress` container is now moved to the top of the control set via `position:absolute` and it and all the buttons now need to be wider. In addition, the buttons need to be pushed below the progress container so that they are visible.