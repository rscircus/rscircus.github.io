---
categories:
  - Code
  - Business
date: "2023-02-11T00:00:00Z"
excerpt_separator: <!-- more -->
sub_title: Microsoft wants all your data :/
tags:
  - ai
  - business
  - code
  - legal
title: All Your Data are Belong to Us!
---

Recently we have seen OpenAI, a company excessively funded by Microsoft, trialing a Large Language Model (LLM) called ChatGPT. Guess what? They are selling you your own data.

<!--more-->

It is one of the first Neural Networks (NN) one can basically chat with and have somewhat of a dialogue. The remarkable thing about ChatGPT is that it has an unusually convincing opinion about any topic. To train such a LLM one needs enormous amounts of data. It is somewhat kept in the dark what data ChatGPT was and is trained on. However, there are some leaks by internal sources which make sense considering the accuracy with which ChatGPT is able to talk about any topic. One of the data sets is called `books2`; it is basically a collection of most of the pirated books there are. [0]

That is an interesting fact from a legal perspective.

The sketchy thing with LLMs or NNs in general is that storage and compute somewhat smear into each other and it is basically impossible to tell where what is stored or where what is computed. One could say that the data _and_ the compute are compressed and encrypted in the NN. ChatGPT being a Transformer type of architecture [1], its compression part becomes even more obvious.

Why would Microsoft trial such a legally dangerous thing in a somewhat unrelated company called OpenAI for a while? It is entirely in the realm of possibility to assume that they wanted to see if the world would react in any legal way to this fact. The world did not. And now ChatGPT basically flows into any product of Microsoft.

How can it be that there are companies who run their entire data on Microsoft's premises, from communication to document storage and operating systems, with GitHub being owned by Microsoft and their AI coding assistant GitHub Copilot having a fathomable track record [2] of being trained with and regurgitating copyrighted code?

Who in their right mind would do this?

_Sources:_

- [0] among others, these are interesting
- [0] [1] https://www.searchenginejournal.com/how-to-block-chatgpt-from-using-your-website-content
- [0] [2] https://www.reddit.com/r/ChatGPT/comments/10xfksf/can_anyone_confirm_this_list_100_of_sources_used/
- [0] [3] https://www.reddit.com/r/ChatGPT/comments/10xfksf/comment/j7uk8x4/?utm_source=reddit&utm_medium=web2x&context=3
- [1] https://rscircus.github.io/2020/02/22/transformer.html
- [2] https://www.google.com/search?hl=en&q=copilot%20copyrighted%20code
- [x] In case you wonder about the article's title: https://en.wikipedia.org/wiki/All_your_base_are_belong_to_us
