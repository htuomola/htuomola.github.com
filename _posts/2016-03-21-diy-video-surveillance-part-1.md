---
layout: post
title: "DIY video surveillance system, part 1"
date: 2016-03-21 22:30:00 +02:00
tags: "development azure iot"
---

Back during the summer I thought that it'd be interesting (and not that hard) to setup a motion-activated surveillance camera and have snapshot still pictures shown on a private web site. Now the system is has been up and working for a while so I thought I'd write about it here. This is the first part of a series on what was done & how.

## Goal

Project goal was to show a feed of web cam images on the internet. The system should be motion-activated (no-one is actively watching the feed) and cheap to maintain. Some development effort was perceived but that was also planned to be kept to a minimum. Architecture should be simple but fault-tolerant so that downtime on other components but the camera shouldn't cause loss of images. Instead of big design the idea was to get the first version up and running quickly and improve from there onwards.

## Architecture

At it's core the architecture is very simple, camera sends pictures to an online storage (FTP), from where they are processed and then shown on a website. As Azure is the cloud I'm familiar with, the system is built on Azure, utilizing as many PaaS (Platform-as-a-Service) services as possible. This helps keep the maintenance effort - and often also cost - low.

After fighting a while with the camera's FTP functionality - and to make the system more robust - I've added a local FTP server to act as interim image storage. The FTP server is hosted on a Raspberry Pi. The Pi also runs a worker daemon that uploads all images to Azure Blob Storage. The images on the Blob Storage are then picked up by an Azure App Service Web Job, which writes the data to Azure DocumentDB. Following Microsoft's guidance, the Web Job is not monitoring for new blobs but a message is sent to an Azure Storage Queue when new image has been uploaded. That data is then shown in a website hosted as Azure App Service Web App. The web app itself has been built with Node.js, Angular and TypeScript.

Architecture and the flow of data is illustrated in the below image.
![System architecture](/assets/article_images/2016-03-21-Home surveillance architecture.png)

## Hardware

### 4G Wifi router

In order to get an internet connection to the site, we got the Huawei E5377T 4G mobile hotspot. It was rather cheap (100€) and actually has a surprisingly good range. With the current setup I was seeing a range of maybe 20 meters, through one wall. The router has battery but that's not really needed in my setup as it's constantly connected to a power supply.

### Camera

To start with, we needed a web camera which is capable of uploading images to an online service. After some investigation, I settled on a camera called Opticam O4 (it seems to be a whitelabel product for Verkkokauppa.com, probably made by Foscam), which has Wifi and is capable to sending images via FTP and email. The camera also consumed most of the budget, costing a few hundred euros.

### FTP Server & worker host

As I already mentioned above, the FTP server and image upload worker runs on a Raspberry Pi (2 Model B). Currently the Pi actually is located at our home but the plan is to place it close to the camera, allowing for queueing of images in case the internet connection is down.

This concludes the first post. The following posts will be about the worker and data storage implementations and the UI.