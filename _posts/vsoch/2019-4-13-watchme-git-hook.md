---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2019-04-13 12:34:00'
layout: post
original_url: https://vsoch.github.io//2019/watchme-git-hook/
title: Git Hooks to Plot Reproducible Monitoring
---

<p>I’ve been working hard to generate metrics 
for my system (well, actually no, <a href="https://www.github.com/vsoch/watchme-system">it’s automated</a>
and I just need to push to GitHub, and arguably that could be automated too!) using my reproducible
monitoring tool <a href="https://vsoch.github.io/watchme" target="_blank">watchme</a>, and today I decided it would
be nice to have some images generated automatically on GitHub pages. I didn’t want anything fancy - just
png files rendered in a markdown file would do!</p>

<h2 id="should-we-use-github-actions">Should we use GitHub Actions?</h2>

<p>Since we are pushing to GitHub Pages, I first decided to go with using <a href="https://developer.github.com/actions/managing-workflows/workflow-configuration-options/">GitHub Actions</a>
so that I could keep my GitHub secrets within GitHub. It wasn’t as easy peezy and quick as I would have liked.
I could go into details, but it suffices to say that I couldn’t find the perfect set of GitHub Action steps
to perform exactly what I wanted - some special Python work and the push of a folder to GitHub pages. I did find several
scattered actions to push to pages (one being one of my own) but they weren’t perfectly what I wanted.
In this process of testing I also realized that I really didn’t like not being able to see what was going on until it was successful or
failed, or that the default view of looking at a <code class="highlighter-rouge">main.workflow</code> was a graphical interface. I wanted to see
the lines rolling across the screen like cake batter pouring, and I wanted the workflow to just show itself
without makeup. You’re beautiful as you are, <code class="highlighter-rouge">main.workflow</code>!</p>

<h2 id="the-cinderalla-shoe-didnt-fit">The Cinderalla Shoe Didn’t Fit</h2>

<p>And here we have an interesting thing. GitHub Actions are powerful only if enough work has been done
to put together the (almost) exact example of what you need. The shoe has to fit, otherwise you
are going barefoot to the ball. If this doesn’t exist, you really
need to be somewhat of an experienced developer to get your job done. Putting together the dependency structure
is hard, and getting the containers (action steps) running and working is even harder. Unless you write your
own script with enough print lines to debug, it’s frustrating to not be able to see what’s going on, and
sometimes to get an error that you have no good way to debug. I’m definitely okay with doing the work
needed for a task that I want to share with others, but for my own little repos, I sometimes just
want to quickly get it done. So in this case, I wound up writing a custom deploy script to pass to a container that I found to have
the python libraries needed. However, I found it way overly complicated for what I wanted to do.</p>

<h2 id="git-hooks">Git Hooks</h2>

<p>In times like these, I step back and think about how I can simplify. Sure, I could jump over to another CI service
that lets me do whatever I like. I would need to share GitHub secrets to push back to pages. Should I do this?
I decided to look for other (simpler) ways, and here we delve into Git Hooks!</p>

<blockquote>
  <p>What is a git hook?</p>
</blockquote>

<p>A git hook is an easy way to trigger scripts to run based on git events. For example, you can 
create a hook that is triggered before or after you commit, push, or do some other kind of update.
It’s akin to having a little version control system on your computer! Yes, this means you could
use hooks for linting, testing, or generating content. You can read more about git hooks <a href="https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks">here</a>.
But where are these hooks?</p>

<h3 id="1-look-under-your-nose">1. Look Under Your Nose!</h3>

<p>How do you get started with hooks? It comes down to looking under your nose, and renaming a file! 
If you look in any .git folder, you will find a subdirectory of hooks. Specifically, these are examples
to help you get started:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nv">$ </span><span class="nb">ls</span> .git/hooks/
applypatch-msg.sample  post-update.sample     pre-commit.sample          pre-push.sample    update.sample
commit-msg.sample      pre-applypatch.sample  prepare-commit-msg.sample  pre-rebase.sample

