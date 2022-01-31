# bfg-demo

## With git comes great power and great responsibility

Most people reading this will be familiar with git, the current gold standard for version control. When I first started programming git was one of the hardest things for me to learn and I ran into sticky situations more than I care to remember. 

Although I felt clueless at the time, Git is one of those things that few types of career preparation are adequately covering, as many Software Engineering or Computer Science degrees don't cover version control, period. In my experience, that means a looot of people have a painful ramp-up phase.

On the upside, Git is hugely powerful. The downside is also that it is hugely powerful.

## Gitastrophes 

One of the worst things that can happen to you using Git is that sensitive data like passwords in an environment file, or secrets make it up into a hosted repository on Github, Gitlab, or Bitbucket. 

There are methods of undoing commits, usually through a process called reverting. You can blow away branches, you can squash history, but it can be challenging to fully clear the history of something sensitive that makes it up into a repository, even if you take nuclear steps to get rid of the file. 

What we are going to cover next is one way of doing just that when normal corrective action isn't enough to hide the history of a sensitive file or sensitive data, **we are also going to cover limitations of this approach**.

**One note: this article is more suited for an intermediate audience. If you don't know git well yet, I don't recommend taking nuclear actions like this. I'm also not going to cover the basic terminology.** 

## Setup 

I'm going to start us off by making a new repository, bfg_demo. 

