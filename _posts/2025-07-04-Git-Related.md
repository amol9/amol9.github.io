---
layout: post
title: Git Related
---

The technical work on the website has continued. I have pulled my repo, amol9.com from github. I issued "git clone" for it. The cloning took place. Then, I switched the branch to 'gh-pages' to access the website pages. But, on that branch, there was nothing, or, rather only the single file from the branch, 'master' was showing. I tried various commands, "git pull", etc. They didn't work. This git repo business may be complicated, or is it always that? With the versioning software. Anyway, I got it done after an Internet search. I got the solution, "git checkout origin/gh-pages". After that, the website files were present, locally.

The plan with the website is to switch to a static approach. I am not sure how this works, with github, I have only tried the template based approach, which I gleaned from some tutorial, which may be from the github website itself. I am going to try a static approach now. I am not sure if it is supported. But, it may be so, most likely. It must be possible to host static html files on the github repo and have them rendered for a domain. The github software might just transfer the static html files, without processing, to be shown in the browser.

