---
tags: []
title: Airflow AWS
date: 
description: ''
multiple_images: []
image: ''
category: ''

---
Always use the execution date variables provided by Airflow instead of any dates relating to the current time. This decoupling is necessary to correctly deal with delays, backfills, reruns etc..