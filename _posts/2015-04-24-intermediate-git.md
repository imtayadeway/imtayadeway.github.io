---
layout: post
title: "Intermediate Git"
date: 2015-04-24 14:44:21
---

### Git 101

My early professional career required that I knew how to do six things
in git: branch, stage, commit, merge, push and pull. Beyond that there
was always google. And of course that stack overflow page that
everyone stumbles on eventually: if I effed something up there was
`git reset --hard HEAD`, and if I really effed it up I could do `git
reset --hard HEAD~`. Or was it the other way round?

To my surprise now, I got a lot of leverage out of just those six (or
seven) commands. But that was probably because no-one else really
minded what I was doing. We committed to master and dealt with
problems as they came up. No-one read the history. We pushed to a
gitolite server, which, as great as that is, is so far away from the
world of GitHub that to any novice it was something of a black
box. Code got committed and pushed. Who knows what happened after
that? If something broke, it meant doing more committing and
pushing.

Fortunately for me this didn't last for too long. I decided at some
point that I needed to understand git a little better.

Now, I still don't consider myself an expert in any way. I did give a
talk on the subject at work recently which I enjoyed, and wanted to
summarize more formally the contents of that here. So here it
is. Something like the guide I wish I had read a couple years ago to
get me through the git 101 blues. It will cover:

* some standard and some not-so-standard terms
* how to write a better commit message (that old chestnut)
* how to make better commits
* some ways to configure git to make your life easier
* what the hell rebasing is
* a few odd parts of git's syntax
* some lesser-used tools that you might like

### Terms!

First of all, let's define a few terms. I won't define every term,
just a few that are either vague or that I will use frequently
throughout.

<dl class="dl-horizontal">
  <dt>Private branch</dt>
  <dd>
    A branch that is used by just you. Pushing it to a remote does not
    necessarily make it public.
  </dd>
  <dt>Public branch</dt>
  <dd>A branch that is shared (read: committed to) by many.</dd>
  <dt>HEAD</dt>
  <dd>
    I always wondered if you were supposed to scream this. I might less
    formally refer to it as simply as the 'head' or 'tip'. It is simply
    the current revision of a given branch.
  </dd>
  <dt>The graph</dt>
  <dd>
    <p>A lot can be said about the graph, and it's probably beyond the scope
    of this article to talk about this in any detail. Let's just say that
    a requirement for understanding git's internals is some rudimentary
    knowledge about graph theory. I really do mean rudimentary, so don't
    let that put you off. There is a great resource on explaining git in
    terms of graph theory <a href="http://think-like-a-git.net">here</a>,
    which I would highly recommend.</p>
    <p>In terms of graph theory, your git history is essentially a graph
    composed of commit 'nodes'. The commits at the HEAD of branches are
    your 'leaf' nodes. Your current revision in this sense refers to the
    series of changes (i.e. Commit nodes) that are 'reachable'
    (i.e. Pointed to by HEAD, or pointed to by commits that are pointed to
    by HEAD, and on and on).</p>
  </dd>
  <dt>Merge bubble</dt>
  <dd>When you merge two branches, you will get a merge 'bubble' by creating
a new commit in the target branch that retains the integrity of both
branches. This is a special 'merge commit', and it's special because
it points to two different commits in the history - the tip of the
target (typically `master`) branch, and the tip of the topic
branch. You wouldn't create this commit by hand, it will happen
automatically depending on how you've set up your
`.gitconfig`. Typically, if you're working on a team and you haven't
configured git at all, or if you're using the github web interface to
merge branches, you will end up with lots of merge bubbles.
</dd>
  <dt>Fast-forward</dt>
  <dd>This is what happens when you merge without creating a merge
bubble. Git will merge your changes in at the top of your target
branch as if you had just been committing to it all along. No merge
commit is created.
</dd>
  <dt>Squashing</dt>
  <dd>
This is a technique used for combining commits that have already been
made into bigger, more consolidated ones.
</dd>
</dl>

### Some committing anti-patterns

Here are some things that can generally go wrong:

- putting everything into one big commit
- writing an incomplete commit message
- breaking something. Committing. Fixing it later.
- (More advanced) rebasing or committing in hunks without checking the
  state of each commit

