---
layout: post
title: "Introduction to Neural Networks"
categories: ai
excerpt_separator: <!--more-->
imageid: datanet.jpg
---

Neural Networks are the leading strategy in many Artificial Intelligence applications, and at their core, they are surprisingly simple. Today we'll build a single layer neural network <!--more-->in python, then extend that out to several layers and understand what is happening at each stage.

### First, a summary of Neural Networks.

The basic building block of the neural network is the perceptron, first developed around 60 years ago, was actually designed as  piece of hardware, meant to recognise patterns in images and classify them. The perceptron is essentially a set of inputs with weights applied, fed into a unit step function.

![Perceptron](/assets/images/postimages/perceptron.png)

The idea is that if the weighted sum of the inputs is high enough, i.e. if the inputs are strong enough, then the inputs will pass the unit step function, so to speak. A very basic unit-step function looks like this:

![Unit Step Funcion](/assets/images/postimages/unit-step.png)

Notice the function outputs only two values, 0 or 1. If the inputs are not strong enough, no matter how close they get, it will not pass the unit-step function. This early model was somewhat inspired by how researchers thought our own neurons work. 
For example, if you touch something that may hurt you, such as a stove (the input), the temperature of that stove (the *weight* of the input) will need to be high enough to make you pull your hand away. If the input is not high enough, the stove may just be warm and you can keep your hand there all day. However, at some point, the weighted input is high enough, i.e. it gets hot enough, that you have no choice but to pull your hand away. That is the basic working of the perceptron and the unit-step function.

Think of this in a more abstract way with other possible inputs. If you're considering whether or not to touch something, your brain may consider a few things:
1. Is it hot?
2. Is it sharp?
3. Is it poisonous?
4. Is it dirty?
5. Do I want to touch it?

In the case of the stove, the input from each of these is vastly different. A stove is unlikely to be poisonous or sharp, so the weights applied to those inputs would be close to zero.
It may be dirty, covered in grease or otherwise unpleasant to touch, and it may be very hot, so the weights on those could be zero or much higher.
Finally, you may know it is dirty or too hot to touch, but do you want to touch it anyway? This is an important point to understand. The weights applied to the five inputs above, are not necessarily about their own properties. I.e. a hot stove may have higher weight input than a cold one, but what's more important is **how its temperature contributes to your decision making process.** That is what determines the weight of the input - **how does that input contribute to your decision making process?** 
The *total weight* of the inputs should be 100. That means if the stove is hot as hell, covered in grease, rusty and sharp, and you touch it anyway because you want to, the weights of inputs 1-4 are extremely low, if not close to zero, while the weight applied to input 5, do I want to, is very high. 
If you really want to touch it but restrain yourself because of reasons 1-4, the weight applied to 5 is low, or at least lower than the combined weights applied to 1-4.
This will become clearer in practice, although it's important to keep in mind. In researching some otherwise successful neural networks, researchers have found some inputs hold almost all of the weight - i.e. they contribute to almost all of the decision making. This is not usually a sustainable practice, especially in complex decision making processes that should take input from many sources.

Coming soon - Implementing a simple neural network in Python.