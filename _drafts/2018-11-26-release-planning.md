---
layout: post
title: Release Planning
tags: [Outline, Release Management]
---

# Thesis

Successful releases require a combination of long and short term planning reflecting internal and external priorities

# Long Term Planning

* Target architecture funnel - the precision of the architecture can only decrease the longer the time horizon
* Identify work
  * Accumulated back log
  * Customer requests
  * Technical Debt Payments
  * Focused improvement (e.g. developer experience, performance)
  * Customer feedback and data (e.g. UX improvements)
* Focus on coarse-grained features and problem spaces
* Draw relationships between the work to identify dependencies
* Different release cycles
  * Early in a product cycle, the demands are low and the priorities are driven by complexity, interdependency, ...
  * Once features start to land and we gain some perspective on feature complete, we can target a minor release for the product and start RCs
  * Focus then turns to burn down major bugs to get to a product release
  * After minor release, weekly patch releases to stabilize the product

# Tools

* VS Code
* Graph Paper
* Gliffy + Confluence Wiki

# Short Term Planning

* Not about driving work but how to decide what work to do
* Currently, we pluck work onto a kanban board
  * high touch - requires regular management which requires mental expense
* Trying to shift to Epic-driven
  * Select a couple epics that are the highest priorities and drive those to done before pulling more in