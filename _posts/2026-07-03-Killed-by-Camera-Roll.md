---
layout: post
title: "How to Get Killed by Your Camera Roll"
---

At some point in your life you've heard the word "metadata" in the news. But what is it exactly? Who would want your metadata? And why should you care? In this blog post, I will be using a real-life photo as an example of what kind of data can be extracted from your camera roll pictures. The image is from a Tyler Childers concert I attended with my sister back in 2023 (and we'll be able to confirm this further down).

# What is Metadata and Why Does it Matter?

Simply put, metadata is data about data. It is information about content that is not contained within the content itself. Think of it as reading the nutrition facts label on a gallon of milk at the grocery store. But what if instead of containing proteins, carbs and fats, this metadata contained information about your device, when a file was created, and where the device was at the exact time of file creation. That could be valuable to advertisers, law enforcement, even foreign militaries and intelligence agencies.

## "We Kill People Based on Metadata"

A direct quote from former head of NSA, General Michael Hayden, during a panel debate regarding privacy at Johns Hopkins University in April 2014. I bring this up not to scare you, but rather inform you of what metadata can be used for. From the most "benign", like advertising, all the way to shortening the kill chain to launch pinpoint accurate artillery and drone strikes.

At intelligence agencies like NSA, CIA, MI5/6, and KGB, there exists a job role called Targeting Officer. According to CIA's own job summary:

> As a Targeting Officer at CIA, you will identify the people, relationships, and organizations having access to the information needed to address the most critical U.S. foreign intelligence requirements and find opportunities to disrupt terrorist attacks, illegal arms trade, drug networks, cyber threats, and counterintelligence threats.

What often surprises people is how much OSINT (Open Source Intelligence) is used by law enforcement and the IC (Intelligence Community). Movies often depict advanced satellite imagery and recruiting assets. Sometimes all it takes is an OPSEC (Operational Security) slip-up. A cell phone someone forgot to power-off or place in a faraday bag. A message sent at the wrong time. Or a picture posted to social media.

# Extracting Metadata with exiftool

As stated at the top of the article our example will cover the picture and social media example:

![Exif Example](/assets/img/exif-example.jpg)
