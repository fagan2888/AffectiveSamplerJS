# AffectiveSamplerJS
A Javascript library to record people's affective reactions to audio/video sequences.

You can preview it on CodePen here:
https://codepen.io/QuentinAndre/pen/ROrByr

## 1. Using AffectiveSamplerJS in Qualtrics

### A. Word of caution

Using videos in online experiments can be problematic if the respondents are using outdated browsers or have a slow 
connection. Before using this task, I recommend that you include this code on the first page of the survey to test 
that the respondent can see the video/audio:

#### For video content:
```html
<video id="video" controls>
  <source src="http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4">
   If you see this text rather than a video, it means that your browser does not support this task. 
   Please return the HIT.
</video>
```

#### For audio content:
```html
<audio id="audio" controls>
  <source src="https://sample-videos.com/audio/mp3/crowd-cheering.mp3">
   If you see this text rather than an audio file, it means that your browser does not support this task. 
   Please return the HIT.
</audio>
```

### B. Setting up the library in Qualtrics

Just follow those four steps:

1. Navigate to the "Look and Feel" section of your survey, and click on the "Advanced" tab
2. Edit the "Header" section, and add the following lines to load the library script:
```html
<script src="https://cdn.jsdelivr.net/gh/QuentinAndre/AffectiveSamplerJS/lib/affectivesampler.min.js"></script>
```
3. Create a "Text" question, and add the following HTML code:
```html
<div id="mysamplingtask">&nbsp;</div>
```

4. Edit the "Custom JS" of the question, and add the following Javascript code in the `Qualtrics.SurveyEngine.addOnReady` section:
```javascript
const aff_sampler = new AffectiveSampler({
    parentId: "mysamplingtask",
    mediaUrl: "http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4", // The media to show. 
                                        // It should be hosted online (as .mp4 for maximum compatibility).
                                        // Qualtrics hosting does not work.
    mediaType: "video", // The type of the media, can be "video" or "audio"
    showMediaControls: false, // Whether the show the media controls. If false, only a play/pause button will be displayed.
    sliderMin: -50, // The minimum value of the affective slider
    sliderMax: 50, // The maximum value of the affective slider
    timeResolution: 1, // The time resolution (in seconds) at which to sample. Should not be lower than 1.
    textLabel: "Your current happiness rating is: ", // The label to display next to the current affective rating.
    minLabel: "Sad", // The left anchor of the affective scale.
    maxLabel: "Happy", // The right anchor of the affective scale.
    onInterval: function () {console.log("The interval has ticked")}, // A function to call when recording a value.
    onMediaEnd: function () {console.log("The media has ended")} // A function to call when the media ends.
});
```

That's it! You have added an affective sampling task to your Qualtrics survey! To access and store the participants' 
responses, follow the instructions below.

### C. Accessing and storing participants' ratings

AffectiveSampler records, at a regular interval, the rating of the media. The ratings are stored in a javascript object, 
where the key is the timestamp (in seconds) and the value is the rating at this timestamp. If, for instance, the media 
is four seconds long, the ratings might look like this:

```javascript
const ratings = {
    1: 35, // At 1 second, the rating was 35.
    2: 41, // At 2 seconds, the rating was 41
    3: 12, // At 3 seconds, the rating was 12
    4: 56  // At 4 seconds, the rating was 56
}
```

Those ratings can be accessed using two convenient methods:
* `expSampler.getRatingsAsJSON()` returns the javascript object of the ratings. This method is useful when you want
to use the data in javascript.
* `ExperienceSampler.getRatingsAsString()` returns the stringified javascript object of the ratings. This method is 
useful when you want to store the data in Qualtrics.

Combined with the `onMediaEnd` argument, you can use those methods to store the ratings in Qualtrics when the task is
complete:

```javascript
function storeRatingsInQualtrics() {
    const ratings = this.getRatingsAsString();
    Qualtrics.SurveyEngine.setEmbeddedData("ratings", ratings) // Store the data in an embedded data field called "ratings"
}

const aff_sampler = new AffectiveSampler({
    parentId: "mysamplingtask",
    mediaUrl: "http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4", // The media to show.
    mediaType: "video", // The type of the media, can be "video" or "audio"
    showMediaControls: false, // Whether to show the media controls. If false, only a play/pause button will be displayed.
    canPauseMedia: true, // Whether the user can pause the video after starting it.
    sliderMin: -50, // The minimum value of the affective slider
    sliderMax: 50, // The maximum value of the affective slider
    timeResolution: 1, // The time resolution (in seconds) at which to sample. Should not be lower than 1.
    textLabel: "Your current happiness rating is: ", // The label to display next to the current affective rating.
    minLabel: "Sad", // The left anchor of the affective scale.
    maxLabel: "Happy", // The right anchor of the affective scale.
    onInterval: function () {console.log("The interval has ticked")}, // A function to call when recording a value.
    onMediaEnd: storeRatingsInQualtrics // No parenthesis! This function will be called when the task ends.
});
```

## 2. Version History

### v0.6.0
* Added the `canPauseMedia` option. If set to `false`, the user will not be able to pause the media after starting it.

### v0.5.0
* First release of the library.
