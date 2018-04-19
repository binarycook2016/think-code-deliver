# think-code-deliver

## Steps

### 1. Build Your Own Toolchain
* After you [sign in](https://ibm.biz/BdZzHH) to IBM Cloud, go to `Catalog`
* From the left menu, select `DevOps` catalog
* Choose `Continuous Delivery` and `Create` an instance
* Click `Getting started` and choose `Build your own toolchain`
### 2. Create a Git Repository
* Click `Add tool` and choose `Github repos`
  - Select `clone` and enter the following link:
  `https://github.com/IBM/watson-conversation-slots-intro`
  then name your repo
### 3. Create a Delivery Pipeline
* Click `Add tool` and choose `Delivery Pipeline`
* Name your pipeline and click `create`
* Click on your pipeline to configure it
* Click `Add stage` and name the stage **BUILD**
  - From `INPUT` tap:
    - make sure `Input type` is `Git repository`
    - Select your repo under `Git repository`
    - Make sure `Branch` is set to `master`
    - Leave `Stage trigger` as `Run jobs whenever a change is pushed to Git`
  - From `JOBS` tab:
    - Choose `Simple` as a `Builder type`
    - Make sure `Stop running this stage if this job fails` is checked
* Click `Add stage` and name the stage **DEPLOY**
  - From `INPUT` tap:
    - Input type should be `Build artifacts`
    - Stage is `BUILD`
    - Job is `Build`
  - From `JOBS` tab:
    - Deployer type is `Cloud Foundry`
    - Choose your a location, org and space
    - Enter your app name. It must be unique!
    - In Deploy script, replace it with this code
    ```
    #!/bin/bash
    cf create-service conversation standard wcsi-conversation-service
    cf push "${CF_APP}"
    ```

## Let's make it cognitive!
Let’s imagine the customer has ordered a delivery using pizza-bot and the driver is being (even) slower than normal.
If the customer asks
**“Where is my pizza?”**
We return the standard message all pizza takeaways use when calling to inquire where the driver is….
**“The driver has just left, he’ll be ten minutes.”**
An hour later…
When the driver still hasn’t arrived, the customer would probably ask again and with a bit less civility…
**“Where the heck is my pizza? I ordered an hour ago! This is ridiculous.”**
At this point, the “just ten minutes” reply is not going to be well received!
Building bots that can understand conversation tone will mean we can script a suitable response, rather than infuriating our hungry customers.[* ](jamesthom.as/blog/2016/05/10/bots-with-ibm-watson/)
### 1. Edit Conversation Service
* From Intents tab, add **#deliver** and `$where* pizza` as an example
* From Dialog tab, add a node to reply when the customer as asking about delivery and he is angry
if bot recognizes **#deliver** and **$emotion:anger**
Then respond with:
`Please accept our apologies for the delivery driver being very late. Could you call us on 0800 800 800 and we'll get this fixed`
* And add another node to reply normally when the customer is not
if bot recognizes **#deliver** and **!$emotion:anger**
Then respond with:
`I've just checked and the driver is ten minutes away, is there anything else I can help with?`
### 2. Add Eclipse Web IDE
* From the toolchain, click `Add tool` and choose `Eclipse Web IDE`
* Click on the icon to edit the code! :)
### 3. Add Tone Analyzer
* Define tone analyzer before post function in app.js
```js
var ToneAnalyzerV3 = require('watson-developer-cloud/tone-analyzer/v3');

var toneAnalyzer = new ToneAnalyzerV3({
  username: 'e1e6a2e6-1278-4a89-b2f3-37f87e338e03',
  password: 'Z3RDoD4dmJhD ',
  version: '2016-05-19',
  url: 'https://gateway.watsonplatform.net/tone-analyzer/api/'
});

const getEmotion = (text) => {
  return new Promise((resolve) => {
    let maxScore = 0;
    let emotion = null;
    toneAnalyzer.tone({text: text}, (err, tone) => {
      let tones = tone.document_tone.tone_categories[0].tones;
      for (let i=0; i<tones.length; i++) {
        if (tones[i].score > maxScore){
          maxScore = tones[i].score;
          emotion = tones[i].tone_id;
        }
      }
      resolve({emotion, maxScore});
    });
  });
};
```
* In the post function and after defining the payload, replace existing code with the following:
```js
getEmotion(req.body.input.text).then((detectedEmotion) => {
      payload.context.emotion = detectedEmotion.emotion;
      conversation.message(payload, (err, data) => {
        if (err) {
            return res.status(err.code || 500).json(err);
        }
        return res.json(updateMessage(payload, data));
      });
});
```
* Then, from the top `menu` click on File then `Save`
* Click on Git icon and `commit` your changes then `push`

