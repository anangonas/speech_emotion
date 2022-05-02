# Speech Emotion Detector

Final project for the Building AI course

## Summary

This project detects the emotion of a speaker with a neural network. A speech recording will first be analyzed for relevant acoustic properties of speech and based on these properties, the best matching emotion (neutral, sad, happy, and angry) will be assigned.

## Background

An emphatic human, native listener of a language can mostly detect emotion in speech without big efforts. However, in some cases, emotion in speech is not easy to recognize:

* Non-native listeners, i.e. learners of a language, need a lot of experience with the target language until they feel confident to identify the emotional background of an utterance. Emotion is marked with so-called suprasegmental properties, i.e. phonological properties that describe more than one specific speech sound, as for example duration, pitch, and loudness. Regular language classes cannot teach which exact combination of duration, pitch, and loudness implies which emotion. This is mostly because the correlation between acoustic properties and specific emotions is mostly not straight forward. Therefore, a language learner of an intermediate state will have problems to detect the emotion of an interlocutor when no visual information of the speaker's face is available. This is the case in a phone call or when the listener suffers from visual impairments.<br>
* Automatic Speech Recognition (ASR): Home assistant systems and call center bots can profit from the planned AI-system. An ASR program that only focusses on understanding _what_ is being said by the user will at some point leave the user unsatisfied. If the ASR programm detects the emotion on the flow, the home assistant or the chat bot will be empathic, the answers can be adapted (content-wise and intonation-wise), and the dialogue will be more human-like.


## How is it used?

The solution will have two potential user groups:
* Language learners can use the program to check the emotion of a sound file (.wav format). The user will upload a wave file and the emotion will be displayed on the screen. The learner can for example upload a voice message received on any messenger app. For live analysis, the learner needs to rely on ASR systems that make use of the present solution, as described below.
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
* Input layer: With the reduced sample data set, we have 3x5=15 input nodes: 3 phonetic properties (ac_pitch_min, ac_pitch_max, ac_loudness_mean) with 5 samples each.
* Hidden layer: In the hidden layer we apply a logistic regression function and a sigmoid function; biases and weights are specified.
* Output layer: this layer reveals the information on which emotion category to select. There will be 4 output neurons in the output layer, one neuron for each potential emotion. The sigmoid function from the hidden layer passes a number between 0 and 1 over to the output layer neurons. The neuron with the highest value will fire and therefore inform us about the detected emotion of a speech sample. Neuron1 will fire if the detected emotion is 'neutral', neuron2 fires if 'sad' is detected, neuron3 for 'happy', and neuron4 if the test input was identified as 'angry'.

If we have a big enough set of training data available, we will use cross-validation to check the quality of our model.


## Challenges

We will start with a small set of acoustic variables that were found to give a hint on the emotional background of a speaker. However, our understanding of acoustical markers for emotion in speech is still incomplete and we are restricted in what we can measure in a speech signal. Moreover, tools for acoustic analysis of wave files are error prone. Also, the input data, i.e. the wave files to be analyzed need to have a minimum level of quality to be useful. So our model might not make meaningful predictions or the predictions are not confirmed by human raters.<br>
For integrating this model into an ASR-system, a lot of work is required still. It needs to be well considered how big the data chunks that are used as input shall be (a half-sentence, a full sentence or even several sentences?), and how to handle the collected emotion data: Use the most frequently detected emotion of one 20-minute window? Consider differences as actual emotional instability of the interlocutor?<br>
With respect to ethical considerations, we need to explicitly explain our user group of language learners that the results constitute an idea, an orientation and not the absolute truth. Importantly, human beings might interprete the behaviour and therefore also the emotions of one individuum differently depending on their own previous experiences and knowledge. In the case of our network, many different previous experiences can create a big, diverse foundation for any interpretation. This can be a big advantage on the one hand, but also be an obstacle for valid predictions.

## What next?

* Technically: Big sets of annotated data in different languages will help to obtain meaningful results for the present project. Moreover, the concrete setup of the neural network is not yet defined.<br><br>
* Content-wise: The training data (that shall help non-native listeners, i.e. language learners of the target language) shall ideally be produced by native speakers of a respective language to work well with test data produced by native speakers. It is getting more complex, but also interesting, if we consider test data produced by non-native speakers of a certain language. In the "Background" section of this document, we are assuming that acoustic markers for emotion in speech are language specific and therefore justify the use of this program for language learners. But what if acoustic markers for emotion are universal across most languages? A first step to investigate this question can be a cross-validation with a) native speaker training data and b) non-native speaker test data, both producing the same language.


## Acknowledgments

* https://buildingai.elementsofai.com/
* Oflazoglu & Yildirim (2013). EURASIP Journal on Audio, Speech, and Music Processing, http://asmp.eurasipjournals.com/content/2013/1/26
* Eyben, Wöllmer, & Schuller (2010). International conference on multimedia. openSMILE: the Munich versatile and fast open-source audio feature extractor (ACM Firenze, Italy, 25–29 Oct 2010)
