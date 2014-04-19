#Cross Browser Video Player Styling

In a [previous article](https://developer.mozilla.org/en-US/Apps/Build/Manipulating_media/cross_browser_video_player) I described how to build a cross-browser HTML5 video player using the Media and Fullscreen APIs. This follow-up article describes how to style this custom player, including making it responsive.

##HTML Markup

In order to make the styling task easier, there are a number of changes that will be made to the HTML markup shown in the previous article. The change surrounds the custom video controls themselves which, instead of remaining as an unordered list, these are now contained withing a `div`, as is the `progress` element.

The markup for the cutom controls now looks as follows:

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