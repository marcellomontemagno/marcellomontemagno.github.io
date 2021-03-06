---
layout: post
title: A free, simple, static, host-as-you-want blog in 10 minutes using jekyll
metaDescription: A short guide on how to create a static blog with jekyll
category: Misc
---

### What is jekyll?

Jekyll is a static site generator. It takes raw text files (e.g: Markdown a plain text formatting syntax), and spits out a complete, ready-to-publish static website. You can then serve the generated site using your favorite web server, or, because of Jekyll is also the engine behind GitHub Pages, you can host your website from GitHub’s servers, for free, through a simple commit.

### Features?

- Free, MIT license
- Is static
- Blog-aware
- Customizable

### Looks cool, how to create and run my blog or static site?

- [Install Ruby](http://example.com/ "Install Ruby") (remember to check 'add ruby executables to you path' during the installation)
- run these commands from the directory where you want to create your site
	gem install jekyll
	jekyll new your-site-name
	cd your-site-name
	jekyll serve
- open your browser at http://localhost:4000

### How to write my first post?

- navigate into the _posts folder
- create a new file with the following naming convention YEAR-MONTH-DAY-title.md
- write your post using Markdown plain text formatting syntax
- refresh your browser at http://localhost:4000, changes should be displayed

### Is learning Markdown worth it? I only care about contents!

That's all you need to know trough an example:

	### An Header

	A simple paragraph

	Another simple paragraph

	> A blockquote.
	>
	> Another paragraph in the blockquote.

	- list element 1
	- list element 2

	*emphasized text*

	**strong emphasis text**

	a link [example link](http://example.com/ "With a Title"){:target="_blank"}

	notice that in the previous link "With a Title" and {:target="_blank"} are optionals

	an image ![alt text](/path/to/img.jpg "Title")

	a litle code block `<blink>`

	a comment block (notice the indentation tabs)

		<div>
			<p>some code</p>
		</div>

[Full Markdown documntation](https://daringfireball.net/projects/markdown/ "markdown documentation"){:target="_blank"}

### What if I want to customize the style or pages that are not posts?

I've choosen the [beautiful-jekyll](http://deanattali.com/beautiful-jekyll/ "beautiful-jekyll"){:target="_blank"} theme

You can search for an existing theme online (e.g [jekyllthemes.io](http://jekyllthemes.io/ "Jekyll Themes"){:target="_blank"}) or create your own theme.

Anything you need is in the [Jekyll official documentation](https://jekyllrb.com/docs/home/ "Jekyll doc"){:target="_blank"}

### How to host my new blog for free through github?

Create a new repository on GitHub.com through the "new repository" button once logged, you can skip to initialize the new repository with README, license, or gitignore files, these files can be added later.

Open Terminal (for Mac and Linux users) or the command prompt (for Windows users).

Change the current working directory to your local project.

Initialize the local directory as a Git repository.

	git init

Add the files in your new local repository. This stages them for the first commit.

	git add .
	# Adds the files in the local repository and stages them for commit

Commit the blog files.

	git commit -m "First commit"
	# Commits the tracked changes and prepares them to be pushed to a remote repository

Copy remote repository URL fieldAt the top of your GitHub repository's Quick Setup page.

In Terminal, add the URL for the remote repository where your local repository will be pushed.

	git remote add origin [your remote repository URL]
	# Sets the new remote
	git remote -v
	# Verifies the new remote URL

Push the changes in your local repository to GitHub.

	git push origin master
	# Pushes the changes in your local repository up to the remote repository you specified as the origin

Access to your new repository through Github.com and then click on the setting button.

Rename your repositoy to yourGitUsername.github.io

The page should then be displayed to http://yourGitUsername.github.io in a couple of minutes.

