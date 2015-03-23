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

first of all, let's define a few terms:

#### private branch

a branch that is used by just you. pushing it to a remote does not
necessarily make it public.

#### public branch

a branch that is shared by many.

#### HEAD

the current revision of a given branch.

### committing

here are some things that can generally go wrong:

- putting everything into one big commit
- writing an incomplete commit message
- breaking something. committing. fixing it later.
- (more advanced) rebasing or committing in hunks without checking the
  state of each commit

one thing i learned early on was that it is a good idea to commit
frequently. unfortunately that's not the whole story. although it does
address antipattern #1, it will often mean trading it for #2 or
\#3. practicing TDD is actually conducive to making frequent, small
commits because you're concentrating on getting to green (a
requirement for a good commit) without getting distracted or writing
more code than is needed. essentially, it's OK to do #2 or #3 as long
as you're working in private branch and you squash or rewrite your
commits before merging by performing an interactive rebase (more on
this later). squashing everything isn't necessarily a good idea
either. the goal should be to be left with a small number of commits
that mark a distinct progression toward some goal (adding a new
feature, refactoring, etc.). as you become more savvy with interactive
rebasing you may fall prey to antipattern #4. in other words, when
you're rewriting history it's important to check the integrity of each
commit that you're creating after the fact. if you really care about
your history, and not just your HEAD, you'll want every commit to be
green.
