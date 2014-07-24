---
layout: default
title: teamcoco.com redesign
class: post
---

{{page.title}}
================================

on aug 1st we launch an all new version of teamcoco.com. while the most noticeable change to our users is the new visual design, the project also represents a ground up rewrite of the entire team coco code base; and a completely new core infrastructure.

from an engineering perspective, we had a few major goals with the redesign:

1. **standardize** pick a framework and stick with it. improve code reuse
1. **modernize** framework. streamline CI process. improve unit test coverage. automated build & deployment process. more efficient auto-scaling.
1. **keep (and improve) what works** flexible database schema. simplicity in infrastructure. development process.

## 1. standardize

### Pick a framework
teamcoco.com has gone through several iterations in its short four year history. the original site was a simple wordpress blog; then drupal; and for the past two years a collection of different PHP frameworks. in total, the old version of the site used five different PHP frameworks (two custom frameworks, Symfony, Silex & Drupal), along with several different service components written in Perl and Node.js. we desprately needed to standardize on a single framework and build as many components on that framework as possible.

the new site is built almost entirely on our new custom framework ([bolt](https://github.com/boltphp/core))

###
while the framework choices were (mostly) the right decisions at the time, as the site has evolved, have so many dispirit frameworks has become a huge headache.


## 2. modernize


## 3. keep (and improve) what works