One thing I learned early on was that it is a good idea to commit
frequently. Unfortunately that's not the whole story. Although it does
address anti-pattern #1, it will often mean trading it for #2 or
\#3. Practicing TDD is actually conducive to making frequent, small
commits because you're concentrating on either getting to green (a
requirement for a good commit) without getting distracted or writing
more code than is needed, or refactoring in small steps. Essentially,
it's OK to do #2 or #3 as long as you're working in a private branch
and you squash or rewrite your commits before merging by performing an
interactive rebase (more on this later).

Squashing everything isn't necessarily a good idea either. The goal
should be to be left with a small number of commits that each mark a
distinct progression toward some goal (adding a new feature,
refactoring, etc.). As you become more savvy with rebasing
interactively you may fall prey to antipattern #4. In other words,
when you're rewriting history it's important to check the integrity of
each commit that you're creating after the fact. If you really care
about your history, and not just your HEAD, you'll want every commit
to be green and deployable.

There are actually a few reasons why you might want to take such care
of your history. The first that comes to mind is being able to use
git's `bisect` feature with more confidence. `bisect` is a tool used
for examing a portion of your history, typically for locating a commit
that introduced some regression. It is a very powerful and useful tool
that I've personally seen rendered completely useless by careless
committing. More on `bisect` later.

Another reason might be being able to generate metrics for your
application across a range of commits.

Another is simply being able to read your history with relative
ease. This is more a comment on composing good commits with good
commit messages. (Occasionally, for inspiration, I'll go spelunking
through the history of some open source software that I love, go right
back to the first commit and rediscover the steps of creating its
first complete feature.)

