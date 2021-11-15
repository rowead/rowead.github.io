---
layout: article
title: Classify Images and text with cloud services
excerpt: >
  A command line tool to classify and describe images or extract named entities from text.
published: true
---
Classify is a tool that lets you easily run Computer Vision or Natural Language services over your own data. All you need is an account with either Microsoft or Google wih the relevant services enabled. See account setup instructions: [Google](https://github.com/googleapis/nodejs-vision#before-you-begin)  [Microsoft](https://docs.microsoft.com/en-us/azure/cognitive-services/computer-vision/quickstarts-sdk/image-analysis-client-library?tabs=visual-studio&pivots=programming-language-javascript#prerequisites).

The tool comes with two commands, "classify images" and "classify text". The first allows you to label and describe a folder full of images and save the results to a csv. Image descriptions are only available using Microsoft's services at the moment but both return labels relevant to the content of your images.

```shell
./classify images --path=/path/to/images \
  --vendor=microsoft \
  --microsoft-key="INSERT KEY" \
  --microsoft-endpoint="INSERT ENDPOINT" \
  --output-file="my_labels.csv"
```

As you can see, the results are not bad but would require some checking. They would be very useful as a first pass over new unlabeled content.

<div class="center">
  {% include fig-caption.html url="/assets/images/2021-11/GM03618_jpg.jpg" class="image--xl" description="Caption: a building with a sign on it (0.44 confidence)<br />Labels: text, building, outdoor, road, black, white, old, city, town,<br />government building" %}
</div>
<div class="center">
  {% include fig-caption.html url="/assets/images/2021-11/GM03627_jpg.jpg" class="image--xl" description="Caption: a group of people on bicycles (0.58 confidence)<br />Labels: outdoor, person, posing, bicycle, group, people, old, vintage" %}
</div>
<div class="center">
  {% include fig-caption.html url="/assets/images/2021-11/GM095010_jpg.jpg" class="image--xl" description="Caption: a man pushing a wheelbarrow (0.53 confidence)<br />Labels: outdoor, ground, transport, old" %}
</div>

The second command is designed to be used on database exports as part of a processing pipeline to clean and enrich.

```shell
./classify text --csv="./samples/samples.csv" \
  --id-column="Accession Number" \
  --classify-column="Description"
```

This will add additional fields that are returned from [Google's Natural Language API](https://cloud.google.com/natural-language/) including identified persons, organisations, locations, events etc.

 

This is a quick proof of concept but it could easily be adapted to work with any data source. The tool is available on github [https://github.com/rowead/classifier](https://github.com/rowead/classifier)