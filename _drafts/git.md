---
layout: post
title: "intermediate git"
---

### git 101

my early professional career required that i knew how to do six things
in git: branch, stage, commit, merge, push and pull. beyond that there
was always google. and of course that stack overflow page that
everyone stumbles on eventually: if i fucked something up there was
`git reset --hard HEAD`, and if i really fucked it up i could do `git
reset --hard HEAD~`.

to my surprise now, i got a lot of leverage out of just those six (or
seven) commands. but that was probably because no-one else really
minded what i was doing. we committed to master and dealt with
problems as they came up. no-one read the history. we pushed to a
gitolite server, which, as great as that is, is so far away from the
world of github that to any novice it was something of a black
box. code got committed and pushed. who knows what happened after
that? if something broke, it meant doing more committing and
pushing.

fortunately this didn't last for too long as i decided at some point
that i needed to understand git a little better.

i still don't consider myself an expert in any way. i did give a talk
on the subject at work recently which i enjoyed, and wanted to
summarize more formally here. so, here it is: the guide i wish i had
read a couple years ago to get me through the git blues.

first of all, let's define a few terms:

#### private branch

a branch that is used by just you. pushing it to a remote does not
necessarily make it public.

#### public branch

a branch that is shared (read: committed against) by many.

#### HEAD

i always wondered if you were supposed to scream this. i might less
formally refer to it as simply 'head' or the 'head of such and so a
branch'. it is simply the current revision of a given branch.

#### the graph

a lot can be said about the graph, and it's probably beyond the scope
of this article to talk about this in any detail. let's just say that
a requirement for understanding git's internals is some rudimentary
knowledge about graph theory. i really do mean rudimentary, so don't
let that put you off. there is a great resource on explaining git in
terms of graph theory here[think-like-a-git], which i would highly
recommend.

in terms of graph theory, your git history is essentially a graph
composed of commit 'nodes'. the commits at the HEAD of branches are
your 'leaf' nodes. HEAD in this sense refers to the series of changes
(i.e. commit nodes) that are 'reachable' (i.e. pointed to by HEAD, or
pointed to by commits that are pointed to by HEAD, an on and on).

#### merge bubble

when you merge two branches, you will get a merge 'bubble' by creating
a new commit in the target branch that retains the integrity of both
branches. this is a special 'merge commit', and it's special because
it points to two different in the history - the tip of the target
branch, and the tip of the source branch. you wouldn't create this
commit by hand, it will happen automatically depending on how you've
set up your `.gitconfig`.

#### fast-forward

this is what happens when you merge without creating a merge
bubble. git will merge your changes in at the top of your target
branch as if you had just been committing to it (e.g. `master`) all
along. no merge commit is created.

#### squashing

this is a technique used for combining commits that have already been
made into bigger, more consolidated ones.

### some committing anti-patterns

here are some things that can generally go wrong:

- putting everything into one big commit
- writing an incomplete commit message
- breaking something. committing. fixing it later.
- (more advanced) rebasing or committing in hunks without checking the
  state of each commit

one thing i learned early on was that it is a good idea to commit
frequently. unfortunately that's not the whole story. although it does
address anti-pattern #1, it will often mean trading it for #2 or
\#3. practicing TDD is actually conducive to making frequent, small
commits because you're concentrating on getting to green (a
requirement for a good commit) without getting distracted or writing
more code than is needed. essentially, it's OK to do #2 or #3 as long
as you're working in a private branch and you squash or rewrite your
commits before merging by performing an interactive rebase (more on
this later).

squashing everything isn't necessarily a good idea either. the goal
should be to be left with a small number of commits that mark a
distinct progression toward some goal (adding a new feature,
refactoring, etc.). as you become more savvy with rebasing
interactively you may fall prey to antipattern #4. in other words,
when you're rewriting history it's important to check the integrity of
each commit that you're creating after the fact. if you really care
about your history, and not just your HEAD, you'll want every commit
to be green and deployable.

there are actually a few reasons why you might want to take such care
of your history. the first that comes to mind is being able to use
git's `bisect` feature with more confidence. `bisect` is a tool used
for examing a portion of your history, typically for locating a commit
that introduced some regression. it is a very powerful and useful tool
that i've personally seen rendered completely useless by careless
committing. more on `bisect` later.

