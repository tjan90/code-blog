---
layout: post
author: Tanveer jan
title: Face recognition with Docker
date: 2020-07-14
thumbnail: /assets/img/posts/face.jpg
tags: python opencv machineLearning
summary: Facial recognition application 
---

This application recognized face in video stream. The video stream need to passed as the parameter when deploying docker application.

### Installation
<h3>Docker</h3>

You need to install docker in you machine. To install docker check this website

[Install Docker](https://www.docker.com/get-started)

<h3>Pulling Image and deplying the container</h3>

You can pull docker image from my repository
Here is the link to repository

[repo](https://hub.docker.com/repository/docker/tjan90/event-processing)

To directly pull image give the below command

{%highlight console %}
docker pull tjan90/event-procesing
{%endhighlight%}

### Usage
After pulling the images you need to configure few things to deploy the container
 - dataset foler
 - videostream

Here is command to deploy the container

{%highlight console %}
docker run --privileged -e DISPLAY=docker.for.mac.host.internal:0 -ti --net=host --ipc=host -v /tmp/.X11-unix:/tmp/.X11-unix -v /your-dataset-folder-here/:/app/data/ -v /Your-video-stream-folder-here/:/app/recognition/ eventprocessing 
{%endhighlight%}

Put the desired application folder in the above command
