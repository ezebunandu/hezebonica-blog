+++
title = "Engineers Cosplaying Lawyers Part 1 -- An Open Source Software License Primer"
date = 2025-09-02T23:29:11Z
draft = false
slug = "engineers-cosplaying-lawyers-an-open"
description = "Let us imagine that like me you also identify as a non-developer developer — someone who knows enough about programming to be dangerous."
canonicalURL = "https://ezebunandu.substack.com/p/engineers-cosplaying-lawyers-an-open"
originalURL = "https://ezebunandu.substack.com/p/engineers-cosplaying-lawyers-an-open"
tags = ["substack-migration"]
+++

> Let us imagine that like me you also identify as a non-developer developer — someone who knows enough about programming to be dangerous.

Let us imagine that like me you also identify as a non-developer developer — someone who knows enough about programming to be dangerous. You have been asked to build an automation project to solve a need for your team. Unlike other automation projects you have worked on in the past, this one would actually ship to “production” — It will serve customer traffic in some form in your production environment. You set off in panic and excitement to build the thing. To help you get started, you search your favorite code repository for a package or library that might be helpful. Aha, this must be my lucky day you think. You have found just the right package that does most of the heavy lifting for solving your use case. You open up your code editor, import the library and get coding with your favorite LLM companion to guide you.

If you are anything like I was a few weeks ago, you probably didn’t even bother to read the license file for the library. Oh but Sam, It is open source. The code is right there on GitHub for anyone to use in whatever way they please. But is being on GitHub really all it takes to make a project open source? Could it have a license that might make it unattractive for us to use in a project that might be shipping to production? What even is open source software? What is a license and why should you care what the text of the license of a project on GitHub stipulates? What types of licenses might you want to avoid and what makes them particularly pernicious? Well, buckle up your seat belt and let us go on an adventure learning about open source licensing.

#### **What Even is Open Source Software?**

If you asked me a couple of weeks ago to describe what makes a piece of software open source, my answer would almost certainly have centered around the fact of the source code being publicly available, for example on GitHub. While not entirely off the mark, such a description tells only part of the story. It takes more than merely having the source code publicly available to make a piece of software open source. Putting your code up on GitHub is like leaving your car parked on the street. Just because you’ve left it out doesn’t mean your neighbour gets to drive it around. You would have to grant them permission to do so. The license for a piece of software is the equivalent of granting permission. “Yes, you can use my car, but subject to certain terms and conditions.”

The license for a project that is open source grants quite a bit of liberty when it comes to the driving of the car out on the street. It allows the user of the software to use, study, change, and distribute the software, but still there are terms and conditions. Life is all about terms and conditions, after all. What might those terms and conditions be? Before I tell you that, let me first tell you about copyright as a form of legal protection for software intellectual property. .

#### **LIC101 Software Copyright and Open Source Licenses**

Software copyright is a form of legal protection that grants the creator of software exclusive rights to the thing they have created. They can use, distribute, and modify their code however they please, which is quite commonsense when you think about it — it is your thing so you can do whatever you want with it. You do not have to go through any formal process to obtain the copyright to software you create. You get it for free and for as long as 70 years from the date you create the software. As the creator of the software, you may also choose to grant others right to use it through a license.

A license is **a legal instrument governing the use or redistribution of software**. It is like the written note from your neighbour granting you permission to drive the car they’ve left out on the street. You would very much want to know what that written note says in case you get pulled over by the cops. For open source software, what the license has to say about what you can and cannot do with the software falls into two broad categories — permissive licenses and copyleft license. Both of these categories still grant the user the right to use the software with a reasonable degree of liberty. The difference really comes in when a user of the software makes changes to it and want to distribute these changes.

#### **Permissive Licenses and The Freedom to Play**

A permissive license is the most basic type of open source license. These licenses are called permissive because they grant a user almost complete freedom in what they can do with it. As a user of a piece of software with a permissive license you can do almost anything with it, including changing it and sharing those changes for free or a fee, provided you always acknowledge that you have included the software. You can think of permissive licenses as saying “You can do anything you want with this software but there are no warranties. And if you decide make changes that you share or distribute, please acknowledge us, thank you.”

The most popular permissive open-source licenses include the MIT, Apache 2.0 and the BSD Licenses. The MIT License is perhaps the most used open source license, likely because of its brevity, simplicity and clarity. It is only two paragraphs long. I don’t know about you, but I haven’t read too many legal documents that are nearly as brief. Let’s not even talk about the essay in tiny font that ships with my credit card statement. It is long, it is dense, and seems intended to confuse. The MIT license is not like this.

#### **Copyright Meets Copyleft**

With a copyleft license the user can still use the software freely. They can also modify the code and distribute work that includes these modifications, so long as the new work is also licensed under the same or similar open source terms. The spirit and intent of copyleft licenses is to ensure that any piece of work derived from the originally licensed work does not get hidden behind the paywall of being proprietary.

The most popular copyleft license is the GNU General Public License (GPL), which was published by Richard Stallman’s Free Software Foundation in 1988. The latest version of the GPL is the GPLv3. The Free Software Foundation also introduced variations of the GPL such as the Lesser General Public License (LGPL) and the Affero General Public License (AGPL). If you encounter any of these licenses in an open source project, know you are looking at a project that is copyleft licensed.

All copyleft licenses are not created equal. Some copyleft licenses, such as the Mozilla Public License (MPL) and the Eclipse Public License (EPL) are considered “weak” whereas others like the GNU Affero General Public License (AGPL) and the General Public License (GPL) are considered “strong”. What does this strength really imply?

