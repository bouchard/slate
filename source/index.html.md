---
title: Voting Integrations (VI) Documentation

language_tabs: # must be one of https://git.io/vQNgJ
  - ruby
  - python
  - php

toc_footers:
  - Back to <a href='https://electionbuddy.com'>electionbuddy.com</a>

includes:
  - v2
  - v1
  - errors
  - webhooks

search: true
---

# Introduction

Welcome to the Voting Integrations (VI) documentation for ElectionBuddy (electionbuddy.com). We continue to maintain both our version 1 (or V1) integrations, as well our version 2 (or V2) voting integrations to allow your voters to authenticate with ElectionBuddy within your own member or customer portal or customer relationship management (CRM) software.

**V1** focuses on providing you with a method to generate a signed URL to send your voters to a specific election.

**V2** expands on this by allowing you to give your voters a list of all running elections in which they are eligible to vote, and allowing them to vote in any or all of them in turn while being authenticated behind your customer portal.

**Webhooks**, in contrast, sends requests to a URL provided by you. This allows integration with services that supoort recieving webhooks, or possibly your own web server, if configuerd appropriately.
