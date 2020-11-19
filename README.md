# nnnn: a fast terminal filemanager with Miller columns in text mode

nnnn is a [fork of nnn](https://github.com/jarun/nnn)

## Name

To avoid confusion with nnn, on which nnnn is based, the name was changed.

nnnn is a short recursive acronym in the free software tradition.

"nnnn" stands for: "nnnn (is) notnnnn", while "nnn" apparently meant "nnn (is)
not noice".

However, to be accessible with fewer keypresses, the binaries will use a
shorter name: simply the letter n, with a symlink to n4 and nnnn made at
install time.

The name may evolve. Other candidates are nfm (for n file manager) or just n.

## Goal

The goal is to create a perfect filemanager for msys2 on windows, including:

 - distributing binaries or packages to ease installation with pacman

 - packaging at least one font that can be used by mintty to immediately access
   icons

 - any other extra features that make sense on Windows 10 computer running
   msys2, such as:

   1. replacing the slow fork of a shell (such as when pressing !) by a
     persistent shell, which can be immediately displayed by a shortcut;
     however a quake-like terminal is another interesting idea.

   2. adding dynamic Miller columns to take advantage of the large screen

   3. not requiring a recompilation only to change shortcuts

## Dynamic Miller columns

This fork was driven by the lack of good text-mode file manager featuring
Miller columns: ranger and lf each have serious drawbacks.

Dynamic Miller columns will be the #1 difference between nnnn and nnn.

### Background

In nnn issue #332, the reason given for not adding Miller columns was the
redundant disk reads:

> The fundamental issue with Miller View is too many redundant disk reads due
> to reading of subdir contents on opening or minor navigation in parent. On a
> remote mount this subdir content load will also slow you down.

_Originally posted by @jarun in https://github.com/jarun/nnn/issues/332#issuecomment-534528993_

This is a very valid reason. Having 3 columns used at all times is wasteful.

### Dynamic Miller Columns

However, as proposed in https://github.com/jarun/nnn/issues/794 that led to
this fork, a Miller-like mode can be done just with the existing information,
without extra disk reads or CPU cycles waste.

This needs one simple change: not having the numbers of columns fixed to 3 at
all times (which starts by wasting time at startup on acquiring information on
2 levels above) but dynamic (which further avoids having to refresh minor view
when navigating in parent) starting with 1 column, and returning to 1 column
when going back to the initial point.

To use less than 3 columns would simply mean to erase stale information and not
show anything until the navigation is confirmed by entering into a folder, like
by pressing on the left or right key - which would keep the performance
constant, while enriching the navigation with context about the content of the
folder previously visited in the filesystem hierarchy.

This can be seen in the 2 extreme situations (going into subfolders vs going
into parent folders), that eventualy converge back to the initial 1 column mode
when changing direction.

### For the "going into subfolders" situation:

 - nnn would start exactly as it does now, with the exact same look and feel

 - but upon opening a child folder, the screen is not redrawn the same: just
   keep the information of the parent folder on the left hand side (where it
   already is!), while supplementing it on a middle column now displayed (to
   the right of the parent folder) with the information of the child folder,
   which now becomes navigable: 2 columns are now displayed, the middle column
   is navigable

 - if another child folder is opened, the same applies, except this time it
   gets displayed in the right column (.), with the immediate parent in the
   middle (..), and the parent own parent in the left column (../..) : 3
   columns are now displayed, the right column is navigable

 - when going further in, the 2 rightmost columns are moved to the left: 3
   columns are still displayed, the right column is navigable

 - when leaving the child folder, the child content is erased (the right column
   is cleared),  its parent becomes navigable again, but no other folder is
   read: 2 columns are now displayed, the center column is navigable

 - if going back again, likewise, the center column is erased, its parent
   become navigable: only 1 column is now shown

### For "going into parent folders" situations:

 - likewise, start as is

 - when pressing on the left key, redraw the current folder content in a middle
   column, while drawing the parent content folder now in the left column: 2
   columns are now displayed, the left column is navigable

 - when pressing again on the left, same thing: 3 columns are now displayed,
   the left column is navigable
        
 - when starting navigation on the left column: the stale content of the middle
   and right column is erased. Only one column is shown and navigable. This
   allows to goes back to the "going into subfolders" situation, without visual
   distraction.

It could be argued that knowing which column is navigable can be hard, as when
showing >1 column it is not always the same column. However, the column being
navigable can be identified by the presence file selector, which also moves
when pressing on up/down, while other non navigable columns would have no such
visual identifier.

If keeping multiple lists of the files available in the displayed folders when
having >1 column is a concern for the memory used, the previous lists can be
discarded, keeping instead only the information displayed on the screen:  this
means that when coming back to a previously visited folder, instead of having
the information cached and immediately available, it will simply have to be
read from disk - which is preferable anyway, as new file may have been created,
and which would be identical performance-wise to the current situation.

### Implementation

Work in progres...

