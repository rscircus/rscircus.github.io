---
categories:
- Code
date: "2023-10-06T00:00:00Z"
excerpt_separator: <!-- more -->
sub_title: Practicing Software Engineering in a manner that minimizes the cognitive burden for developers.
tags:
- Software Engineering
- Cognitive Load
- SWE
title: Philosophy of Software Engineering and Design
---

We find ourselves in an era defined by rapid technological advancements. So much that the term "developer" or "programmer" was replaced with "Software Engineer". Reflecting the natural arts and craft approach of similar occupations like mechanical engineer or civil engineer. Software development has been transformed from the mere act of code creation to an intricate dance of design, maintenance, delivery, and reliability.

As organizations grapple with increasingly complex software ecosystems, there is a growing need to cultivate a broad perspective.  

<!--more-->

In this essay I want to approach Software Engineering with humans and time as guides instead of machines as guides. By delving into Michael Feathers' "*Working Effectively with Legacy Code*", John Osterhout's "*A Philosophy of Software Design*", the insights from "*Continuous Delivery*" by Humble and Farley, and "*Site Reliability Engineering*" authored by the Google engineers Beyer, Jones, Petoff, and Murphy, we can extract a coherent philosophy that can guide modern software practitioners.

## A Coherent Philosophy of Software Design

![](https://rscircus.github.io/assets/img/20231006_SWE_BestPractices_Books.jpg)

My favorite book of these four mentioned and being pictured above is John Osterhout's "A Philosophy of Software Design". The reason is that "A Philosophy of Software Design" designs software with focus on simplicity and evolvability. At the heart of Osterhout's perspective is the notion that complexity is the primary impediment to effective software design. He posits that the *majority of costs in software development are rooted in complexities that are unintentionally introduced*. Deep modules, which have a lot of functionality but expose a simple interface, are advocated as the hallmark of great design. These encapsulated modules lead to more robust and maintainable systems, and the **focus should always be on reducing cognitive load for developers**. Let me repeat:

> The focus should always be on reducing cognitive load for developers.

Clear interfaces, minimal operational complexity, and intentional design can dramatically reduce future costs and lead to more efficient and scalable systems.

## The Future is Built upon the Past

At the core of Feathers' work elaborated in "Working Effectively with Legacy Code" is a pressing reality: Much of the software we engage with is not new, shiny, or designed with modern best practices. Instead, it carries the weight of decisions made years ago.

> Much of the software we engage with is not new, shiny, or designed with modern best practices.

To work effectively with legacy code, one must first understand it, *safely make changes without introducing defects, and incrementally refactor to improve its structure*. The focus is the very definition of legacy code: Code without tests. So it is worth it to regard legacy code as an asset rather than a liability, embracing and improving it while ensuring its continued functionality. 

## Automation and Feedback Loops

Let's refocus on practicing Software Engineering in a manner that minimizes the cognitive burden for developers. In "Continuous Delivery - Seamless and Reliable Software Releases" Humble and Farley shift the conversation to the critical juncture where code moves from development to production. These days, with the omnipresent cloud, a significant chunk of time is spent upon infrastructure. 

> CD provides a continuous feedback loop, allowing developers to detect and rectify issues promptly.

Continuous Delivery (CD) emphasizes automating the build, deployment, and testing processes to produce software in short cycles, ensuring it can be reliably released at any time. CD provides a continuous feedback loop, allowing developers to detect and rectify issues promptly, leading to more stable releases and increased developer and user satisfaction. In essence, it not only reduces the cognitive load on developers but ensures that software is always in a releasable state, bridging the gap between development and operations.

## Maintainability and Performance

More often than not, software interacts with hardware. And hardware is hard. "Site Reliability Engineering - Merging Development with Operations" originating from Google's approach to large-scale service management, Site Reliability Engineering (SRE) is about applying software engineering practices to operations.

> *"Blameless Postmortems: When incidents happen, the focus is on learning from them rather than placing blame."*

Beyer et al. describe how to maintain high availability and performance while allowing for rapid changes and growth. Central to SRE is the Service Level Objective (SLO), ensuring that systems meet users' reliability expectations. Moreover, error budgets and risk management play crucial roles, promoting *a balance between innovation and stability*. Topics like Capacity Planning, Load Testing, and Monitoring. Understanding the capacity needs of a service, testing its limits, and consistently monitoring its performance are crucial components of the SRE methodology.

> Balance between innovation and stability.

## Tying it All Together

While each book presents its unique insights, they collectively converge on a few key ideas. Firstly, there's an overriding emphasis on simplicity, be it in code design, deployment, or operations. Simplicity reduces errors, expedites processes, and facilitates understanding.

> Simplicity reduces errors, expedites processes, and facilitates understanding.

Secondly, automation and feedback loops are crucial in modern software practices, allowing for swift response to issues and ensuring consistent quality. Lastly, there's an acknowledgment of the evolving nature of software: whether dealing with legacy systems or designing new architectures, we must anticipate change and design for adaptability.

In conclusion, while the tools, languages, and infrastructures may change, the principles of good software design and delivery remain rooted in simplicity, maintainability, and reliability. By assimilating the insights from these seminal works, you as a software practitioner can be better equipped to navigate the dynamic landscape of software development in the modern era. And I highly recommend them and hope I have created some appetite for more.
