# Webgazer Tutorial


- [JsPysch links](#JsPysch)
- [Webgazer](#Webgazer)
- [Calibration plugin-in](#Calibration)
- [Record eye data](#Recording)
- [Beckend Options](#Beckend)
- [Eye data](#Eyedata)
- [Templetes](#Templetes)





## JsPsych

- You may want to check JsPsych website:  <a href="https://www.jspsych.org" target="_blank">https://www.jspsych.org</a>.

- Use this as a general reference for making the experiment trials, function calls in JavaScript.



## Webgazer

- Important documentation about webgazer: <a href="https://webgazer.cs.brown.edu" target="_blank"> https://webgazer.cs.brown.edu</a>.

- The regression modules determine how the regression model is learned and how predictions are made based on the eye positions extracted from the tracker module. The regression modules that are available with webgazer:
    - ridge - a simple ridge regression model mapping pixels from the detected eyes to locations on the screen.
    - weightedRidge - a weight ridge regression model with newest user interactions contributing more to the model.
    - threadedRidge - a faster implementation of ridge regression that uses threads.

- In the templete, we used the threadedRidge. If you use this regression method, make sure that you download the three files from the website: https://github.com/brownhci/WebGazer/tree/master/src, and put them in a folder under your library: 
    - mat.js
    - ridgeWorker.js
    - util.js



- Some important function calls in webgazer you have to know: 

```javascript
webgazer.begin()
webgazer.pause()
```
- You only have to call webgazer.begin() once during the study, this will be embeded in the calibration script, so you don't have to worry about this. But make sure your webcam is ready at the beginning of the study. Sample code is shown below:

```javascript
$('body').on('click', '.goExp', function () {
        showAlert();
    }).on('click', '.checkmark', function (event) {
        goExp.addEventListener('click', () => {
            ensureWebcam(() => startExperiment()); // start the experiment after the webcam is ready.
        });
        event.stopPropagation();
    });

// ensure the user's webcam is able to use. Some browsers may automatically block the webcams.
    function ensureWebcam(callback) {
        $('body').html('<p class="center", style="font-family:Arial, Helvetica, sans-serif">Checking webcam ...</p>');
        window.navigator.mediaDevices.getUserMedia({ video: true })
        .then(function(stream) {
            stream.getTracks().forEach(track => track.stop());
            callback();
        })
        .catch(function() {
            alert('Cannot open webcam.\nIs it blocked? This app requires webcam.');
            location.reload();
        });
    }
   
```

- You have to pause the webgazer when there is a break during the experiment. A example is like this for the break:

```javascript

var breaktime = {
  type: "html-keyboard-response",
  stimulus: `<div> Now you can take a short break if you want. When you are ready to continue the study, press the <b>SPACE BAR</b>.</div>`,
  choices: ['spacebar'],
  on_start: function () {
    webgazer.pause(),
      webgazer.clearData() // clear the training data
  },
};

```



- Other important function calls you may want to use:

```javascript
// show prediciton points on the screen, default is false.
webgazer.showPredictionPoints();

// show the video feed on the screen, default is true.
webgazer.showVideo(doVideo);

// show the face overlay on the screen, default is true.
webgazer.showFaceOverlay(doVideo);

// show the face feedback box on the screen, default is true.
webgazer.showFaceFeedbackBox(doVideo);

// clear the previous training data
webgazer.clearData()
```



## Calibration
- An example of calibration:

[![](http://img.youtube.com/vi/PRz2LWKyngw/0.jpg)](http://www.youtube.com/watch?v=PRz2LWKyngw "webgazercalibration")








- Parameters for eye-tracking.js plugin. The eye-tracking plugin does the calibration and validation phases before the start of the task using webgazer.

- Parameters with a default value of undefined must be specified. Other parameters can be left unspecified if the default value is acceptable.


| Parameter           | Type    | Default value | Description                                                                                                                                        |
|---------------------|---------|---------------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| doInit              | boolean | False         | Specifies whether it is the first time to load the web cam and do the calibration.                                                                 |
| doVideo             | boolean | False         | If true, the video from the web cam is shown to the subject.                                                                                       |
| showPoint           | boolean | False         | If true, the prediction points will be shown on the screen.                                                                                        |
| doCalibration       | boolean | False         | If true, Calibration phase is initiated.                                                                                                           |
| calibrationMethod   | string  | ‘watch’       | Label determines which calibration method is used.                                                                                                 |
| calibrationDots     | numeric | 5             | Specifies the number of calibration dots used.                                                                                                     |
| calibrationDuration | numeric | 3             | Specifies how long the calibration dot appears on the screen in seconds                                                                            |
| doValidation        | boolean | False         | If true, validation phase is initiated.                                                                                                            |
| validationDots      | numeric | 5             | Specifies the number of validation dots to be used.                                                                                                |
| validationnDuration | numeric | 2             | Specifies how long the validation dot appears on the screen in seconds                                                                             |
| validationTol       | numeric | 200           | Specifies the tolerance level in pixels for a prediction to be considered successful.                                                              |
| validationThreshold | numeric | 0.7           | Specifies the proportion of successful validation points that need to be successful in order for the validation phase to be considered successful. |
    
    

- An example of calibration, this is an inital calibration before the study starts, the timeline for the experiment is:
    -  eyeTrackingNote: show the instruction for eye calibration.
    -  do calibration: specify the plugin-in type: "eye-tracking", and then specify the parameters. 


```javascript
var inital_eye_calibration = {
  timeline: [
    eyeTrackingNote, // instruction 
    {
      type: "eye-tracking",
      doInit: true,
      doCalibration: true,
      doValidation: true,
      calibrationDots:  12,
      calibrationDuration: 3, 
      doValidation: true,
      validationDots:  12,
      validationDuration: 2,
      validationTol: 130,
     ],
};

```
- if doInit is false, this massage will be shown: 

```HTML
We need to re-calibrate you.  Take a short break and proceed by pressing the SPACE BAR when you are ready.
```

- if doInit is true, this massage will be shown. This is a waiting page for subjects to adjust their positions.

```HTML
Before you begin the calibration, please wait until the video feed appears on your screen. Follow the previous tips to adjust your position relative to your webcam.  When you are ready, please press the SPACE BAR to continue.
```

## Recording
- In this templete, eye tracking data is recorded for a two alternative forced choice experiment. However, you can easily put this recording function into any trial you want to record. You just have to call prediction function and save the eye tracking data:

    - Step 1:  add an option for doEyeTracking in your plugin infomation. An example is like this: 
    ```javascript
    doEyeTracking: {
        type: jsPsych.plugins.parameterType.BOOL,
        pretty_name: 'eye-tracking',
        default: true,
        description: 'Whether to do the eye tracking during this trial.'
      }
    ```
    
    - Step 2: create an object to save eye tracking data.
    ```javascript
    var eyeData = {history:[]};
     ```
     
    - Step 3:  Copy this block of the copy into your script. Make sure before this trial, webgazer has already began. So you are able to resume webgazer. Make sure the positions are relative to the user's screen becasue different users have different screen size. You may also record the screen size as an extra variable.
   
   ```javascript
    if(trial.doEyeTracking) {
    webgazer.resume();
    webgazer.showVideo(false);
    webgazer.showPredictionPoints(false);
    webgazer.showFaceOverlay(false);
    webgazer.showFaceFeedbackBox(false);
    var starttime = performance.now();
    var eye_tracking_interval = setInterval(
        function() {
            var pos = webgazer.getCurrentPrediction();
            if (pos) {
                var relativePosX = pos.x/innerWidth ;
                var relativePosY = pos.y/innerHeight;
                eyeData.history.push({
                    'relative-x': relativePosX,
                    'relative-y': relativePosY,
                    'elapse-time': performance.now() - starttime
                });}},20);}  
                // specify your sampling rate, 20 means every 20ms, this function will be called.

     ```
    - Step 4:  Save eye tracking data.
    
    ```javascript
    if(trial.doEyeTracking) {
    webgazer.pause(); // pause the webgazer before you save the data, this is optional
    clearInterval(eye_tracking_interval); } // clear the time interval before you save the data, so next time you use this trial, the timer will start from beginning. 
      // data saving
      var trial_data = {
        "rt": response.rt,
        "key_press": response.key,
        "choices": trial.choices,
        "eyeData": JSON.stringify(eyeData), //save your eye tracking data 
      };
      jsPsych.finishTrial(trial_data);
    };
    ```



## Beckend

- After coding an experiment, you have to build an experiment online. 
    - If your lab has in-house server, then you just have to link the experiment to your server and post the link to experimental platform.
    - If you are running the experiment on MTurk, feel free to refer to this page and use PsiTurk as a beckend support: https://www.jspsych.org/overview/mturk/
    - We used Heroku as our beckend support https://www.heroku.com . Running Psychology experiments won't cost you any fees (unless you have many experiments need to run at the same time). We packed our experiment into an node.js app and then deploy the experiment on Heroku. 
        - Helpful Links: https://blog.heroku.com/six-strategies-deploy-to-heroku
    - Heroku has its own data storage service that you can save your data. 
        - Helpful Links: https://www.heroku.com/managed-data-services
    - We directly upload the data into our dropbox account. 
        - Helpful Links: https://www.dropbox.com/developers/documentation/javascript#tutorial


- Here is the documentation of use the script as a templete and then save data into dropbox account (you may find more useful infomation on other weblinks):
     - two data will be added into your dropbox for one subject:
         - this first is a separate JSON file that will update whether this subject has passed the initial calibration or not. These two objects are related to make the separate JSON file. If a subject pass the calibration, his/her folder name will begin with 'cg'; otherwise, his/her folder name will begin by 'sb'. (There's no particular reasons I picked these characters... ;)
         - the second is all the experiment data.
        
        - related objects are as follows:

            ```javascript
            function makeSurveyCode(status) {
                uploadSubjectStatus(status);
                var prefix = {'success': 'cg', 'failed': 'sb'}[status]
                return ... 
                }
            
            var uploadSubjectStatus = function(status) {...}
            var on_finish_callback = function () {...}
            ```
     - Craete a file called "app.js", this is the file that should be separated from your front-end scripts. Here, if you don't have knowledge in beckend, watching a YouTube video will be very helpful. You only need to do this once, and you can use same script every time.
     - Remember do intial steps as the tutorial said if you are linking to dropbox (https://www.dropbox.com/developers/documentation/javascript#tutorial). 
      
      - First, load modules:
       ```javascript
        const Dropbox = require("dropbox").Dropbox;
        const fetch = require("node-fetch");
        const body_parser = require("body-parser");
        var express = require("express");
       ```
       - second, instaniate the app: 
       ```javascript
       var app = express();
       ```
       - set port and dropbox file saving details (see the templete for an example)

    - After you successfully tested the experiment app in your local enviroment and successfully receive the data from your dropbox, then you can push your experiment onto heroku.  
        - pushing code from a Git repository to a Heroku app. 
        - You simply add your Heroku app as a remote to an existing Git repository, then use git push to send your code to Heroku
        - details can be found here: https://blog.heroku.com/six-strategies-deploy-to-heroku
        - you may check your beckend status looking at the log files in order to see if the experiment is succssfully pushing on to the server or not (no need to pray :)
   
   
   

     

## Templetes


> The example video: insert the experiment video here
 
> If you want to make this templete work on your laptop, you should first go to https://www.dropbox.com/developers/apps to create an app and then replace your access token here. You had better create a separate file to save your accesstoken to protect the data. 

```javascript
const dbx = new Dropbox({
    accessToken: 'YOURACCESSTOKEN',
    fetch
});
```
> Once you have your app setup in your dropbox, you can cd to your experiment folder: 

```shell
$ npm install
$ node app.js
```


> Next, you can go to your Chrome or Firefox browser and type in the port number (dafault is 2000) and begin the experiment.


> A front-end only templete is also available?

## Eyedata
- Example code for transferring data into csv in python：


```Python
#import all the libraries
import os, json
import pandas as pd
import numpy as np
import glob
from pprint import pprint
from jsonmerge import merge
import matplotlib.pyplot as plt  
pd.set_option('display.max_columns', None)
```

```Python
status= pd.read_csv('status.csv') # here you have to transfer "SubjectStatus.JSON" to csv first

success_list = list(status[status['record'] == 'success']['folderName']) #select all success status workers

for i in range(len(success_list)): 
    user_id = success_list[i]
    path_to_json = success_list[i] #user_id 
    json_pattern = os.path.join(path_to_json,'*.json') 
    file_list = glob.glob(json_pattern)
    
    all_data = pd.DataFrame()
 
    for lisr in file_list:
        data = pd.read_json(lisr)
        all_data = all_data.append(data, ignore_index = True)
    
    all_data= all_data.sort_values(by=['trial_index'])
    all_data.to_csv(....)
```




## Others: useful git command:
```shell
$ Git add   
$ Git commit -m “message”
$ Git status  (what’s current going on in your repository)
$ Git push  //push our code from our local computer to the repository 
$ Git pull  // pull down 
$ Git log
```





## Reference
- remember to add webgazer's license 