another reason might be being able to generate metrics for your
application across a range of commits.

another is simply being able to read your history with relative
ease. this is more a comment on composing good commits with good
commit messages. (occasionally, for inspiration, i'll go spelunking
through the history of some open source software that i love, go right
back to the first commit and rediscover the steps of creating its
first complete feature by a more skilled practitioner.)

there are two rules i like to follow when composing a commit
message. the first is to use the present tense imperative in the first
line. the reason for this is that this is the tense/mood used in git's
generated messages such as on merge commits. a nice side effect of
this is that you will probably find that your messages are shorter and
succinter. the second rule is never to use the `-m` flag. trying to
fit your entire message onto the first line is just way too much
pressure! how formal you want to get with your message after that is
up to you. generally it's a good idea to have a short, descriptive
first line, followed by a longer description and a link to an issue
number or ticket if one exists. i add tim pope's template to my config
to help remind me:


```
# ~/.gitconfig
[commit]
  template = ~/.gitmessage
```

```
# ~/.gitmessage


# 50-character subject line
#
# 72-character wrapped longer description. This should answer:
#
# * Why was this change necessary?
# * How does it address the problem?
# * Are there any side effects?
#
# Include a link to the ticket, if any.

```

### more on your gitconfig

there are a couple more things that you may want to consider adding or
tweaking in your gitconfig. often you'll see official advice telling
you to use the git command line interface to accomplish this, but i
prefer to edit my `.gitconfig` by hand. you can do that by typing:

```
$ <editor of choice> ~/.gitconfig
```

here are a few things i recommend playing with:

```
[alias]
  a = add
  br = branch
  ci = commit
  co = checkout
  st = status
```

these are a few simple and common aliases that have become more or
less standard (see that git wiki article for others). i won't
enumerate all the ones i use here, but feel free to check out my
[dotfiles]. aliasing is essential to being productive if you
interacting with git at the command line. feel free to create aliases
in your `.bashrc` too. alias `git` to `g`, and more common commands
such as `git status` to `gs`. it might seem trivial at first, but you
type `git status` about 200 times a day as do i, you are going to be
saving quite a few keystrokes by the end of the week. and that's time
you could be spending thinking about your design, or even going for a
walk in the park.

```
[push]
  default = current
```
words go here.

```
[merge]
  ff = only
```

and here.

```
[branch]
  autosetuprebase = always
```

here also.

### rebasing

if you only learn one thing beyond the git 101 stage it should
probably be this. never rebase a public branch! i said never rebase a
public branch!

Linus Torvalds has said that all of git can be understood in terms of
`rebase`. but i think there's another command that helps illuminate even
further: the `cherry-pick`.

this is what a cherry-pick looks like:

```
$ git cherry-pick <commit>
```

what it does is apply the changes introduced by a given commit to the
head of another branch. if that sounds confusing, or if you've never
really thought about git in those terms, go back and read that a
couple of times.

`cherry-pick` is sort of the basic unit of a `rebase`. the difference
is with `rebase` you're saying: take this series of commits and
_replay_ them at another point in history.

this is what a rebase looks like:

```
$ git rebase master
```

with interactive rebasing you have even more control over how to
rewrite history. you can take commits out, shuffle them around, squash
commits into other commits, stop the replay right in the middle and
change something and continue where you left off. powerful stuff.

this is what an interactive rebase looks like:

```
$ git rebase -i master
```

there are two distinct benefits that you get from rebasing. one is
that you can introduce any upstream changes into your code, address
any breakages or refactoring that can be done, then merge all your
changes directly onto the tip of master, without a merge 'bubble', as
if you had just written them in some kind of coding frenzy. the other
is that you can commit however you want while you're developing, and
then go back and recompose your commit history into a string of coding
pearls, squashing smaller changes, typos and errors, and writing
beautiful commit messages with love and care.

one thing you might notice is that if you were pushing your topic
branch before you rebased, when you try to push after the remote will
refuse and complain about it too. this is normal and to be
expected. it just means that you have to 'force' push your branch.

the reason for this is that you changed history by rebasing. these
words are often thrown around when people are talking about rebasing,
but the explanation can be a little vague.

