# Speech Emotion Detector

Final project for the Building AI course

## Summary

This project detects the emotion of a speaker with Neural Networks. A speech recording will first be analyzed for relevant acoustic properties of speech and based on these properties, the best matching emotion (neutral, sad, happy, and angry) will be chosen.

## Background

An emphatic human, native listener of a language can mostly detect emotion in speech without big efforts. However, in some cases, emotion in speech is not always easy to recognize:

* Non-native listeners, i.e. learners of a language, need a lot of experience with the target language until they feel confident enough to identify the emotional background of an utterance. Emotion is marked with so-called suprasegmental properties, i.e. phonological properties that describe more than one specific speech sound, e.g. duration, pitch, and loudness. Regular language classes can not include lessons to teach which exact combination of duration, pitch, and loudness implies which emotion. Moreover, mostly not not one easy to define combination of acoustic properties describes clearly one single emotion. Therefore, a language learner of an intermediate state will have problems to detect the emotion of an interlocutor when no visual information of the speaker's face is available (e.g. in a phone call or when the listener suffers from visual impairments).<br>
* Automatic Speech Recognition (ASR): Home assistant systems, call center bots can profit from the planned AI-system. An ASR program that only focusses on understanding _what_ is being said by the user will at some point leave the user unsatisfied. If the ASR programm detects the emotion on the flow, the home assistant or the chat bot will be empathic, the answers can be adapted (content-wise and intonation-wise), and the dialogue will become more human-like.


## How is it used?

The solution will have two potential user groups:
* Language learners can use the program to check the emotion of a sound file (.wav format). The user will upload a wave file and the output will be displayed on the screen. The learner can for example upload a voice message received on any messenger app which will then be analyzed. For live analysis, the learner needs to rely on ASR systems that make use of the present solution (see next bullet point).
* The present solution can be integrated into ASR systems. The suggestion is to record full sentences to wave files and use these wave files as input. The output will be an emotion for each sentence and the global system can use this information for further processing.

The program shall ideally run for many different languages. As a starting point and for practical reasons (data availability), it will be set up for Turkish only.


## Data sources and AI methods

The first set of training data will be taken from a study investigating acoustic markers of emotion in Turkish (Oflazoglu & Yildirim, 2013). After some pre-processing, their data can be facilitated to the following structure:

```
utterance = ['ut_train1', 'ut_train2', 'ut_train3', 'ut_train4', 'ut_train5']
ac_pitch_min = [223, 534, 643, 1034, 124]
ac_pitch_max = [2423, 2534, 1343, 1079, 991]
ac_loudness_mean = [35.54345, 47.53445, 39.1343, 40.1079, 49.4587]
emo = ['sad','angry','neutral','happy','angry']
```

* "utterance" defines the ID of a unique training data point
* "ac_pitch_min" defines the minumum, i.e. lowest, pitch measured in the sample
* "ac_pitch_max" defines the maximum, i.e. hihgest, pitch measured in the sample
* "ac_loudness_mean" defines the mean loudness of the sample
* "emo" is the emotion category that was determined for the data point (neutral, sad, happy, and angry)

In addition to the above mentioned acoustic properites, additional properties as suggested by Oflazoglu & Yildirim (2013) will be added.

The test input will be a wave file, therefore the current solution will need to perform acoustic measurements as a first step. As suggested by Oflazoglu & Yildirim (2013), this will be performed with te openSMILE toolkit (Eyben, Wöllmer, & Schuller, 2010). This is a sample data point:

```
utterance = ['ut_test1']
ac_pitch_min = [723]
ac_pitch_max = [2032]
ac_loudness_mean = [33.54345]
emo = [NA]
```

We are using supervised learning for implementing the emotion categorization task. 

x is our training data that we will bring to the following format. The information for "emotion" will be remapped to numbers (neutral=0, sad=1, happy=2, angry=3):
```
x=np.array([[223, 2423, 35.54345, 1],
            [534, 2534, 47.53445, 3],
            [643, 1343,  39.1343, 0],
            [1034, 1079, 40.1079, 2],
            [124, 991, 49.4587, 3]])
```

y is our test data, the output from the acoustic analyzer:
```
y=np.array([723, 2032, 33.54345)])
```


Following the section "Neural Networks" in the "Building AI" course, we will train a network with the data from x:
* Input nodes: With the reduced sample data set, we have three input nodes (node1=ac_pitch_min, node2=ac_pitch_max, node3=ac_loudness_mean)
* Hidden layers: In the hidden layers we apply a logistic regression function and a sigmoid function; we might also add a bias node
* Output layer: Here, we will get the information on which emotion category to select. The output will be a number between 0 and 3 which defines the emotion.

If successfully analyzed, the test data string with its newly added emotion category will be added to the training data to further improve the model.


## Challenges

We will start with a small set of acoustic variables that were found to give a hint on the emotional background of a speaker. However, our understanding of acoustical markers for emotion in speech is still incomplete and we are restricted in what we can measure in a speech signal. Moreover, tools for acoustic analysis of wave files are error prone. Also, the input data, i.e. the wave files to be analyzed need to have a minimum level of quality to be useful. So it might be that our model will not be able to make meaningful predictions or that the predictions cannot be confirmed by human raters. To check the quality of the model, we can use cross-validation, though.<br>
The idea to add test data strings to the training data set will only make sense if we can be sure our model performs well. For checking that, it will even be useful to manually check a set of analyzed new, real-live data that is not part of the training data set - maybe even data based on lower quality wave files incl. background noise, for example.<br>
For integrating this model into an ASR-system, a lot of work is required still. It needs to be well considered how big the data chunks that are used as input shall be (one sentence or several sentences?), and how to handle the collected emotion data: Use the most frequently detected emotion of one 20-minute window? Consider differences as actual emotional instability of the interlocutor?<br>
With respect to ethical considerations, we need to make clear to the user group of language learners that the results constitute an idea, an orientation and not the absolute truth. We want to stress that even human beings might interprete the behaviour and therefore also the emotions of one individuum differently depending on their own previous experiences and knowledge.


## What next?

Obviously, big sets of annotated data in different languages will help to obtain meaningful results for the present project. Moreover, the concrete setup of the neural network is not yet defined in this README. The network needs to generate a categorical output, either 0, 1, 2, or 3.<br><br>
The training data (that shall help non-native listeners, i.e. language learners of the target language) shall ideally be produced by native speakers of a respective language to work well with test data produced by native speakers. It is getting more complex, but also interesting, if we consider test data produced by non-native speakers of a certain language. In the "Background" section of this README, we are assuming that acoustic markers for emotion in speech are language specific and therefore justify the use of this program for language learners. But what if acoustic markers for emotion are universal across most languages? A first step to investigate this question can be a cross-validation with a) native speaker training data and b) non-native speaker test data, both producing the same language.


## Acknowledgments

* Oflazoglu & Yildirim (2013). EURASIP Journal on Audio, Speech, and Music Processing, http://asmp.eurasipjournals.com/content/2013/1/26
* Eyben, Wöllmer, & Schuller (2010). International conference on multimedia. openSMILE: the Munich versatile and fast open-source audio feature extractor (ACM Firenze, Italy, 25–29 Oct 2010)
