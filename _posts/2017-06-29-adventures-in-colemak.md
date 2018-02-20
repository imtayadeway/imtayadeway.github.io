---
layout: post
title: "Adventures in Colemak"
tags:
- ergonomics
- debian
date: 2017-07-01
---

Without much warning I recently decided to learn Colemak.

### What?

Colemak is an alternative layout for keyboards. It aims to improve on
both the traditional QWERTY and the only slightly better-known Dvorak
by placing the commonest keys on the home row, along with certain
other considerations, to improve ergonomics and comfort while typing.

### Why?

This came as a bit of a surprise to me as I have always felt somewhat
opposed to learning a new keyboard layout. This may have stemmed from
my own frustration in the past
in [doubling](https://en.wikipedia.org/wiki/Woodwind_Doubler) on
Clarinet and Saxophone. While the two are keyed similarly, they
correspond to different "notes" as they are written down. Though it is
very common for people to do this, I really don't enjoy the feeling of
disorientation at all.

The drawbacks I identified as:

- the initial effort of learning
- having to "double" when confronted with a QWERTY keyboard
- really, having to collaborate with anyone on anything ever again

The supposed benefits of faster typing speed and prevention of RSI I
never saw as a net gain. Which is not to say that I don't care about
those things (I take injury prevention very seriously,
having
[blogged about this before](http://timjwade.com/2014/08/25/how-to-type.html)). It's
just such an inexact science that I would welcome both of those
benefits if they came, but couldn't reasonably expect them as
guaranteed.

But I think there was one other factor that has completely swung this
for me that has probably not been present at any other time that I've
been thinking about this. It is that I am incredibly bored. So bored
that I don't want to learn anything exciting like a new programming
language, or even a new natural language, or how to ride a unicycle or
spin poi. I've been craving the dull repetition that I've felt as a
musician, a quiet confidence that if I do this dance with my hands
slowly and correctly enough times, I'll program myself to perform a
new trick. I've been actually longing for the brain ache you get when
you're trying to do something different and your muscle memory won't
quit.

### How?

There are many of these online, but I
found [The Typing Cat](http://thetypingcat.com/) particularly good in
getting started out. Not wanting to take the plunge straight away,
this let me emulate the new layout while I went through the exercises,
preserving QWERTY for everything else. For the first couple of weeks
I'd do QWERTY during the day and practice 1-2 hours of Colemak in the
evening, until I got up to an acceptable typing speed (for me, 30 wpm,
while still very slow, would not interfere too much).

Once I was ready to take the leap, I was confronted by a great number
of ways to do this, ranging from reconfiguring the keyboard at the
system level (useless, since X ignores it), configuring X from the
command line (annoying, because those changes aren't preserved when I
make any customizations in the Gnome Tweak Tool), to discovering I
could do most of this by adjusting settings in the UI. I'll describe
only what I eventually settled on in detail, in case you are trying to
do this yourself and are running a similar setup to me (Debian
9/Stretch, Gnome 3, US keyboard).

To set up Colemak, simply open Settings, go to Region & Language, hit
the `+` under Input Sources, click `English` then `English (Colemak)`
and you're done. You should now see a new thing on the top right that
you can click on and select the input source you wish to use. You can
also rotate input sources by hitting Super (aka Windows key) and
Space.

Unfortunately I wasn't done there because I had a few issues with some
of the design choices in the only variant of Colemak offered. Namely,
I didn't want Colemak to reassign my Caps Lock key to Backspace (as I
was already reassigning it to Escape), and I wanted to use my right
Alt key as Meta, something I use all the time in Emacs and pretty much
everything that supports the basic Emacs keybindings (see: everything
worth using). While there may have been a way to customize this from
the command line, I never found out what that was, and besides I
wanted to find a solution that jelled as much as possible with the
general solution I've outlined above. It was with this spirit that I
decided to add my own, customized keyboard layout. If you're having
similar grumbles, read on.

First, a word of caution. You're going to have to edit some
configuration files that live in `/usr/share`. If that makes you
queasy, I understand. I don't especially love this solution, but I
think it is the best of all solutions known to me. Either way, as a
precautionary measure, I'd go ahead and backup the files we're going
to touch:

```sh
sudo cp /usr/share/X11/xkb/symbols/us{,.backup}
sudo cp /usr/share/X11/xkb/rules/evdev.xml{,.backup}
```

Next we're going to add a keyboard layout to the
`/usr/share/X11/xkb/symbols/us` file. It'll be an edited version of
the X.Org configuration which you can
find [here](https://colemak.com/pub/unix/colemak-1.0.tar.gz). It can
probably go anywhere, but I inserted it immediately after the existing
entry for Colemak:

```
// /usr/share/X11/xkb/symbols/us

partial alphanumeric_keys
xkb_symbols "colemak-custom" {

    include "us"
    name[Group1]= "English (Colemak Custom)";

    key <TLDE> { [        grave,   asciitilde ] };
    key <AE01> { [            1,       exclam ] };
    key <AE02> { [            2,           at ] };
    key <AE03> { [            3,   numbersign ] };
    key <AE04> { [            4,       dollar ] };
    key <AE05> { [            5,      percent ] };
    key <AE06> { [            6,  asciicircum ] };
    key <AE07> { [            7,    ampersand ] };
    key <AE08> { [            8,     asterisk ] };
    key <AE09> { [            9,    parenleft ] };
    key <AE10> { [            0,   parenright ] };
    key <AE11> { [        minus,   underscore ] };
    key <AE12> { [        equal,         plus ] };

    key <AD01> { [            q,            Q ] };
    key <AD02> { [            w,            W ] };
    key <AD03> { [            f,            F ] };
    key <AD04> { [            p,            P ] };
    key <AD05> { [            g,            G ] };
    key <AD06> { [            j,            J ] };
    key <AD07> { [            l,            L ] };
    key <AD08> { [            u,            U ] };
    key <AD09> { [            y,            Y ] };
    key <AD10> { [    semicolon,        colon ] };
    key <AD11> { [  bracketleft,    braceleft ] };
    key <AD12> { [ bracketright,   braceright ] };
    key <BKSL> { [    backslash,          bar ] };

    key <AC01> { [            a,            A ] };
    key <AC02> { [            r,            R ] };
    key <AC03> { [            s,            S ] };
    key <AC04> { [            t,            T ] };
    key <AC05> { [            d,            D ] };
    key <AC06> { [            h,            H ] };
    key <AC07> { [            n,            N ] };
    key <AC08> { [            e,            E ] };
    key <AC09> { [            i,            I ] };
    key <AC10> { [            o,            O ] };
    key <AC11> { [   apostrophe,     quotedbl ] };

    key <AB01> { [            z,            Z ] };
    key <AB02> { [            x,            X ] };
    key <AB03> { [            c,            C ] };
    key <AB04> { [            v,            V ] };
    key <AB05> { [            b,            B ] };
    key <AB06> { [            k,            K ] };
    key <AB07> { [            m,            M ] };
    key <AB08> { [        comma,         less ] };
    key <AB09> { [       period,      greater ] };
    key <AB10> { [        slash,     question ] };

    key <LSGT> { [        minus,   underscore ] };
    key <SPCE> { [        space,        space ] };
};
```

Next you need to register it as a variant of the US keyboard layout:

```xml
<!-- /usr/share/X11/xkb/rules/evdev.xml -->
<xkbConfigRegistry version="1.1">
  <!-- ... -->
  <layoutList>
    <layout>
      <!-- ... -->
      <configItem>
        <name>us</name>
        <!-- ... -->
      </configItem>
      <variantList>
        <!-- Insert this stuff =-> -->
        <variant>
          <configItem>
            <name>colemak-custom</name>
            <description>English (Colemak Custom)</description>
          </configItem>
        </variant>
```

Finally, you'll need to bust the xkb cache. I read about how to do
this
[here](https://askubuntu.com/questions/482678/how-to-add-a-new-keyboard-layout-custom-keyboard-layout-definition),
but it didn't seem to work for me (most likely differences between
Ubuntu and Debian, or different versions). So to prevent giving you
the same disappointment, I'm going to tell you the best way to get
this done that is sure to work: restart your damn computer. If you can
figure out a better way, that's great.

Having done all the above, you should now be able to select your
`Colemak (Custom)` layout in the same way by going through the
settings in the UI.

Since I've made the switch, I've seen my speed steadily increasing up
to 50-60 wpm. That's still kind of slow for me, but I have every
confidence that it will continue to increase. I think doing drills has
helped with that. Since I have no need for emulation anymore, I've
found the CLI utility `gtypist` to be particularly good. I try to do
the "Lesson C16/Frequent Words" exercises for Colemak every day.