when you rebase a branch onto another commit, you take that first
commit you made when you first branched off and point it to a
different commit (typically the HEAD of the base branch). doing so
actually creates a new commit with a distinct SHA1 hash, and points
HEAD to it. your original commit is still there, it's just not visible
in your log any more because it's not reachable from HEAD.

the next commit in your project branch is now pointing at this ghost
commit. it needs to be updated to point to its new parent. the process
begins again. a new commit is created, HEAD is moved, and on and
on. as the rebase replays all your changes, it effectively changes
every commit hash in the branch. your local branch and origin now have
two different copies of the same changes but none of the hashes is the
same. this is why git gives you the somewhat confusing indication to
pull your changes down before trying to push. what you need to do
instead is tell the remote to forget everything and just accept your
local branch in place of whatever it has. and that looks like this:

```
$ git push -f origin <branch>
```

### some useful things to know

#### reflog

for the longest time i held the reflog at arm's length. i knew it
existed and that it could be of help if you were in serious
trouble. maybe there was some security in thinking that if i managed
never to use it then i could never have done anything _that_ bad.

but i was wrong. the reflog is actually exciting, powerful and pretty
straightforward.

```
$ git reflog
$ git reflog show <branch>
```

#### ranges

ranges can be pretty confusing because they can mean different things
in different contexts. it's important to know how to use them, though.

```
# git log
# commits that b has that a doesn't have
$ git log <commit a>..<commit b>
```

```
# commits in a and b but not both
$ git log <commit a>...<commit b>
```

```
# git diff
# changes between commit a and commit b
$ git diff <commit a> <commit b>
```

```
# same
$ git diff <commit a>..<commit b>
```

```
# changes that occurred on a's branch since it branched off of b's
$ git diff <commit a>...<commit b>
```

```
# git checkout
# checkout the merge base of a and b
$ git checkout <commit a>...<commit b>
```

#### commit parents

sometimes it can be easier to refer to commits not by their SHA1 hash
but by their relationship with another commit. this is especially so
when dealing with recent history and your point of reference is HEAD,
aka the current revision, or last commit.

```
# the current commit
$ HEAD
$ HEAD~0
```

```
# the 1st parent of the current commit
$ HEAD~
$ HEAD~1
```

```
# the 1st parent of the 1st parent of the current commit
$ HEAD~~
$ HEAD~2
$ HEAD~1~1
```

```
# the 2nd parent of the current commit
$ HEAD^2
```

```
# uh...
$ HEAD~2^2~5^2
```

#### add

you already know how to do that, right? try adding in hunks.

```
# stage changes in hunks
$ git add -p
```

#### bisect

```
# start it all off
$ git bisect start

# mark a known good commit
$ git bisect good <commit>

# mark a known bad commit
$ git bisect bad <commit>

# tell bisect the commit it checked out is good
$ git bisect good

# tell bisect the commit it checked out is bad
$ git bisect bad

# automate it
$ git bisect run rspec spec/features/my_broken_spec.rb
```

#### blame

my FAVORITE tool. in all seriousness though, this can be useful in
situations where you have some code you really don't understand
despite your best efforts, and you need to have a chat with its
author.

```
$ git blame path/to/file
```

#### revert

```
# create a new commit reversing the changes
$ git revert <commit>
```

```
# revert a merge
$ git revert -m 1 <merge commit>
```

#### reset

Moves HEAD to the specified commit

```
# leave changes in previous HEAD in staging area
$ git reset --soft HEAD~
```

```
# leave changes in previous HEAD in working tree (default)
$ git reset --mixed HEAD~
```

```
# destroy all changes in previous HEAD
$ git reset --hard HEAD~
```

```
# reset to previous commit
$ git reset --hard <commit>
```

```
# reset to previous point in the reflog
$ git reset --hard <branch>@{<reflog entry>}
```

```
# reset to where you were last week
$ git reset --hard <branch>@{one.week.ago}
```

### references

1. Linus Torvalds tech talk: https://www.youtube.com/watch?v=4XpnKHJAok8
2. Think Like a Git: http://think-like-a-git.net/
3. thoughtbot rebase like a boss: http://robots.thoughtbot.com/rebase-like-a-boss
4. Git ready: http://gitready.com
5. Destroy all Software: https://www.destroyallsoftware.com/screencasts
6. http://typicalprogrammer.com/linus-torvalds-goes-off-on-linux-and-git/
7. tim pope's commit message template
