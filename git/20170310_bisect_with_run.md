# Git - bisect with automatic run

The other day I had made several commits to a feature branch. While I coded, I
only ran the test files associated with the code I wrote.

After around 10 commits, I found out another test was suddenly failing. I was
pretty sure it was not introduced by the last commit, but I had no idea which
of the 10 commits made the test fail.

Git bisect to the rescue!

`git bisect` is basically a feature where git does binary search in your
commits. You tell it which commit is after a bug is introduced and which commit
is before. Typically these commits will be `HEAD` and `master`.

I only knew about the manual way to run `git bisect` which I will explain first:

## Manual git bisect

Start git bisect, then tell it which commit is bad and which is good:

```
git bisect start
git bisect bad         # Current commit is "bad" because it has a broken test
git bisect good master # master is good, because no test fails in master
```

After this series of commands, git has now checked out a commit that is
approximately in the middle between the good and the bad commit.

You can now do your thing to find out if this commit is good or bad. In my case
I would run my tests.

Then if the tests fail I will mark it as bad:

```
git bisect bad
```

and if the tests pass I will mark it as good:

```
git bisect good
```

Either way, git will now take me to the next commit I should check, just like
when doing a binary search.

This is all good, but a lot of manual typing. There must be a better way.
Spoiler: There is! :)

## Automatic git bisect

After the first commands where we tell git which commits are known to be good
and bad, we can simply tell git bisect to run a certain script that will tell
if the current commit is good or bad.

The script should exit with code 0 if the commit is good and any other code if
it's bad (some exit codes have special meaning in git bisect, but I'll leave that
out here).

In my case I would simply run my tests:

```
git bisect run mix test
```

Then git bisect will do a binary search along several commits and run my
command for each one. When it's done it will tell me which commit was the first
bad one. Nice.

## What's next

I recommend reading the man page for git bisect.

```
man git bisect
```
