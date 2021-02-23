---
title: Think twice, cut once
description: 
date: 2019-02-15
tags:
  - skills
  - engineering
  - coding-style
layout: layouts/post.njk
image: /img/pexels-pixabay-34520.jpg
---

Sometimes it seems to me that tools are getting too important. Don't get me wrong, I really appreciate those little helpers modern IDE's and their plugins are offering. For example if they're suggesting to me a method might be rather static instead of const. Or not all paths of a lengthy, chained and nested If statement are returning something. Especially with legacy code those tools are very helpful.

![Hero Image: Hammer and Wrench, Foto von Pixabay von Pexels](/img/pexels-pixabay-34520.jpg)

But sometimes I stumble upon code where you can immediately see that the tools quick offered help was easier to apply then thinking twice about a problem. I would go so far to say that this behavior shows up for every developer sometimes, especially under pressure. "This feature has to be finished till the end of the week, we promised the customer." "But the sources are barely beyond prototyping state and not cleaned up at all. Well ok I let the tools do their work, so it's at least in a certain cleaned up state." Sounds familiar? If I'm honest, to me it does. Is it bad? Well, the tools do their job, they do what their made for and they do nowadays a really good job.

But still, trusting them blindly can, sooner or later, lead you into problems. And the price to pay might be higher than you think upfront. For example, I've seen code where the tool exchanged types with auto as much as possible. At the end it was not a problem with buggy code, it was more a problem of readability. The code was now partly much harder to understand because the types became obfuscated. That wasn't the intention of introducing auto into C++, for sure not.

A more subtle, but in my opinion much more dangerous, problem occurs with transforming methods into const methods blindly, only because a tool is suggesting it. A const method is not really changing the binary code a compiler is producing (if I'm wrong with my assessment feel free to correct me). It's more to promise to the developer, and his companion and/or users of his library, that a call of a particular method won't change the state of a class instance. If a method `foo()` of a `class Bar` is const, I could call it as many times I want, it wouldn't change the state of an instance of Bar.

But let's have a look at this little and simple code snippet:

```cpp
class MyClass {
public:
  int * getArray() const { 
    arr[0]++;
    return arr; 
  }
  void setFirstVal(int val) { arr[0] = val; }

private:
  int * arr = new int[10];
};
```

A class with an array as member. We will just focus on the const, nothing of all the other problems `MyClass` has. That class was "optimized" by a tool, and it's correct from the compilers point of view, no errors, no warnings, awesome. If we would just see the interface, or let's say the contract, `MyClass` is offering, we would think that `getArray` is harmless and has no side effects. But unfortunately `getArray` has a side effect we would probably realize to late. The method is incrementing the first element of the array every time `getArray` will be called. How can that be if this method is const? There where no compiler errors nor warnings. And in principle it's correct. The pointer to the first array element is never changing, but it's content is. But this means the objects state has changed, does it? Well, if you see only the pointers, the objects state didn't change. If you know more about the implementation you will realize it's not true what `getArray` is promising, not changing the objects state.

This way the tool is sabotaging our encapsulation we want to accomplish and preserve with our interfaces and contracts. We would need to know the internal implementation of `MyClass` to use it correctly. Thinking twice, before we trusting a tool blindly and cut the rope which is preventing us from falling into the abyss, is, in my opinion, one of the more important skills, every engineer should practice.