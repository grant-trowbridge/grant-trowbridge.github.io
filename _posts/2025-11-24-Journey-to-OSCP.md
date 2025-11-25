---
layout: post
title: "Journey to OSCP"
---

Having recently passed my Offensive Security Certified Professional exam, I wanted to give back to the OffSec community and share my three year journey towards OSCP. You heard that right, not only *three* years, but passing on my *fourth* attempt as well. It hurts the ego to say that out loud, but I feel it is important to be fully transparent, especially for those out there considering beginning your own journey, or perhaps feel like giving up.

# In the Beninging

I first heard of the OSCP exam during a Cyber Security club presentation in college from the one and only [Connor McGarr](https://www.linkedin.com/in/connor-mcgarr-7522a6152/). After watching his buffer overflow demonstration, and learning of the OSCP exam's difficulty, I knew I was hooked. At this time, I was still a fairly green Cyber Security student and didn't consider preparation or further investigation until a year later.

## Exploring Offensive Security

My first dive into the topic began in my sophomore year of college with the free [Practical Ethical Hacking Course](https://www.youtube.com/playlist?list=PLLKT__MCUeixqHJ1TRqrHsEd6_EdEvo47) YouTube playlist offered by Heath Adams. The course has been updated in recent years, but even back then I felt it was a great primer of the different stages of penetration testing, and appropriate tools to be used for each stage. Even at the end of the course, I still felt out of my depth and didn't attempt any CTF boxes at the time. My note taking methodology needed **a lot** of work, I wasn't seriously considering OSCP down the road. In reality, I remained a casual observer. This would change drastically my senior year of college.

## Purchasing PEN-200

Halfway through my senior year in 2021, I decided to take my career seriously and push myself to learn more about offensive security. I figured if I could pass my OSCP after graduation, it would set me up for a job. This turned out to be true, but not in the way I thought it would. I picked up my coursework during OffSec's end of the year sale for a slight discount and began taking notes as fast as I was able to. At this time, I modeled my notes after the chapter structure in the PDF itself. Remember when I said my note taking methodology needed work? This is going to be a common theme throughout this post.

I recall completing the PDF entirely about one month before graduation and spent the rest of my time practicing in the labs and revisiting extra mile challenges from the PDF. The exam purchase only included 90 days of lab access which left me rushing to prepare as much as I could, essentially grasping for straws. I scheduled my exam for June of 2022 and nearly prepared every day leading up to my exam.

### Attempt #1

Since I had no idea what to expect, I spent the entire night staring at the ceiling with my mind racing. I considered all possibilities and imagined every topic I felt underprepared for appearing on the exam. What if HTTP is blocked on a machine and I have to transfer files another way? What if I have to exploit blind SQLi? etc, etc. Needless to say I was wound tight. After grabbing breakfast, I connected to the exam VPN, followed all prerequisite steps, and was off to the races. I decided to focus on Active Directory first, which at the time did not have assumed breach credentials. I actually managed to gain an initial foothold, and then escalated privileges to spawn a system shell on the Windows target! I don't recall how long this took, but I can assure you it wasn't impressive. I went back through my notes and saw the next step was to dump credentials. I transferred the necessary tools to the target, dumped hashes, but I wasn't able to find credentials that could provide access to the next machine in the AD network. What was I missing??

I spent the rest of the exam toiling on the Windows machine, retracing my steps and looking for something, *anything* that may provide access to the next AD machine. I would occassionally review my `nmap` output from the other exam machines, did some basic enumeration, but never found any initial access. I felt defeated, and figured it was time to lay down and get some sleep.

I woke up around 5AM to revisit the machines for one last ditch effort, but it was futile. I knew this attempt was a fail. I decided to not waste time on drafting a PDF report, and laid down to catch up on some sleep.

#### The Silver Lining

Remember how I said this certification would help set me up for a job? It did, but more specifically the *failure* set me up for my current job! Believe it or not, I was hired on to [Scientific Research Corporation](https://www.scires.com/) roughly one month after my first OSCP attempt. I didn't discover this until a year and a half in when I decided to ask my boss:

> Why did you hire me?

I was aware that my situation was unusual. It is rather uncommon for a new graduate with no prior DoD experience to be working Cyber Security right out of the gate. I can't remember what he said word for word, but I clearly remember him mentioning my OSCP exam attempt as something that separated me from the rest of the candidates. To his credit as well, he wanted to give a fresh graduate a chance in Cyber Security, which I am eternally grateful for. The lesson for the reader: 

Your efforts, even in failure, do not go unnoticed. Hard work and dedication is always valued.

### Attempt #2

At this point, I began working full time at SRC starting on July 11th, 2022. OSCP was still in the back of my mind, but my full attention was now on learning the ins and outs of my new job. I knew I wasn't ready to give up, so I saved over a few months to purchase my PEN-200 Learn One subscription on December 28th, 2022.