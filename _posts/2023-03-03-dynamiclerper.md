---
title: Dynamic Lerper
date: 2023-03-03 03:00:00 -0500
categories: [Regret, Interpolation]
tags: [unreal,ue4,ue5,interp,interpolation,lerper,c++]
---

# What is Dynamic Lerper?

Most of the times that we want to do an interpolation, we have to go through the same overwhelming process of implementing interpolation, handling delta time, updating values, etc...

By creating this helper class, any interpolation could be done using this class.

> There are two lerper classes available:
[Normal Lerper](../lerper) & [Dynamic Lerper](#dynamic-lerper)
{: .prompt-tip}

> Lerper implements a Lerper_Type which defines the behavior of the lerper: Default, PingPong
{: .prompt-tip}

# Dynamic Lerper

``` cpp
enum class Lerper_Type : uint8_t
{
	Default = 0,
	PingPong = 1
};
```