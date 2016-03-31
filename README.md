# Contributing to Kubernetes Chinese Documentation

Thanks everyone for contributing your time on translating Kubernetes documentation into Chinese.

If you're not familiar with processes working with git, please follow the steps below:

 1. Have git installed in your machine.
 2. OPTIONAL: Clone already translated file to your local machine.

        git clone https://git.gitbook.com/linfan1/kubernetes-chinese-docs 

 3. Use the `Fork` button in the upper-right side of the screen to fork this repo to your own github account.
 4. Clone your repo to your local machine.
 5. git clone https://github.com/YOURACCOUNT/kubernetes-docs-cn.git
 6. Enter the cloned folder and set your emailaddress and username of your github account, so when you commit a change, git will know who you are.

        git config set user.email YOUREMAILADDRESS
        git config set user.name YOURGITHUBACCOUNT

 7. Set this repo as an upstream.

        git remote add upstream https://github.com/kubernetes/kubernetes-docs-cn.git

 8. Check out to branch `release-1.1-cn`.

        git checkout release-1.1-cn

 9. Snychronize your repo with upstream.

        git fetch upstream
        git merge upstream/release-1.1-cn

 10. Create a new working branch.

        git checkout -b NEWWORKINGBRANCH

 11. Do you translation work, the documents are located in `/docs` folder. If you have done step 2, find your translated file and overwrite the coresponding file. If you're working on an untranslated file, just edit it.
 12. Commit your changes. Input your commit title and message as the prompt says. Then press `Ctrl+X` and then `Shift+Y`.

        git add .
        git commit

 13. Push your commit to your own repo and create a branch coresponding to your working branch.

        git push origin NEWWORKINGBRANCH:NEWWORKINGBRANCH

 14. Now go to your repo page `https://github.com/YOURACCOUNT/kubernetes-docs-cn`, use the `New Pull Request` button to create a new PR. **DO** choose `release-1.1-cn` branch in the left and `NEWWORKINGBRANCH` in the right. 
 15. Want to send another PR? Repeat step 8 to 14.

## How to squash your commits

1. Make sure your code is up-to-date: git pull
2. Use `rebase` command to squash recent commits:

        git rebase -i release-1.1-cn

    Following content will be shown:

    	pick fb554f5 This is commit 1
    	pick 2bd1903 This is commit 2
    	pick d987ebf This is commit 3
    
    	# Rebase 9cbc329..d987ebf onto 9cbc329
    	#
    	# Commands:
    	#  p, pick = use commit
    	#  r, reword = use commit, but edit the commit message
    	#  e, edit = use commit, but stop for amending
    	#  s, squash = use commit, but meld into previous commit
    	#  f, fixup = like "squash", but discard this commit's log message
    	#  x, exec = run command (the rest of the line) using shell
    	#
    	# If you remove a line here THAT COMMIT WILL BE LOST.
    	# However, if you remove everything, the rebase will be aborted.
    	#
	
    The first three lines represent the most recent three commits, mark the commit you want to keep as *pick* or *p*, and mark the following as *squash* or *fixup*, refer to the instruction for the distinction between the marks.
    After your modification is done, the result will be like:

    	pick fb554f5 This is commit 1
    	squash 2bd1903 This is commit 2
    	squash d987ebf This is commit 3

    Press `Ctrl+X` to exit, and then `Shift+Y` to confirm.

    >**What if I want to squash commits more than three?**

    	git rebase -i HEAD~N

    N is the number of commits you want to squash.

3. After squashing done, don't forget to push it to the remote repo:

    	git push -f

## NOTES
- The steps above are just very basic practices of using git. There're more problems you may meet. So if you're new to git, learn it ASAP.
- **Start an issue before you create a PR**, so other people would know which file you're working on. Issue can be named as `Translating docs/admin/accessing-the-api.md`.
- Bound the issue and PR together using `#ISSUEID` in your PR.
- Comment under your PR use `@SOMEONE` to notify someone to review your PR.
- Send only one translated file in one single PR. So reviewers can make a quick review and merge the PR early.
- The PR I created to modify the README file can be considered as an example.
- For more information, continue reading this file.

# Contributing to the Kubernetes Documentation and Website

Welcome! We are very pleased you want to contribute to Kubernetes.

You can click the "Fork" button in the upper-right area of the screen to create a copy of our site on your GitHub account called a "fork." Make any changes you want in your fork, and when you are ready to send those changes to us, go to the index page for your fork and click "New Pull Request" to let us know about it.

If you want to see your changes staged without having to install anything locally,
change your fork of our repo to be named:

    YOUR_GITHUB_USERNAME.github.io

Then, visit: [http://YOUR_GITHUB_USERNAME.github.io](http://YOUR_GITHUB_USERNAME.github.io)

You should see a special-to-you version of the site. 

## Running the site locally

If you have files to upload, or just want to work offline, run the below commands to setup
your environment for running GitHub pages locally. Then, any edits you make will be viewable
on a lightweight webserver that runs on your local machine.

First install rvm

	\curl -sSL https://get.rvm.io | bash -s stable

Then load it into your environment

	source /Users/(USERNAME)/.rvm/scripts/rvm (or whatever is prompted by the installer)

Then install Ruby 2.2 or higher

	rvm install ruby-2.2.4
	rvm use ruby-2.2.4 --default
	
Verify that this new version is running (optional)

	which ruby
	ruby -v
	
Install the GitHub Pages package, which includes Jekyll

	gem install github-pages

Clone our site

	git clone https://github.com/kubernetes/kubernetes.github.io.git

Then, to see it run locally:

	cd kubernetes.github.io
	jekyll serve

Your copy of the site will then be viewable at: [http://0.0.0.0:4000](http://0.0.0.0:4000)
(or wherever Ruby tells you).

If you're a bit rusty with git/GitHub, you might wanna read
[this](http://readwrite.com/2013/10/02/github-for-beginners-part-2) for a refresher.

The above instructions work on Mac and Linux.
[These instructions ](https://martinbuberl.com/blog/setup-jekyll-on-windows-and-host-it-on-github-pages/)
might help for Windows users. 

## Thank you!

Kubernetes thrives on community participation and we really appreciate your
contributions to our site and our documentation!