There are two rules I like to follow when composing a commit
message. The first is to use the present tense imperative in the first
line. The reason for this is that this is the tense/mood used in git's
generated messages such as on merge commits. A nice side effect of
this is that you will probably find that your messages are shorter and
succinter. The second rule is never to use the `-m` flag. Trying to
fit your entire message onto the first line is just way too much
pressure! How formal you want to get with your message after that is
up to you. Generally it's a good idea to have a short, descriptive
first line, followed by a longer description and a link to an issue
number or ticket if one exists. I add [thoughtbot's template] to help
remind me:


```conf
# ~/.gitconfig
[commit]
  template = ~/.gitmessage
```

```conf
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

[Thoughtbot's template]: https://github.com/thoughtbot/dotfiles/blob/master/gitmessage

### More on your gitconfig

There are a couple more things that you may want to consider adding or
tweaking in your gitconfig. Often you'll see official advice telling
you to use the git command line interface to accomplish this, but I
prefer to edit my `~/.gitconfig` by hand.

Here are a few things I recommend playing with:

```conf
[alias]
  a = add
  br = branch
  ci = commit
  co = checkout
  st = status
```

These are a few simple and common aliases that have become more or
less standard (see that [kernel wiki article] for others). I won't
enumerate all the ones I use here, but feel free to check out my
[dotfiles]. Aliasing is essential to being productive if you're
interacting with git at the command line. Feel free to create aliases
in your `~/.bashrc` too. Alias `git` to `g`, and more common commands
such as `git status` to `gs`. It might seem trivial at first, but if
you type `git status` about 200 times a day as do I, you are going to
be saving quite a few keystrokes by the end of the week. And that's
time you could be spending thinking about your design, or even going
for a walk in the park.

[Kernel wiki article]: https://git.wiki.kernel.org/index.php/Aliases
[dotfiles]: https://github.com/imtayadeway/dotfiles

```conf
[merge]
  ff = only
```

This is useful if you don't want git to create a merge bubble unless
specifically asked to do so. If your branch can't be fast-forwarded,
it won't be merged either until you rebase, or you pass a flag
overriding the above.

```conf
[branch]
  autosetuprebase = always
```

Useful if you are using a rebase-style workflow (more below). With
this set, if you pull from an upstream on a branch where you have
revisions that have not yet been pushed, your unpushed revisions will
get shoved to the front, and no merge commit is made.

### Rebasing

If you only learn one thing beyond the git 101 stage it should
probably be this. Never rebase a public branch! Now, I don't like
making hard and fast rules with exclamatory remarks like that,
particularly because I think they contribute to the fear and
trepidation that surrounds rebasing, and the reluctance to use git's
most powerful feature. Please don't let that put you off. It really is
the only thing you need to remember. Everything else is easy to fix =)

Linus Torvalds has said that all of git can be understood in terms of
`rebase`. But I think there's another command that helps illuminate even
further: the `cherry-pick`.

This is what a cherry-pick looks like:

```sh
$ git cherry-pick <commit>
```

What it does is apply the changes introduced by a given commit
anywhere else in your history to the tip of your current branch. You
can tell it to apply it somewhere else if you want, but that's what it
does with no other args. If that sounds confusing, or if you've never
really thought about git in those terms, go back and read that a
couple of times.

`cherry-pick` is sort of the basic unit of a `rebase`. The difference
is with `rebase` you're saying: take this _series_ of commits and
_replay_ them all, starting at another point in history.

This is what a rebase looks like:

```sh
# rebase against local master
$ git rebase master

# rebase against remote master
$ git fetch origin
$ git rebase origin/master
```

With interactive rebasing you have even more control over how to
rewrite history. You can take commits out, shuffle them around, squash
commits into other commits, stop the replay right in the middle and
change something and continue where you left off. Powerful stuff.

This is what an interactive rebase looks like:

```sh
$ git rebase -i master
```

There are (at least) two distinct benefits that you get from
rebasing. One is that you can introduce any upstream changes into your
code, address any breakages or refactoring that can be done, then
merge all your changes directly onto the tip of master, without a
merge 'bubble', as if you had just written them in some kind of coding
frenzy. The other is that you can commit however you want while you're
developing, and then go back and recompose your commit history into a
string of coding pearls, squashing smaller changes, typos and errors,
and writing beautiful commit messages with love and care.

One thing you might notice is that if you were pushing your topic
branch before you rebased, when you try to push after the remote will
refuse (and complain about it, too). This is normal and to be
expected. It just means that you have to 'force' push your branch.

The reason for this is that you changed history by rebasing. Now,
these words are often thrown around, but you might find that
explanation to be a little vague. And rightfully so.

Here's what's really going on: when you rebase a branch onto another
commit, you take that first commit you made when you first branched
off and point it to a different commit. Doing so actually creates a
new commit with a distinct SHA1 hash (what a commit points to is an
essential part of the 'content' of a commit), and points HEAD to
it. Your original commit is still there, it's just not visible in your
log any more because it's not reachable from HEAD.

The next commit in your project branch is now pointing at this 'ghost'
commit. It needs to be updated to point to its new parent. The process
begins again. A new commit is created, HEAD is moved, and on and
on. As the rebase replays all your changes, it effectively changes
every commit hash in the branch. Your local branch and origin now have
two different copies of the same changes but none of the hashes is the
same. This is why git gives you the somewhat confusing indication to
pull your changes down before trying to push. What you need to do
instead is tell the remote to forget everything and just accept your
local branch in place of whatever it has. And that looks like this:

```sh
$ git push -f origin <branch>
```

### Some useful things to know

#### Reflog

For the longest time I held the reflog at arm's length. I knew it
existed and that it could be of help if you were in serious
trouble. Maybe there was some security in thinking that if I managed
never to use it then I could never have done anything _that_ bad.

But I was wrong. The reflog is actually exciting, powerful and pretty
straightforward.

```sh
$ git reflog
$ git reflog show <branch>
```

This will show you something that looks like this:

```
e58096a HEAD@{0}: commit: Really committed now.
5a4acd2 HEAD@{1}: commit: Commitment issues.
6f10f0e HEAD@{2}: commit: Committing some more.
146778b HEAD@{3}: commit: The awkward second commit.
8838e8d HEAD@{4}: commit: Initial commit.
```

It's possible that some of the commits the reflog will show you will
no longer be reachable on the graph (such as after a rebase). Want to
undo a rebase? Just point HEAD to where it was before you started by
using `reset` (more below).

#### Ranges

Ranges, which is to say the `..` and `...` syntax, can be pretty
confusing because they can mean different things in different
contexts. It's important to know how to use them, though.

In the context of logs:

```sh
# git log
# commits that b has that a doesn't have
$ git log <commit a>..<commit b>
```

```sh
# commits in a and b but not both
$ git log <commit a>...<commit b>
```

```sh
# the last n commits
$ git log -<n>
```

In the context of diffs:

```sh
# git diff
# changes between commit a and commit b
$ git diff <commit a> <commit b>
```

```sh
# same
$ git diff <commit a>..<commit b>
```

```sh
# changes that occurred on a's branch since it branched off of b's
$ git diff <commit a>...<commit b>
```

in the context of checking out:

```sh
# git checkout
# checkout the merge base of a and b
$ git checkout <commit a>...<commit b>
```

#### Commit Parents

Sometimes it can be easier to refer to commits not by their SHA1 hash
but by their relationship with another commit. This is especially so
when dealing with recent history and your point of reference is
HEAD. There are a number of different ways of saying the same thing,
and you can combine them too:

```sh
# the current commit
$ HEAD
$ HEAD~0
```

```sh
# the 1st parent of the current commit
$ HEAD~
$ HEAD~1
```

```sh
# the 1st parent of the 1st parent of the current commit
$ HEAD~~
$ HEAD~2
$ HEAD~1~1
```

```sh
# the 2nd parent of the current commit
$ HEAD^2
```

```sh
# uh...
$ HEAD~2^2~5^2
```

#### Add

You already know how to do that. But have you tried adding in hunks?
It looks like this:

```sh
# stage changes in hunks
$ git add -p
```

This allows you to add interactively. Git will try to present you with
smaller 'hunks' of your code to stage one by one. If it's not granular
enough for you, you can just tell git to get more granular by
splitting it. Here's what it looks like:

```
Stage this hunk [y,n,q,a,d,/,j,J,g,s,e,?]?
```

The most useful options to remember are `y` for yes, `n` for no, and
`s` for split.

#### Bisect

This does a divide-and-conquer approach to locating a commit in your
history that introduced some change (typically a regression). It
requires only that can identify some point in your history that you
know was good, and another point that is bad. Working with bisect will
typically look like this:

```sh
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
```

You then repeat steps 4-5 until you're down to one commit.

You can even automate the process:

```sh
# automate it
$ git bisect run rspec path/to/broken_spec.rb
```

Great stuff!

#### Blame

My FAVORITE tool. Mwahaha! In all seriousness though (ahem), this can
be useful in situations where you have some code you really don't
understand despite your best efforts, and you need to have a chat with
its author. Alternatively, you may want to credit someone for a
revision that was really good. It looks like this:

```sh
$ git blame path/to/file
```

#### Revert

Creates a 'mirror image' of another commit that backs out the changes
it introduced:

```sh
# create a new commit reversing the changes
$ git revert <commit>
```

You can even revert a merge commit by passing the `-m` flag and the
parent that you want to keep. Typically this will just be `1`,
indicating `master` in situations where you merged a topic branch into
it. The topic branch would be `2`:

```sh
# revert a merge
$ git revert -m 1 <merge commit>
```

#### Reset

Something you may have used in desperation. Like `rebase`, `reset` is
a powerful tool and it's worth knowing what a few of the options
do. Something all resets have in common is that they move HEAD to a
new, specified commit. Unless you're resetting to a point way back in
history, it's usually easier to provide a commit relative to HEAD.
Here are a few options you want in your tool-belt:

```sh
# leave changes not in target in staging area
$ git reset --soft HEAD~
```

```sh
# leave changes not in target in working tree (default)
$ git reset --mixed HEAD~
```

```sh
# destroy all changes not included in target
$ git reset --hard HEAD~
```

```sh
# reset to previous point in the reflog
$ git reset --hard <branch>@{<reflog entry>}
```

```sh
# reset to where you were last week (!!!)
$ git reset --hard <branch>@{one.week.ago}
```

### Conclusion

That's more or less everything I know about being a git. There are
some great resources, included below, that include more advanced
topics if you're interested in learning more. Being an intermediate
git only really requires some curiosity and practice using the tools
and techniques above. Once you get them, you'll want to use most of
them every day, and you'll have internalized everything. And being an
intermediate git won't merely bring you up to scratch - it will
actually set you apart from the rest (most of the time).
