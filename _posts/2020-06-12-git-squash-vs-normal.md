---
layout: post
title: "Git - Squash Merge, Normal Merge 차이"
category: Git
tags: [git]
---
> git merge command에는 크게 rebase, normal, squash merge가 존재하며 그 중 squash, normal merge의 차이에 대한 commit graph

## Squash vs Normal
![merge]({{ "/assets/images/git-squash-1.png" | absolute_url }})

## Example
![actual]({{ "/assets/images/git-squash-2.jpg" | absolute_url }})

### Using Squash
![result_squash]({{ "/assets/images/git-squash-3.jpg" | absolute_url }})

### Using Normal
![result_normal]({{ "/assets/images/git-squash-4.jpg" | absolute_url }})