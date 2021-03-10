---
title: Weekly Knowledge Candy
description: Post about our knowledge transfer process.
date: 2019-03-21
tags:
  - eductation
  - skills
  - engineering
layout: layouts/post.njk
image: /img/dreamstime_xl_2033016_original3.jpg
---

Welcome back to a new post at thoughts-on-cpp. This time I will write about something different, not quite technical and C++ related. Well, not exactly. It's a post about team education, about how to evolve the C++ knowledge of a team which is, in my case, not build up with computer science experts, but purely with domain experts (in my development team we are all mechanical engineers or mathematicians). You may ask, why the hell are they doing software engineering only with domain experts? This will be for sure another blog post, I promise, to talk about this.

![Hero Image: Visualization of knowledge transfer](/img/dreamstime_xl_2033016_original3.jpg)

I'm working in this company for quite a while, basically since I finished my studies of mechanical engineering. But have I've been prepared for the professional development of desktop applications? Well...... not really. Back in university, we had C++ seminars for about a year, and the fact that we have been taught C++ was quite new in 2008. If you have studied mechanical engineering before, you would have been probably taught how to code in [Fortran][1] 77, or if you're lucky in Fortran 95. And basically, the level of programming in C++ was the same as it was in the good old Fortran days. No classes, no encapsulation, no C++ idioms, just pure procedural programming. They just exchanged the syntax and the compiler, that's it. Clearly, the intention was not to fully educate us in the domain of programming, but I think it's also a bad idea, if not a dangerous one, to give students such a primitive bunch of half knowledge. And that's in principle what you get as programming background in classical engineering areas (electrical engineers might be on a higher level). You end up with a team in which C++ knowledge is very heterogeneous. From pure procedural programming with bare knowledge about the STL, to very experienced/self-educated people who are firm with advanced metaprogramming and memory management topics. So somehow there has to be a way to raise the level of programming knowledge for everyone. Not only the most proactive developers, but also the ones which are simply not able, for so many reasons, to raise their programming skills.

## How Weekly Knowledge Candy Originated

One day I stumbled upon Jonathan Boccara's blog [Fluent {C++}][2]. And there was one small page on the side, called [Daily C++][3]. Johnathan states that you learn the best while your teaching, and I totally agree with my own experience of teaching in seminars. So I thought it would be a good idea to somehow adapt his concept of Daily C++ to our needs and introducing it into the company. Many of his ideas work nicely for us. For example the short 10 to 20 minute presentations and fine granular topics which fit into the time frame. We just felt that having daily presentations, even such short ones would be too exaggerating for us. Once a week would be fine because, with only 17 developers, everyone would have at least a week to prepare him-/herself. To start we carried a list of topics together we wanted to introduce or deepen our knowledge. Not only C++, but also topics about patterns, established architectures, frameworks, tools, but well... mostly C++. We ended up with a list of around 20 presentations topics, such as RAII, Rule of zero/three/five, operator overloading, memory management, observer and factory pattern, and much more. We encourage the developers to pick a topic and prepare it to present in any way they like. That can be on a whiteboard, PowerPoint, purely code, or most often a mixture of several forms. The only requirement is that the topic is documented and self-explanatory. That way developers who haven't been able to attend can read it whenever they like. The second and last requirement is that everyone should try to pick the topic he's/she's the most unfamiliar with. Picking the topic they have the weakest knowledge gives the developers the opportunity to learn and earn the most out of the weekly's because it's not just a simple repetition for them. Contrarily to Jonathan's approach we always gather in a dedicated and equipped meeting room. We experienced an increase in attention and broader discussions afterward. Additional I think a dedicated room is not only helping to focus but also it's much quieter. Noise can really be a big problem in large open-plan offices.

Let's summarize the benefits of the Weekly Knowledge Candy:

- Little easy digestible topics
- Developers train their ability to give presentations and speeches
- 10 to 20 minutes focus helps to understand a topic
- Every developer is focusing on his weakest topics. This way the whole team is gaining the most benefits.

[1]: https://en.wikipedia.org/wiki/Fortran
[2]: https://www.fluentcpp.com/
[3]: https://www.fluentcpp.com/dailycpp/