</code></pre></div></div>

<p>I don’t know if it’s just me, but I love simple tools that have functionality based on the location or
names of files. For example, instead of writing some yaml config nonsense, you just put your file in the .git/hooks folder.
I decided that I’d try creating a <code class="highlighter-rouge">pre-push</code> hook that would run my script, add images in a “docs” folder
for GitHub pages, and then finish the push. The drawback here is that the hook requires the python
dependencies to be installed on my system. But actually, I can just plop a container in there to use,
and some future user could do the same to reproduce the generation.</p>

<h3 id="2-generate-a-script">2. Generate a Script</h3>

<p>I started with a template, and note that removing the extension will activate the hook.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>cp .git/hooks/pre-push.sample .git/hooks/pre-push
</code></pre></div></div>

<p>I first did a simple test, and simply printed and listed the content the present working directory:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nv">$ </span>git push origin master 
/home/vanessa/.watchme/system
data  README.md  task-cpu  task-memory	task-network  watchme.cfg
...

</code></pre></div></div>

<p>Cool! I next decided to write the <a href="https://github.com/vsoch/watchme-system/blob/master/hooks/pre-push">full script</a> 
to pull a container and run the generation, and then provide the actual hook in my repository, so others could copy to .git/hooks
to use it.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nv">$ </span>mkdir <span class="nt">-p</span> hooks
<span class="nv">$ </span>cp .git/hooks/pre-push hooks/pre-push
<span class="nv">$ </span>git add hooks/pre-push

</code></pre></div></div>

<p>It’s too bad that GitHub doesn’t just grab this folder and include custom scripts for others to clone. I suspect
there could be some kind of security issue, but I’ve never looked into it.</p>

<blockquote>
  <p>How is it for a developer?</p>
</blockquote>

<p>It’s way easy to test because all you need to do is run scripts and containers on your local machine.
As a sidenote, the coolest part of the script (and watchme in general) is that I’m using the git repository as a temporal
database. I export each task data series like this:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="c"># Export all data</span>
<span class="k">for </span>task <span class="k">in </span>cpu memory network python sensors system users
<span class="k">do
    </span>watchme <span class="nb">export </span>system task-<span class="nv">$task</span> <span class="nt">--out</span> data/task-<span class="nv">$task</span>.json vanessa-thinkpad-t460s_vanessa.json <span class="nt">--json</span>
<span class="k">done</span>

</code></pre></div></div>

<h3 id="3-unexpected-coolness">3. Unexpected Coolness</h3>

<p>In testing this, I stumbled on a cool use case. If you commit and push (or do a similar action) within a hook
that is triggered by the same action - you create an infinite loop! Although I haven’t tried it and it could
have evil consequences, this would be a very silly way to use .git to run some action continuously. Likely
you would put some long sleeps in there so it wouldn’t be a terrible burden on your system. I’m not saying to
do this, it’s probably a terrible idea, it was just really cool to discover it by accident.</p>

<h3 id="4-the-end-result">4. The End Result</h3>

<p>What is the end result? A <a href="https://vsoch.github.io/watchme-system/">simple page with plots</a>. Most of them
aren’t even very good, but as I discuss <a href="https://github.com/vsoch/watchme/blob/master/paper/paper.md#watcher-example">here</a>, 
it still serves as a simple example of doing reproducible monitoring. For example, in this early example (only two days)
you can see my memory usage dip up and down alongside the time when I’m working on my computer:</p>

<p><br /></p>

<p><img src="https://github.com/vsoch/watchme/raw/master/paper/img/virtual-memory-used.png" alt="https://github.com/vsoch/watchme/raw/master/paper/img/virtual-memory-used.png" /></p>

<p>Git Hooks are simply awesome. I like the simplicity and control better than GitHub Actions, and the
only constraint is my host resources and needing to set up the hooks. I’ll definitely be exploring
these more in the future! Also, if you would like any help to set up a watchme watcher to collect
some kind of data, please reach out! Reproducible monitoring doesn’t seem to be a thing, and
it should be.</p>