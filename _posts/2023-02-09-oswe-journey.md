---
layout: post
title: "I Cracked OSWE at 18"
categories: "non-technical"
---

My journey and review of AWAE/OSWE (WEB-300) by Offensive Security.

![OSWE Certificate](/assets/oswe-certificate.png)

## Backstory

One of my 2022 goals was to get started with source code review and crack the OSWE. I did not get out of my comfort zone for almost eight months as I kept telling myself I’d do it after graduating high school. In August, I graduated from high school and was ready to dive into the realm of code review.

One month of basic learning in August, and the journey got postponed again due to the NullCon Goa trip in September. In Nepal, October is the month of festivals (along with me turning 18) so that was another excuse to postpone the course. Meanwhile, I took one month of PentesterLab and did a lot of code review challenges during my free time [103/107 challenges]. Until November 18th when I finally decided to get the AWAE course materials.

## Introduction

Advanced Web Attacks and Exploitation (AWAE) is the course material offered by Offensive Security and Offensive Security Web Expert (OSWE) is the certification you can get after passing the exam based on AWAE. The course focuses on teaching you how to find vulnerabilities from a completely white-box perspective. It covers vulnerabilities that are almost impossible to find in a black-box approach, but also covers black-box testing to some extent as well. More information about the course and exam can be found [here][course-link].

I bought the course for $1649 and now it is available at $1599 🤷—unlucky me.

### Prerequisites

AWAE course material assumes that you have prior knowledge of web vulnerabilities, and programming languages i.e, have the ability to read code written in different languages and write basic scripts in a language of your choice, to automate the whole exploit in one go.

## Courses and Labs

The 90-day time was a total overkill. 60 days would have been perfect to complete all the exercises and extra miles easily without being strangled by time. Offensive Security needs to bring back to 30 and 60 days of course time.

I spent the first month doing almost nothing. In the second month, I had to get serious. I was able to complete all exercises and extra miles, and do the exercise myself without any help from course materials by the end of the second month. I spent half of the third month revising what I learned and tried to learn similar vulns in real-world scenarios to get the most out of the course.

The extra three labs were really fun to do. Those labs are there to prepare you for the exam. They do not contain any solutions or walkthrough and you are expected to complete it by yourself. It took me around 3 days to complete two white-box labs with multiple pathways to RCE. Initially, I did not intend to solve the black box lab but due to having too much free time, I solved the black box lab. Took half an hour to pwn it.

## About the Exam

The exam is of 48 hours with two boxes to bypass auth and get RCE on with full access to the application’s source code. Another 24 hours is provided to write a detailed report on the vulnerabilities found and PoC along with screenshots for each vulnerability.

My exam was scheduled at 7:45 PM. Students are asked to join 15 minutes earlier to do all the proctoring stuff and by 7:45 PM, I was ready to dive right into the machines.

### Setup

A laptop with a single screen (yeah all laptops have a single screen; by screen, I am talking about monitors) with Manjaro as the operating system (Arch based distro btw).

### Box 1

After taking some time to familiarize myself with the exam environment, I set up all the necessary debugging setups and was ready to jump into the first box. By 12:00 AM, I was able to complete a chain of vulnerabilities leading to auth bypass. The chain was really fun. Automating it took me around 1 hour due to the nature of the bug. By 1 AM, I was done with the auth bypass on the first machine. I grabbed the flag, submitted it and went to sleep.

Next day, I woke up at 7 AM, got ready and sat for the exam at 7:45 AM. The RCE was quite straightforward, and I was able to get a reverse shell by 9:00 AM. With breaks in between, I chained both auth bypass and RCE into a single script so that one command can give me the reverse shell at around 11:00 AM.

### Box 2

At around 12 PM, I started the second box. The auth bypass for this was a little bit tricky, but thanks to Offsec’s exam objectives section, I was able to figure out what to look for. At around 5 PM, I was able to bypass the auth, and completed the automation by 6 PM. I was able to get RCE at around 8:00 PM. Again, RCE was straightforward here too. I pwned both machines in 24 hours. Having gotten all the machines flags along with automation by 11 PM, I went to sleep.

