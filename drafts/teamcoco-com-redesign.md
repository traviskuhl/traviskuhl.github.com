---
layout: default
title: teamcoco.com redesign
class: post
---

{{page.title}}
================================

Earlier this month with launched our latest redesign of TeamCoco.com. While the most noticeable change to our users is a new visual design, the changes I'm most excited about are under the hood and behind the scenes. The new site runs on a 100% rewritten codebase, a brand new infrastructure, a modern CI process and a completely automated deployment system.

In it's short four year history, TeamCoco.com has gone through several iterations. Our last major redesign was a little over a year and a half ago and in that time a lot has changed in world of web development. And while the old codebase was generally stable, it was beging to show signs of it's age. Rather than try to patchwork update the legacy codebase, we instead desided to start from scratch.

As we began planning out the rewrite, we had three main goals in mind:

1. **Standardize** pick a framework and stick with it. improve code reuse
1. **Modernize** framework. streamline CI process. improve test coverage. automated build & deployment process. more efficient auto-scaling.
1. **Keep (and improve) what works** flexible database schema. simplicity of architecture. development process.

The five biggest changes to the code base/infrastruture ended up being:

1. PHP Framework
1. Front End Framework
1. Shareable components
1. Modern CI process
1. Automated Deployment System

## 1. PHP Framework
As TeamCoco.com has grown and evolved over the past year and a half since the last redesign, the PHP codebase had became very fractured. Before the rewrite, we were using a total of five different PHP frameworks across the different layers of the site (two custom PHP frameworks, Symfony, Silex & Drupal).

These fractured implementation choices lead to very little code reuse across our different layers. Whenever we added new features to one layer using one framework, we then had to reimplement that same code in the other frameworks, on the other layers.

In additional to the multiple frameworks, we were using a tangled mess of Pear, Pecl and a custom package manager to manage dependencies. This lead to a lot of headaches with installation and environment setup. Getting a new environment up and running was a complicated process, involving way to many steps.

The new code base is built completely on a single framework, [bolt](https://github.com/boltphp/core). With all of our layers using the same framework, we've been able to centralize a lot of our code into shareable component packages. We've seen the biggest gains in our Model components.