![Creating repository BFG demo](https://dev-to-uploads.s3.amazonaws.com/i/y1n5bp1y45fp5xucuoks.png)

I'm going to clone in, then cd into our new repo, and make some new files for us to work with.

```sh
➜  Desktop git clone https://github.com/jtkaufman737/bfg_demo.git
Cloning into 'bfg_demo'...
➜  Desktop cd bfg_demo
➜  bfg_demo (main) ✗ touch .env .gitignore file.py file.js file.go file.sql
➜  bfg_demo (main) ✗ ls
README.md file.go  file.js  file.py  file.sql
```

Now I'm going to add some "sensitive" data to our .env file, which as you'll know is usually a gitignored file that doesn't get committed to repositories, used to keep sensitive data like a secret. After adding some text, our .env looks like this: 

```sh
➜ bfg_demo (main) ✔ touch .env .gitignore file.py file.js file.go file.sql
➜ bfg_demo (main) ✗ ls
file.go  file.js  file.py  file.sql
➜ bfg_demo (main) ✗ echo "SECRET=b8ee43a51b6ba41eb873dc83e11dbbcbbb3a2131" >> .env
➜ bfg_demo (main) ✗ cat .env
SECRET=b8ee43a51b6ba41eb873dc83e11dbbcbbb3a2131
``` 

Now say we are a well-meaning person and go to gitignore this env file, but have a typo in .gitignore. Oops! 

```sh
➜ bfg_demo (main) ✗ echo ".emv" >> .gitignore
➜ bfg_demo (main) ✗ cat .gitignore
.emv
``` 

That's enough premise for our pretend-disaster. Let's check out a feature branch, commit our work, and push it up for a pull request. 

```sh
➜  bfg_demo (main) ✗ git checkout -b "disaster"
Switched to a new branch 'disaster'
➜  bfg_demo (disaster) ✗ git status
On branch disaster

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	.env
	.gitignore
	file.go
	file.js
	file.py
	file.sql

nothing added to commit but untracked files present (use "git add" to track)
➜  bfg_demo (disaster) ✗ git add .
➜  bfg_demo (disaster) ✗ git commit -m "First commit. What could go wrong?"
[disaster (root-commit) dffd8b0] First commit. What could go wrong?
 6 files changed, 4 insertions(+)
 create mode 100644 .env
 create mode 100644 .gitignore
 create mode 100644 file.go
 create mode 100644 file.js
 create mode 100644 file.py
 create mode 100644 file.sql
➜  bfg_demo (disaster) ✔ git push origin disaster 
Counting objects: 5, done.
Delta compression using up to 12 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (5/5), 458 bytes | 458.00 KiB/s, done.
Total 5 (delta 0), reused 0 (delta 0)
To https://github.com/jtkaufman737/bfg_demo.git
 * [new branch]      disaster -> disaster
```

I'm going to create and accept a merge request for this new content, including our accident-super-secret content in .env. 


![Pull request from disaster branch](https://dev-to-uploads.s3.amazonaws.com/i/pu14qis1924hvv795d47.png)

With that accepted, we have our premise. Let's move on to BFG. 

## BFG 

At this point you may be thinking, there's still plenty of ways to fix this situation without going to the nuclear option, what about reverting, why not just axe the repo and start over to get rid of problem history? 

Sure, those things _may_ be on the table. But BFG may be better than those options if this were an active repo with many contributors, where the problem commit could already be buried under other commits, and reverting or blowing away content would disrupt critical git history or otherwise not be realistic. 

So what is BFG? 

[BFG](https://rtyley.github.io/bfg-repo-cleaner/) is one of three tools that you will see time and again if you have ever done something **very bad** in git and are scrambling for how to fix it.

There are two other tools that can deal with axing sensitive info, [git filter-branch](https://git-scm.com/docs/git-filter-branch) and [git filter-repo](https://github.com/newren/git-filter-repo). [Git scm docs](https://git-scm.com/) reference git filter-branch, and when I was in the situation where I needed these tools I found myself very confused by mixed references between Gitlab and Github docs, saying that git filter-branch is no longer recommended, but having scarce or unclear instructions on how to use git filter-repo.

I'm a person who likes getting things done, so I often pick my tools on whatever has better documentation that can get me from point A to point B faster. In my case, the conflicting/sparse documentation and cross-talk in the instructions on other tools led me to use BFG. Your mileage may vary, but it was the easiest to use for me.

## Getting started 

Even with this being the _more_ approachable option, there are a few steps. I'm running this on mac so I started with a `brew install bfg`. 

(Ahhh, lovely - BFG has a dependency on Java Development Kit! At this point my Mac must be laughing at me whenever I uninstall JDK because I keep having to reinstall it for various reasons -_-)

Next, following the BFG docs I'm going to do a **bare clone**.

```sh
➜  Desktop git clone --mirror https://github.com/jtkaufman737/bfg_demo.git
Cloning into bare repository 'bfg_demo.git'...
remote: Enumerating objects: 11, done.
remote: Counting objects: 100% (11/11), done.
remote: Compressing objects: 100% (6/6), done.
remote: Total 11 (delta 1), reused 7 (delta 1), pack-reused 0
Unpacking objects: 100% (11/11), done.
```  

Before we start rewriting history let's also confirm our problem child is visible. 

![Visible env secret](https://dev-to-uploads.s3.amazonaws.com/i/dn3he8vr1m55qrh007qi.png)

## Rewriting history 

Ok, with everything set up, lets dive into the BFG docs. 

![BFG documentation on removing passwords or files](https://dev-to-uploads.s3.amazonaws.com/i/0r2baquqb20djh4fnm9k.png)

For a situation like ours, either the first or third example is in the ballpark of what might help. Because a .env file should really never make it up into a hosted service like Github or Gitlab, I'm going to go with trying to strike the .env file from history altogether. 

The formula for deleting a file using BFG is as follows: 

```sh
bfg --delete-files [file name] [<bare cloned repo>.git]
``` 

In my case that is:

```sh
bfg --delete-files .env bfg_demo.git
``` 

Ok, lets put the pedal to the medal. Running the command, here's what I see: 

```sh
➜  Desktop bfg --delete-files .env bfg_demo.git

Using repo : /Users/jkaufman/bfg_demo.git

Found 5 objects to protect
Found 4 commit-pointing refs : HEAD, refs/heads/disaster, refs/heads/main, refs/pull/1/head

Protected commits
-----------------

These are your protected commits, and so their contents will NOT be altered:

 * commit 34522682 (protected by 'HEAD') - contains 1 dirty file : 
	- .env (157 B)

WARNING: The dirty content above may be removed from other commits, but as
the *protected* commits still use it, it will STILL exist in your repository.

Details of protected dirty content have been recorded here :

/Users/jkaufman/bfg_demo.git.bfg-report/2020-10-16/10-05-46/protected-dirt/

If you *really* want this content gone, make a manual commit that removes it,
and then run the BFG on a fresh copy of your repo.
       

Cleaning
--------

Found 3 commits
Cleaning commits:       100% (3/3)
Cleaning commits completed in 24 ms.

BFG aborting: No refs to update - no dirty commits found??
```

Shoot. I'm guessing you can read between the lines of what's happening here but the TL:DR is:
- BFG found a "dirty" commit, referencing our .env file we are trying to remove
- BFG identifies the **ref**, or internal reference to that file history as being protected, because it is now accepted via pull request into our main branch
- For that reason, BFG skips over rewriting our history

As you probably also noticed, it gives us an alternate way to do this: a new commit deleting the problem file, then re-running the command. In a new terminal tab I'm going back to my non-bare repo, pulling down from main, and deleting .env. 

```sh
➜  bfg_demo (main) ✔ git pull origin main
remote: Enumerating objects: 1, done.
remote: Counting objects: 100% (1/1), done.
remote: Total 1 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (1/1), done.
From https://github.com/jtkaufman737/bfg_demo
 * branch            main       -> FETCH_HEAD
   07692d9..3452268  main       -> origin/main
Updating 07692d9..3452268
Fast-forward
 .env       | 3 +++
 .gitignore | 1 +
 file.go    | 0
 file.js    | 0
 file.py    | 0
 file.sql   | 0
 6 files changed, 4 insertions(+)
 create mode 100644 .env
 create mode 100644 .gitignore
 create mode 100644 file.go
 create mode 100644 file.js
 create mode 100644 file.py
 create mode 100644 file.sql
➜  bfg_demo (main) ✔ rm .env
➜  bfg_demo (main) ✗ ls -a
.          ..         .git       .gitignore README.md  file.go    file.js    file.py    file.sql
➜  bfg_demo (main) ✗ git status
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	deleted:    .env

no changes added to commit (use "git add" and/or "git commit -a")
➜  bfg_demo (main) ✗ git add .env
➜  bfg_demo (main) ✗ git commit -m "Remove env"
[main e88aea6] Remove env
 1 file changed, 3 deletions(-)
 delete mode 100644 .env
➜  bfg_demo (main) ✔ git push origin main
Counting objects: 2, done.
Delta compression using up to 12 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (2/2), 229 bytes | 229.00 KiB/s, done.
Total 2 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To https://github.com/jtkaufman737/bfg_demo.git
   3452268..e88aea6  main -> main
```

Now lets try our mirror clone step once again.

```sh
➜  Desktop rm -rf bfg_demo.git 
➜  Desktop git clone --mirror https://github.com/jtkaufman737/bfg_demo.git
Cloning into bare repository 'bfg_demo.git'...
remote: Enumerating objects: 11, done.
remote: Counting objects: 100% (11/11), done.
remote: Compressing objects: 100% (6/6), done.
remote: Total 11 (delta 1), reused 7 (delta 1), pack-reused 0
Unpacking objects: 100% (11/11), done.
``` 

Ok, moment of truth. Let's run our BFG command again. 

```
➜  Desktop bfg --delete-files .env bfg_demo.git

Using repo : /Users/jkaufman/Desktop/bfg_demo.git

Found 4 objects to protect
Found 4 commit-pointing refs : HEAD, refs/heads/disaster, refs/heads/main, refs/pull/1/head

Protected commits
-----------------

These are your protected commits, and so their contents will NOT be altered:

 * commit e88aea6c (protected by 'HEAD')

Cleaning
--------

Found 4 commits
Cleaning commits:       100% (4/4)
Cleaning commits completed in 39 ms.

Updating 3 Refs
---------------

	Ref                   Before     After   
	-----------------------------------------
	refs/heads/disaster | 69c88c7f | 32084753
	refs/heads/main     | e88aea6c | b8a8c5df
	refs/pull/1/head    | 69c88c7f | 32084753

Updating references:    100% (3/3)
...Ref update completed in 16 ms.

Commit Tree-Dirt History
------------------------

	Earliest      Latest
	|                  |
	  .    D    D    m  

	D = dirty commits (file tree fixed)
	m = modified commits (commit message or parents changed)
	. = clean commits (no changes to file tree)

	                        Before     After   
	-------------------------------------------
	First modified commit | 69c88c7f | 32084753
	Last dirty commit     | 34522682 | ac89d326

Deleted files
-------------

	Filename   Git id          
	---------------------------
	.env     | a23857e3 (157 B)


In total, 4 object ids were changed. Full details are logged here:

	/Users/jkaufman/Desktop/bfg_demo.git.bfg-report/2020-10-16/10-20-44

BFG run is complete! When ready, run: git reflog expire --expire=now --all && git gc --prune=now --aggressive
```

BFG explains the next steps as follows: 

![Git prune step](https://dev-to-uploads.s3.amazonaws.com/i/hyh6yp8ukblhrt4i5cxy.png)

We are going to make use of two git commands here, both used in repo cleanup: Git gc, and git prune, which you can read more about [here](https://www.atlassian.com/git/tutorials/git-gc) and [here](https://www.atlassian.com/git/tutorials/git-prune). 

```sh
➜  bfg_demo.git (main) ✔ git reflog expire --expire=now --all && git gc --prune=now --aggressive
Counting objects: 9, done.
Delta compression using up to 12 threads.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (9/9), done.
Total 9 (delta 1), reused 0 (delta 0)
```

## A caveat

One interesting thing happens as we congratulate ourselves on getting rid of the messy history and go to push our new history up. Let's see if you spot it. 

```sh
➜  bfg_demo.git (main) ✔ git push 
Counting objects: 9, done.
Delta compression using up to 12 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (9/9), 1.25 KiB | 1.25 MiB/s, done.
Total 9 (delta 1), reused 9 (delta 1)
remote: Resolving deltas: 100% (1/1), done.
To https://github.com/jtkaufman737/bfg_demo.git
 + 69c88c7...3208475 disaster -> disaster (forced update)
 + e88aea6...b8a8c5d main -> main (forced update)
 ! [remote rejected] refs/pull/1/head -> refs/pull/1/head (deny updating a hidden ref)
error: failed to push some refs to 'https://github.com/jtkaufman737/bfg_demo.git'
```

I'll spoil the suspense by telling you that this line: ` ! [remote rejected] refs/pull/1/head -> refs/pull/1/head (deny updating a hidden ref)
` seems to indicate that we couldn't remove the information from our pull request...and even playing fast and loose, using the `--force` flag on git push, no go: 

```sh
➜  bfg_demo.git (main) ✔ git push --force
Total 0 (delta 0), reused 0 (delta 0)
To https://github.com/jtkaufman737/bfg_demo.git
 ! [remote rejected] refs/pull/1/head -> refs/pull/1/head (deny updating a hidden ref)
error: failed to push some refs to 'https://github.com/jtkaufman737/bfg_demo.git'
```

So let's take a look at what this means, going back to our repo. 

![List of commits](https://dev-to-uploads.s3.amazonaws.com/i/uy4wbit1thyaa8mrmna4.png)

First I'm going to check out my "First commit what could go wrong" commit from the feature branch where we created .env. 

![disaster feature branch](https://dev-to-uploads.s3.amazonaws.com/i/rv9m5p43sgevwwk1vx76.png)

Amazing! No .env file. That's pretty awesome. Now lets take a look at our most recent commit in main, where we _deleted_ the .env file, to check that none of that secret information shows up in the diff. 

![Main repo showing a sanitized history with no env file information](https://dev-to-uploads.s3.amazonaws.com/i/kldw90bp0tc8ihx5h1ey.png)

Not gonna lie, we are already in better shape than we were previously. Our commits are clean. But there is one lingering issue, the protected ref to the _pull request_, which you'll remember BFG called out and said it couldn't delete. 

## The bitter end

I'm not the only one who has run into this, [as you'll see in this question to the creator of BFG](https://github.com/rtyley/bfg-repo-cleaner/issues/36). As they explain, it is basically outside of their power to change that last problem ref in our situation. And sure enough if we go back to the pull requests > closed pane and look at our PR, we get this in the diff: 

![Oh no! Secrets are still visible in the diff of our first PR](https://dev-to-uploads.s3.amazonaws.com/i/tocumcqzqxw0caxkvp33.png)

Here's where we hit a bit of a wall - if you look at this [stack overflow q](https://stackoverflow.com/questions/18318097/delete-a-closed-pull-request-from-github), it seems like we may have no hope but reaching out to Github customer service. (Who, by the way, are GREAT - I have had them get back to me within ~hour on a Sunday over totally trivial things.) 

So although it isn't the satisfying conclusion I'd hoped for, with an email to Github we should be able to clear the last trace of this embarrassing mistake while preserving our repositories history and a new clean record of commits. 

**While BFG functionality stops here, you CAN actually go farther using git filter-repo. Check out how to do a COMPLETE wipe in Part II, using git filter-repo*
