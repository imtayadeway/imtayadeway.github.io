---
layout: post
title: "intermediate git"
---

### git 101

my early professional career required that i knew how to do 6 things
in git: branch, stage, commit, merge, push and pull. if something
weird happened, there was always google, and of course that stack
overflow page that everyone stumbles on eventually: if i fucked
something up there was `git reset --hard HEAD`, and if i really fucked
it up i could do `git reset --hard HEAD~`.

to my surprise now, i got a lot of leverage out of just those 6 (or 7)
commands. but that was probably because no-one else really minded what
i was doing. we committed to master and dealt with problems as they
came up. no-one read the history. we pushed to a gitolite server,
which, as great as that is, is so far away from the world of github
that to any novice it was something of a black box. code got committed
and pushed. who knows what happened after that? if something broke, it
meant doing more committing and pushing. fortunately this didn't last
for too long as i decided at some point that i needed to understand
git a little better.

i just gave a talk at work on git and was surprised to find i knew
more about the subject than most of my peers. i still don't consider
myself an expert in any way. anywho, here's the guide i wish i had
read a couple years ago to get me through the git blues.

### intermediate git

first of all, let's talk a few terms:

#### git _(noun)_
a person who is stupid or unpleasant

#### private branch
a branch that is used by just you

#### public branch
a branch that is shared by many

#### HEAD
the current revision of a given branch
