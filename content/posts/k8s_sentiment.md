---
title: "Kubernetes project: deploy a Sentiment Analysis application"
date: 2020-06-28T07:17:23+02:00
draft: false
toc: false
images:
tags:
  - Kubernetes
  - Cloud
---

![](https://github.com/hhassen/k8s-app-sentiment/raw/master/images/App_sent_analyser_demo.gif)

In this project, I have deployed a Sentiment Analysis application using Kubernetes on the cloud (Amazon EKS).

The application is composed of 3 main components:
1. **Frontend App** : uses React and Nginx to provide user interface
2. **Web App** : developed with Java to communicate with backend
3. **Backend App** : Logic part develeped with Python

Please find more information in my [repo](https://github.com/hhassen/k8s-app-sentiment).

![](https://github.com/hhassen/k8s-app-sentiment/raw/master/images/App_sent_analyser_architecture.gif)


Credits to [Rinor Maloku](https://medium.freecodecamp.org/learn-kubernetes-in-under-3-hours-a-detailed-guide-to-orchestrating-containers-114ff420e882) for the code and the original article.
