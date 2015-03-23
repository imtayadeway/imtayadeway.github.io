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

#### git _(noun)_

a person who is stupid or unpleasant.

#### private branch

a branch that is used by just you. pushing it to a remote does not
necessarily make it public.

#### public branch

a branch that is shared by many.

#### HEAD

the current revision of a given branch.

### committing

<table class="table table-striped">
    <tr>
      <td>git</td>
      <td>a person who is stupid or unpleasant</td>
    </tr>
        <tr>
      <td>private branch</td>
      <td>a branch that is used by just you. pushing it to a remote does not necessarily make it public</td>
    </tr>
    <tr>
      <td>public branch</td>
      <td>a branch that is shared by many</td>
    </tr>
    <tr>
      <td>HEAD</td>
      <td>the current revision of a given branch</td>
    </tr>

</table>

here are some things that can generally go wrong:

- putting everything into one big commit
- writing an incomplete commit message
- breaking something. committing. fixing it later.
- (more advanced) rebasing or committing in hunks without checking the
  state of each commit

one thing i learned early on was that it is a good idea to commit
frequently. unfortunately that wasn't the whole story, although it
does address antipattern #1.
