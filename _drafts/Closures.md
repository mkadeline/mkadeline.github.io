---
layout: post
title: "Swift Closures"
date: 2017-10-04
tags: psc
categories: ios
imageid: ios.jpg
listHide: correct
---

Closures are defined as 
> self contained blocks of functionality that can be passed around and used in your code. They're similar to blocks in C or lambdas in othe languages.
> Closures can capture and store references to any constants and variables from the context in which they are defined. This is known as closing over those constants and variables. Swift handles all of the memory management of capturing for you.

The simplest verson of a closure is a global function. They are defined, self-contained and can be passed around by name to other parts of code. They do not however, capture values from their context unless explicitly passed as parameters.
Nested functions have a name and can capture values from their enclosing function.
Closure expressions are unnamed closures that can capture values from their surrounding context.
