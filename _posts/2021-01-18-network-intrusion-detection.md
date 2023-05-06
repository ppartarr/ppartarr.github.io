---
layout: post
title:  "Anomaly Based Network Intrusion Detection 🐛"
date:   2022-01-18 21:09:010 +0200
categories: blog
---



As part of the Research Project 2 course at the [Security and Network Engineering Masters @ University of Amsterdam](https://www.os3.nl),  [Philipp Mieden](https://dreadl0ck.net/) and I worked on comparing various ML techniques for network intrusion detection.

<!--more-->

## Abstract
We develop and evaluate a pipeline for extracting and analysing observations from network traffic, using state of the art machine learning and deep learning algorithms to recognise malicious patterns in network communication, without the need for static detection rules.
The evaluation is conducted on the CIC-IDS-2018 dataset, a modern and large scale scenario that includes multiple common attack classes inside a multi-department corporate network infrastructure hosted on AWS.
Our experiments seek to evaluate the detection performance of various algorithms, as well as the impact of different model parameters, to determine which delivers the highest value for security threat analysts.
For this purpose, we compare the results of binary classification, to more fine grained multi-class attack classification, 
and investigate the possibility of knowledge transfer between network topologies as well as limitations for real-world deployment of anomaly- and classification-based network intrusion detection systems.

[Link to the paper](/assets/research/anomaly.pdf)