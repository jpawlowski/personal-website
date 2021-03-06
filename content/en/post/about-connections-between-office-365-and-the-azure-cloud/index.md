---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "About connections between Office 365 and the Azure Cloud"
subtitle: ""
summary: "Explaining what the Azure Cloud has to do with Office 365 and Microsoft 365 services."
authors: []
tags:
- Azure Cloud
- Microsoft 365
- Office 365
- Azure AD
categories: [ "IT-Operations" ]
date: 2019-12-27T14:09:49+01:00
lastmod: 2019-12-27T14:09:49+01:00
featured: true
draft: true

#TODO remove this fix for ToC generation as soon as bug in Hugo 0.60+ was fixed
# see https://sourcethemes.com/academic/docs/writing-markdown-latex/#table-of-contents
markup: blackfriday

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: "Smart"
  preview_only: false
  #placement: 3

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

When I was writing my blog post [Confusion Over Microsoft Licensing Terminology]({{< relref "confusion-over-microsoft-licensing-terminology" >}}), I thought there is also a need to explain something about Microsoft's Azure Cloud.

{{% toc %}}

### Where Azure originated from

Generally spoken, Azure is Microsoft's cloud data center and every service they offer as part of their own online products runs on Azure. If you already understood how Amazon came into the AWS business, it is similar with Azure: Microsoft wanted to bring their existing on-premises software like Exchange and SharePoint into a managed service ([SaaS](https://en.wikipedia.org/wiki/Software_as_a_service)). Naturally, they had an extraordinary need to scale their online services and had to invent new technology to cut software into pieaces that would run more independant from hardware and even whole data centers. Remember that before, Microsoft had only operated data center for their own needs, but not as a full-service provider for their customers. This was a new challenge as now they became responsible and accountable to run and operate their own software.

Over time, a lot of those cloud computing services were shaped into own services that Microsoft now offers for their customers to implement their own business applications. The real benefit is not about just lift and shift your existing virtual servers on-premises into the Azure cloud. The benefit comes up when you re-build and re-think your business processes and even your own products to build cloud-native applications that are designed to run independant from servers (aka _serverless_, which of course is technically not correct but indicates that you no longer have to care about infrastructure, only architecture).

Undoubtly this main concept of a business cloud was pioneered by Amazon AWS.

### Where the confusion starts

_"What does Azure have to do with Office 365 and stuff?"_, you might ask. Where does this question come from?

**Azure Active Directory**

Service wise, there is only one single service that