#### **Strong vs. Weak Copyleft**

The strength of a copyleft license refers to the degree to which it is able to influence how code into which it is incorporated should be licensed. With strong copyleft licenses, such as the GPL and AGPL, any program that incorporates it must also be licensed under the same or compatible license. Weaker copyleft licenses like the LGPL and the EPL allow the project to be incorporated into software with other types of licenses, including proprietary ones, either by dynamic linking or by keeping it in a file of its own.

One other major difference between the various copyleft licenses is how they treat the distribution of software. What happens when software isn’t distributed in the traditional sense, but instead delivered as a cloud service?

#### **Distribution is The Trigger**

I mentioned earlier that the differences between permissive and copyleft licenses really come into play when a user makes a change and wants to distribute these changes. What does it mean to distribute software?

Distribution refers to transferring a copyrighted work (such as software) from one legal person to another. There are a variety of means by which this transfer can accomplished. For example, I can distribute software to you by copying it in a CD and sending it over by post, by making it available on a server to be downloaded or even by storing it in a special kind of web server called a container registry.

Why does distribution matter when talking about open source software? For one, most open source licenses have conditions that are specifically triggered by the distribution of the software. An open source license might say “here are certain terms and conditions, but they don’t apply if you have not made any changes or even if you have made changes but don’t share them outside certain boundaries.” Distributing changes is the trigger for the terms and conditions. For example, most open source licenses have nothing to say if you are make derivative work available only internally within your organization.

As of 1991, when GPL v2 was released, most software was on-premise distributed software. When the technology world started moving to cloud and SaaS services, some people in the open source community were concerned that the way technology changed created a loophole in the license. GPL v2 defined distribution as actual bits of code being transferred to a user’s machine. Thus creators of software products that were delivered as product over the internet were not considered as distributing software to their users and hence could legally avoid the more restrictive requirements of the copyleft license. During the drafting of GPL v3, there was a lot of discussion about the potentially expanding the definition of distribution to also include delivering software as a managed or hosted service. Divisions in the community at that time caused this provision to be excluded, but an alternative license, the AGPL 3, was later developed to close this loophole.

#### **More on the AGPL License and License Compliance**

We now know that copyleft licenses can be problematic because they are more restrictive than permissive licenses, that distribution is the trigger for the terms of an open source license and that the AGPL3 license was invented to address particular questions around distribution for SaaS and cloud hosted software. A license compliance program is how a company might put policies and procedures in place to ensure they only use software they are authorized to use in their products they distribute.

In part two of the blog post, I will delve into more details about the AGPL license, including certain commonly held misconceptions. I will also demonstrate a continuous integration workflow for enforcing license compliance in an automated way using GitHub Actions and Trivy.

## References

#### Articles

- [Open Source Licensing — What every technologist should know](https://opensource.com/article/17/9/open-source-licensing)
- [Heather Meeker on the AGPL License](https://fossa.com/blog/oss-license-compliance-expert-heather-meeker-agpl/)
- [What is Open Source — Opensource.com](https://opensource.com/resources/what-open-source)
- [TheNewStack — What Happens to Relicensed Open Source Projects and Their Forks](https://thenewstack.io/what-happens-to-relicensed-open-source-projects-and-their-forks/)
- [Open Source Licenses: Types and Comparison](https://snyk.io/articles/open-source-licenses/)
- [Deconstructing Software Intellectual Property](https://www.yorku.ca/osgoode/iposgoode/2021/07/05/intellectual-property-of-software-laws-and-protections-for-developers/)
- [Wikipedia — Copyleft](https://en.wikipedia.org/wiki/Copyleft)
- [Understanding Copyleft Licenses and Their Purpose](https://www.pingcap.com/article/understanding-copyleft-licenses-and-their-purpose/)
- [Software Freedom Law Center Guide to GPL Compliance](https://softwarefreedom.org/resources/2014/SFLC-Guide_to_GPL_Compliance_2d_ed.html)

#### Podcasts

- [Whiskey Web and Whatnot -- Breaking Down Open-Source Licensing with Carlo Piana](https://pca.st/1xfjpd3t)
- [About Licenses -- Regular Programming](https://pca.st/ger5xw7m)
- [Scaling DevTools — How not to do Open Source Licensing, with Trigger.dev](https://pca.st/00bdm0d0)
- [Tim Corey’s DevQuestions — Misconceptions Around Open Source Licensing and Monetization](https://pca.st/rnga2brt)

#### YouTube Videos

- [CppCon2015 — A Crash Course in Open Source Licensing](https://www.youtube.com/watch?v=cJIi-hIlCQM)
- [Stanford eCorner — Comparing Open-Source Software Licenses](https://www.youtube.com/watch?v=DDx6gjwU0K8)
- [Pro Tech Show — Open Source License Types](https://www.youtube.com/watch?v=nFU8KoSgEmk)
- [Nextcloud Conference 2017 — Free Software Licensing in a nutshell](https://www.youtube.com/watch?v=LO9klGURyFE)
- [From Project to Profit: Commercial Open Source Software Licensing and Business Models](https://www.youtube.com/watch?v=Ck1gJIZ3Lr4)
- [Open Source Software Licensing Basics for Corporate Users — Heather Meeker](https://www.youtube.com/watch?v=gF4b1TA5Q5w)
- [Stockholm Platform Summit — Open Source as a Business](https://www.youtube.com/watch?v=e5dP9ytn7gA)

Thanks for reading x Through Beginner's Eyes! Subscribe for free to receive new posts and support my work.
