---
layout: post
title: Thinking(deeply) about Immutability
categories: functional programming
---
Immutability is something that a lot of people care about today, but why? What happen in the world of programming languages design to introduce immutability as a necessary feature? Is it  only related with the functional paradigm? These and another questions will be analyzed in the very first post of my blog. Firstly, let's go back in time.

## The mutable Hardware

In past decades(00's and 90's specifically), the hardware was growing according to Moore's Law, that concludes that the number of transistors in a dense integrated circuit doubles each 2 years. This particular tax was very accurate along the time, but currently there are emerging some theories that indicates [the end of Moore's Law](https://rodneybrooks.com/the-end-of-moores-law/). You can see that with the move from single core architectures to parallel multi-core architectures and clustered processing environments.

![moore_law_end]({{site.baseurl }}/public/moore__sl_large.gif)

Parallelism, in fact, is the new growing direction for processors architectures. You can disagree, but this has big impact in the way we should work as developers. The first scenario you should consider, it is the fact that now not only one processor manages memory, but n(_n_ being a natural number) processors, resulting in a scenario where a space in memory could be requested by two or more processors simultaneously having a direct performance impact. In traditional _stateful_ programming paradigms that we used to work, we use mutable variables a lot, without care much about parallelism. Even if you use threads, this _fighting processors_ scenario is possible to happen and another problems like deadlocks, livelocks, code imprevisibility(haha!) and much more.

Think a solution design having immutability as a constraint, lead to a situation where these problems don't exist, because it implies in a scenario where memory is not shared between nodes or threads. Even you think that this can overuse your memory resources, currently there is a tendency in software to modularize the application in many stateless modules(microservices, web-components), and immutability in this aspect is not a very big deal.

## Thinking in transformations

Nowadays, with the popularity of functional programming, many software developers begin about software as sequence of transformations. Thinking in that particular way is only possible with a design that considers immutability, that leads to a software without side-effects. Thanks to immutability you can enjoy the possibility of break your code in many different pure functions and execute it in many possible environments.

![](https://camo.githubusercontent.com/c2c0ba1ad82d003b5386404ae09c00763d73510c/687474703a2f2f692e696d6775722e636f6d2f72764352394f512e706e67)
*Time travel debug, one great wonder that is possible because of immutability*

But how it exactly happens? How immutability will improve my code quality? These questions are not so easy to answers because immutability is not a thing only related with code design, but it's related with other things like hardware and distributed systems. So, in order to help you visualize in a practical way, it will be showed a basic example involving calculations.

Our code example will have to do the following steps:

* Receive a single integer number
* Get 25% of it's value and round down
* Check if it is greater than ten and return this as a boolean value

In simple JavaScript we can do as the following code:

```javascript
const number = 67;
console.log(checkValue(calculateQuarter(number))); // (1)
// Output: true

function calculateQuarter(number) { // (2)
    return parseInt(number * 0.25);
}

function checkValue(number) { // (2)
    return number > 10;
}
```

In the (most) simple code from above we can check the following points:

(1) This particular line of code is not the most beautiful thing that you see in the world, but if we use a more serious functional programming language than JavaScript, it would be possible for use do things like:

```
number
|> calculateQuarter
|> checkValue
```

It's very easy to check that this is a modular approach, and at the end the result would be a sequence of transformations into `number` value.

(2) Both `calculateQuarter(number)` and `checkValue(number)` are pure functions, they are not accessing any value that is external to its context. This can allow you to execute them in any possible environment, you can put one function in one machine and the other anywhere and the code will not be affected by it. Due to immutability, the value and reference that come in the function are different to the ones that it returns.

The code example was very simple, but necessary to show that immutability is a necessary issue in modern software development scenario, and it's necessary to understand it before diving in many good new things that are emerging today.

## Good reads

To finish up my first post I want to share some interesting articles for you read and think more about immutability. The articles are the following:

*  [Immutability Changes Everything; Pat Helland](http://cidrdb.org/cidr2015/Papers/CIDR15_Paper16.pdf) : This paper analyze immutability in many different areas of software development and it's very useful to make you realize how it have great direct impact in everything.

*  [Transformation for Class Immutability; Fredrik Kjolstad, Danny Dig, Gabriel Acevedo, Marc Snir](http://people.csail.mit.edu/fred/immutator.pdf) : This paper cames as glove for those which have a OO background and wants to understand the nuances of immutability in objects.

Thanks for coming up until here, I hope you have enjoyed reading.
