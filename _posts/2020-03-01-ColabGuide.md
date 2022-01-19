---
toc: true
description: A guite how to work with Fastpages, GitHub, Jupyter Notebook and Colab
categories: [Fastpages,  GitHub, Jupyter Notebook , VS Code, Colab]
image: images/vsCodeSreenShot.png
comments: true
---
# Working with fastpages, GitHub and Colab


--- 
The idea is to work with Jupyter Notebook in Colab, the notebook is part of a Fastpages blog and should be edited either in Colab or VS Code

- write a Blog in Fastpages 
- format of Blog is Jupyter Notebook
- run the Notebook in Colab
- also edit in VS Code is necessary
    - use Git to synchronize GitHub repo with VS Code local copy

## Write a blog with Fastpages

A detailed instruction on how to write Fastpage blogs is given at

- [for juypter notebooks ](https://github.com/fastai/fastpages#writing-blog-posts-with-jupyter)
- [for markdown](https://github.com/fastai/fastpages#writing-blog-posts-with-jupyter)

## Working with notebook in Colab

Once the blog is opened there are buttons

- View on GitHub
    The Jupyter Notbeook file is opened in the GitHub environment
- Open in Colab
    The Jupyter Notbeook file is opened in the Colab environment [^colabNote]

[^colabNote]: Anybody can open a copy of any github-hosted notebook within Colab just add for markup  
    `[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googlecolab/colabtools/blob/master/notebooks/colab-github-demo.ipynb)`  
    or for html  
    `<a href="https://colab.research.google.com/github/googlecolab/colabtools/blob/master/notebooks/colab-github-demo.ipynb">
    <img src="https://colab.research.google.com/assets/colab-badge.   svg" alt="Open In Colab"/>
    </a>`

In the Colab environment the file can be run with **GPU** support. An intro to **Colab** is given [here](https://colab.research.google.com/notebooks/welcome.ipynb) 

To save the changes chose under **"File"** the option **"Save a copy in GitHub"** as shown below

---

![]({{ site.baseurl }}/images/colabSaveGithubCopy.png "Save copy to GitHub from within Colab")

---

Colab will open a dialog as shown below
It might be that it asks for your GitHub credentials before opening the dialog 

--- 

![]({{ site.baseurl }}/images/colabAskPermissionGitHub.png "Colab asks for permission to save to GitHub")

---

## Working with VS Code

To work with the Fastpages blog clone the repository as described [here](https://code.visualstudio.com/docs/editor/versioncontrol#_cloning-a-repository) 

It is then easy to synchronize with GitHub, note, don't forget to pull the repo once you saved a notebook version from Colab to GitHub. 

### Avoid conflicting versions

If you do the following

- Change file in Colab
- Save to GitHub
- Change same file in VS
- Try to push

You will have *conflicts in the file** and need command line magic to solve the issue. I ended up setting the whole blog up from scratch.

Better: use **Git: Snyc** which does a **Git: Pull** first and then a **Git: Push**