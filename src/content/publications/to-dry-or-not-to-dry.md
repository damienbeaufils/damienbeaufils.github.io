---
title: "To DRY or not to DRY?"
date: 2018-11-13
pubtype: "Article"
featured: true
description: "A blog post about DRY (Don't Repeat Yourself) principle and when not to apply it."
tags: ["Software Craftsmanship", "Design"]
image: ""
link: "https://www.fluentcpp.com/2018/11/13/to-dry-or-not-to-dry/"
fact: ""
weight: 400
sitemap:
  priority : 0.9
---
You hear it since you started programming: you have to remove, remove and remove code duplication!

Why? If you have ever worked with a legacy project, when there was code duplication, the same bug in the code had to be corrected in several places, which drove you crazy. And I don’t even talk about introducing new features.

Even quality tools like SonarQube tells you about code duplication percentage, with a heavy hint: if you have duplication, it’s bad. And if your manager sees these percentages, he may show up asking you “Why are we having 6% duplication on this project? You need to do something about it!”.

And in the end, they’re right: removing code duplication eliminates code redundancy in order to make your product easier to maintain and to add new features.
It’s the famous principle of DRY: Don’t Repeat Yourself“.
