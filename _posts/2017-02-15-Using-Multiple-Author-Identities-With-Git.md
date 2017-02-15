How do you manage git credentials for different organizations? You may have some projects you work on at work that has a pre-push hook to ensure that your email is part of the correct domain. You might have your own personal projects, some of which might also use difference email addresses. Or, just maybe, you have a secret identity that you don't want your other repositories to know about. How does one manage this schizophrenic multiple personality disorder that we call Version Control?

If I commit with the wrong email at work, I won't be allowed to push to our internal Bitbucket server. If I commit with the wrong email on one of my personal Github repositories, I end up with this ugly situation:

![Github commit history]({{ site.url }}/assets/images/GitCommitEmails/Github_Commit_History.png "Github commit history")

The commits that don't have my image were made with my name but with an **unknown email**. So GitHub doesn't know that it was really me and does not link it to my Github identity. What I need to do is to tell git to use the email that Github expects.

## How to assume multiple identities
Git commits have an author with a name and an email. Run `git log` in a repository you've committed to recently to see some of the metadata of your commits:
```
commit bd294498cbd5c67b51096518ce62c9204068be2c
Author: Bruce Wayne <bruce.wayne@wayneenterprises.com>
Date:   Tue Feb 14 19:58:34 2017 +0200

    Plan attendance to charity events
```

Git uses the user name and email set in your global .gitconfig file, located at `~/.gitconfig` or `C:\Users\MyUser\.gitconfig`. The `[user]` block sets the author name and email address for all commits. You can set it by manually editing the file:
```
$ vim ~/.gitconfig
```

```
[user]
        name = "Bruce Wayne"
        email = "bruce.wayne@wayneenterprises.com"
```

Git also provides a command to update global settings.
```
$ git config --global user.name "Bruce Wayne"
$ git config --global user.email bruce.wayne@wayneenterprises.com
```
This modifies the global .gitconfig for you.

### Specifying identities at the repository level
So now that you've created your global identity, how about adding a different one for a particular repository? Each repository has its own config file in which you can override the global configurations - it's located in the hidden .git folder. Take a look at `.git/config` inside your repository.

To create your repository-specific secret identity, edit the file and add a user config to override the global one:
```
$ vim ~/.gitconfig
```

Or, you can run the `git config` command and specify the change as being local to the repository.

```
$ git config --local user.name "Batman"
$ git config --local user.email "batman@justiceleague.com"
```

This is the configuration that you should end up with in `.git/config`:

```
[user]
        name = "Bruce Wayne"
        email = "bruce.wayne@wayneenterprises.com"
```

Now, you can use Git under your secret identity!

```
Author: Batman <batman@justiceleague.com>
Date:   Wed Feb 15 04:53:57 2017 +0200

    Create initial blueprints for Batcave
```

## That's too late for me, I've already committed with the wrong author email!
Sometimes you make a mistake, like this one here:
```
Author: Bruce Wayne <bruce.wayne@wayneenterprises.com>
Date:   Thurs Feb 16 19:21:35 2017 +0200

    Fight crime in downtown Gotham
```

With a commit author like that, Batman's secret identity is surely at risk of being compromised. Luckily, you still have the opportunity to modify this commit before you push it upstream.

### Amend the commit
If the commit that you need to change is the most recent commit, you can easily change it like this:
```
$ git commit --amend --author="Author Name <authoremail@example.com>"
```

`git commit --amend` allows you to change the commit message if you choose to, and the `--author="Author Name <authoremail@example.com>"` part sets the author name and email to whatever you've specified.

You can use it to modify the previous commit to the following:
```
Author: The Dark Knight <batman@justiceleague.com>
Date:   Thurs Feb 16 19:21:35 2017 +0200

    Fight crime in downtown Gotham
```
Even Batman needs a helping hand sometimes.
### Interactive rebase
One of my favourite tricks, the interactive rebase, can be used to commits other than the most recent one. [This tutorial](http://gitready.com/advanced/2009/02/10/squashing-commits-with-rebase.html) should get you up to speed with how to do a rebase.

To amend commit authors, we run an interactive rebase, specify to edit the commits that we want to change, and then run the `git commit --amend --author` command we used previously to edit the commit's author.

```
$ git rebase -i head~4
```

We've started an interactive rebase. We'll be presented with a text editor containing the following:
```
pick fe79802 Buy new batmobile
pick 04a7c9c Dent new batmobile
pick 2a826b2 Attend Policeman's Ball
pick 1d12e8c Defuse bomb before detonation

# Rebase 56d3e50..1d12e8c onto 56d3e50 (4 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

We attended the Policeman's Ball under Batman's identity, but it should really be Bruce Wayne. On the third line, change `pick` to `edit`.
```
edit 2a826b2 Attend Policeman's Ball
```
Now we can amend the commit to set the correct author.
```
$ git commit --amend --author="Bruce Wayne <bruce.wayne@wayneenterprises.com>"
```

When you're done, run `git rebase --continue` to move to the next commit that you specified to edit, or to continue to the end of the rebasing session.

### Bulk change commits
Using the `git filter-branch` command, which is way beyond the scope of this blog post, you can do all kinds of crazy things. One of the more reasonable usages is to inspect every commit in your repository, check if it matches a particular email address, and if so, change it to a new one.

I've create a Gist on Github to do this: [author-amend](https://gist.github.com/TheCodedSelf/25e6771efa181bad734295d7fe095550)

Just save it in the root of your repository and execute it, or copy it and paste it in your terminal.

Change the values of `OLD_EMAIL`, `CORRECT_NAME`, and `CORRECT_EMAIL` to suit your needs.

## Changing author identities is not that hard.
Once you've been introduced to the right commands, there's always a way to coax git into suiting your needs. There's no need to put up with incorrect names, and definitely no need to scrap the commit and start again. 

So, when something's gone wrong: 
![](https://media.giphy.com/media/l41lSR9xZubfd2Qve/giphy.gif)

Remember, `git commit --amend` is your friend.

Thanks for taking the time to read this, and I hope you learned something that might prove helpful in the future. If you did, please take the time to share it on Facebook, Twitter, Reddit, or whatever it is that people are using these days.