### Report

Next day, I woke up at 8 AM and joined the exam at around 9 AM. I spent the whole day collecting screenshots and writing reports with lots and lots of breaks in between. I didn't want to end the exam even though I had everything needed for the report. I re-ran my scripts again and again to make sure it worked without breaking in between. I checked my flags and proof screenshots more than 10 times, as Offsec is very strict with those. At 7:30 PM, the exam ended and I submitted the report by 8 PM. The extra 24 hours for report writing provided by offensive security was used to write this blog post instead.

### Exam Review

The auth bypass on both machines were really interesting and I learned a lot of stuff even while giving the exam. I had to push myself to get those flags. However, the RCEs were pretty simple compared to auth bypass. The only thing you needed to know was where to look for an RCE ;)

### FAQ

Here are some of the FAQ-styled answers to generally asked questions.

**What did you learn?**

I have become quite good at debugging. I learned how something as small as a '=' can lead to massive vulnerabilities, and most of the times which cannot be found using black-box approach.

XSS to RCE was fun. Using XSS to exploit a CSRF vulnerability that can lead to RCE is definitely fun. I have become really good with manipulating objects to exploit stuffs, thanks to Prototype Pollution and SSTI exercise.

I also loved the in-depth exploitation of blind SQLi in the course, and how SQLi can lead to RCE in certain cases. Blind SQLi has become one of the most exciting vulnerabilities for me to exploit in the white-box scenario. The meme below sums up my experience which I posted on [Twitter][twitter] as well.

![SQLi Meme](/assets/sqli-meme.jpg)

Having automated all the auth bypass + RCE in all of the exercises, I got my Python automation skills really sharpened. A huge codebase does not feel intimidating anymore. There are many more but you get it.

**Should I take this course?**

If you are someone who loves torturing yourself with loads of knowledge and skills, this is for you. But seriously, if you are into source code review and white-box testing or are looking forward to getting into it and have enough cash to spare, this is a no-brainer. I would go one step further and recommend this to bug bounty hunters and black box testers out there as well to get a comprehensive overview of how vulnerabilities look on the other side.

Remember that AWAE only covers a few vulnerability discovery skills and jumps into exploitation most of the time. If you want to be able to find vulnerabilities by looking at a specific piece of code, go for PentesterLab’s [code review badge][code-review-badge]. It is a goldmine.

**Value for money?**

I don't have a specific answer to this question and I am not-so-good at money stuffs. The course is really expensive but it also offers great playground to sharpen your skills and play around with. I haven't found a course equivalent to this yet. If your goal is just vulnerability discovery in whitebox scenario, I believe PentesterLab offers much more than AWAE in terms of ROI. However if you want to actually exploit that flaw you just found, chain it up and create an exploit, you will find this course fun. This certificate is also a great resume booster, hopefully.

**What’s up with that title?**

The reason I added this question is because I myself am not too comfortable with the title I’ve put. Since all the cool kids are putting their age everywhere to show that they are young (absolutely not implying that it’s bad in any way), I also decided to let the world know that I cracked OSWE at 18.

By the way, I asked Offsec whether I was the youngest to pass the OSWE exam. Sadly they refused to answer. I am positive that there are other 18 y/o as well who cracked this exam. Not that it makes any difference in being the so-called youngest OSWE holder ;)

### Ending Thoughts

On passing the exam, it felt extremely fulfilling as I had this goal pending for over a year. I really enjoyed the course materials and the exam and they met my expectations. The exam was an extra-fun ride. I got to learn a lot of cool stuff and that’s all that matters at the end of the day.

Thank you for reading!

_Fun fact: I started writing this blog before attempting the exam, and completed this blog after the exam but before the results were out ;)_

[course-link]: https://www.offensive-security.com/courses/web-300/
[twitter]: https://twitter.com/dhakal_ananda/status/1621357754534486017
[code-review-badge]: https://pentesterlab.com/badges/codereview
