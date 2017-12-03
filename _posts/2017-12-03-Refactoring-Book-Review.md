---
layout: post
title: Refactoring Book Review
tags: [book-review]
---

[Refactoring by Martin Fowler](https://martinfowler.com/books/refactoring.html) has been around for a while now, but I have only recently gotten around to reading it. The book has sat on my Amazon Wish List for years, but every time I looked at it, I told myself I would read it later, that I didn't really want to refactor anything right now so it was ok to put off reading the book for the time being. Whether or not you see the flaw in this train of thought, please keep reading.

A few months ago at work, I was tasked with working on the companies [Android application](https://www.williamhill.us/mobile/). Other than a few test projects at home, I had never developed on Android before, but I do have Java experience and an Android phone, so that made me the top pick for backfilling an open Android developer spot, at least for the time being. I had seen a few snippets of the codebase and had heard rumors about it, but have never actually pulled the repo down and looked through it. When I finally did, I was amazed. The code was a mess. Years of different developers adding bandaids, coping/pasting code, misunderstanding of Java intricacies, and a whole lot of spaghetti code. 

As I was working through the tasks assigned to me, I tried to refactor any method that I touched so that it would be easier to work with in the future. Somewhere during this process, I remembered the book Refactoring by Martin Fowler and finally decided that now was the perfect time to pick the book up.

The book starts off with a fifty page example of how to refactor a class that prints receipts for movie rentals. It is very well explained and throughout the explanations, it references types of refactoring methods that are described in detail, with examples, in another section of the book. I read through the chapter flipping from the case study to the refactoring descriptions, and by the end, I was sad it was over. I wanted more refactoring! Bring it on! 

The next chapter takes a step back and discusses what refactoring is, and the history behind it. Next, chapter three goes into detail about what code should be refactored, by giving an overview of code smells, long classes, temporary fields, etc.  The book was written in 1999, and even though technology is drastically different from what it was 18 years ago, almost all of the topics covered in the book apply today. All of the examples in the book are in Java, but they can easily be applied to any object oriented language. 

A large part of the book describes methods of refactoring ranging from very basic concepts such as "Extract Class," to much more complicated concepts such as "Tease Apart Inheritance". When I first started reading the book, I figured that I would glance over the different concepts and then use it as a reference when needed on a day to day basis. Instead, I ended up going through each one, even the ones that I usually already do, because it helped reinforce the concepts that I know, and better understand the ones that I usually don't use.

While reading the book, I found myself refactoring more and more at work. I kept taking the concepts that I read about and then applying them with ease to the convoluted Android project that I was working on. It made a huge difference. Slowly, the classes that I touched looked better and better and it became easier to reuse methods and read what was actually going on. 

At one point in the book, Martin mentions that code as we see it is not meant to be interpreted by computers, it is meant to be read by humans. The final code will be compiled into byte code for the system to use, but on a day to day basis, it needs to be read and interpreted by humans that work with the code. I had never really thought of it this way. I have always tried to write code as clean and reusable as possible, but have never stopped to think about how much of a difference it makes. It has been a while since I have worked on a large legacy project, and I had forgotten just how bad code can look. I am now amazed at how big of an impact a little refactoring can make.

Refactoring by Martin Fowler is a great book and a wonderful reference tool. Even if you only work on new projects and don't have to worry about digging into messy legacy piles of code, it is still worth reading so that you can construct your code correctly from the start, and continue to improve on it once it is in production.


