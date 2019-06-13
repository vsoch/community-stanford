---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2019-04-08 12:30:00'
layout: post
original_url: https://vsoch.github.io//2019/git-tricks/
title: Git Tricks
---

<p>I’m convinced that git is the greatest software of all time. Not only does it give
you tracking of changes for your files (version control) and a collaborative platform
for working on code, it also serves any kind of creative purpose that you can think of!
For example, I’m finishing up a tool called <a href="https://www.github.com/vsoch/watchme">watchme</a> 
that uses git as a temporal database. I’m calling this “reproducible monitoring” - the 
user can configure a repository (called a watcher) to regularly run one or more tasks 
(using cron) and then save the organized results to a git repository, along with the configuration
parameters to run it in the first place. Although the repository might just (superficially) show
one result file at one timepoint, with git I can do some tricks to turn the repository
into a temporal database that can export results over time! This means saving a series
of git commits, dates, and result content. While I won’t go into the details of watchme in this post 
(See the <a href="https://vsoch.github.io/watchme">documentation</a> for details) I <em>do</em> want to 
review some of the awesome git commands that drive the watchme tool.</p>

<h2 id="commits">Commits</h2>

<p>A lot of indexing in git is based on commits. It follows that it would be super user to get an entire
list of commits, or commits after a particular index. Let’s start with a simple way to get <strong>all the commits</strong>:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>git log <span class="nt">--all</span> <span class="nt">--oneline</span> <span class="nt">--pretty</span><span class="o">=</span>tformat:<span class="s2">"%H"</span>
</code></pre></div></div>

<h3 id="earliest-and-latest-commits">Earliest and latest commits</h3>

<p>What about if we wanted to get the <strong>earliest commit</strong>? That might look like this:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>git rev-list <span class="nt">--max-parents</span><span class="o">=</span>0 HEAD
b74ae5368a7dd2d30efce0c009a2b10de2c0ddc1
</code></pre></div></div>

<p>And what about the <strong>latest commit</strong>?</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>git log <span class="nt">-n</span> 1 <span class="nt">--pretty</span><span class="o">=</span>format:<span class="s2">"%H"</span>
613a9eedaa1133fac6fd3325e5ed04e25d7bd025
</code></pre></div></div>

<h3 id="range-of-commits">Range of commits</h3>

<p>Finally, you can get a list of commits between some range.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>git log <span class="nt">--all</span> <span class="nt">--oneline</span> <span class="nt">--pretty</span><span class="o">=</span>tformat:<span class="s2">"%H"</span> fcddec
d88839c5e5eaa17ae74abab93ec97511be..
613a9eedaa1133fac6fd3325e5ed04e25d7bd025
1c041af537ad4da901725e55aea63a4cbf497621
5cc829fcb47d4f7cb2ef5217171842e3c8c3c69f
75b688717d08d9364b93a4776d7f803b42cd0727
218a1f8c7f222f3b5479f2c2d868680947c88e03
25c26df2b5de776fd9347b47fdeaa34deb00b9b4
e77cef0da7db7e95fb80ce7fe301ab7b3cdf4ad3
</code></pre></div></div>

<p>I added formatting to remove extra text and format on one line with the full commit for the purpose that I
want to pipe this into other commands that just use the commit. You can of course <a href="https://git-scm.com/docs">check out the documntation</a>
to see the gazillion and a half options and arguments and cheeznos that you can use to customize it for your need.</p>

<h3 id="search-commit-messages">Search commit messages</h3>

<p>Okay this is the coolest function - you can search for commits with regular expressions (grep!). For example,
here is an (unfortunate) string that I use sometimes…</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>git log <span class="nt">--all</span> <span class="nt">--oneline</span> <span class="nt">--pretty</span><span class="o">=</span>tformat:<span class="s2">"%H"</span> <span class="nt">--grep</span> <span class="s2">"oups"</span>
8d7e5a20d16fe739726939d9050b8ba9cc8af397
</code></pre></div></div>

<p>Okay not too bad, only one “oups” in this particular repo. But I assure you that there
are others out there…</p>

<h2 id="dates">Dates</h2>

<p>What about dates? You can get the date of your last commit like this:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>git show <span class="nt">-s</span> <span class="nt">--format</span><span class="o">=</span>%ci HEAD
2019-03-26 16:31:30 <span class="nt">-0400</span>
</code></pre></div></div>

<p>The last bit (-040) is the timezone offset. Don’t believe it? Take a look at <code class="highlighter-rouge">git log</code> to verify (the one at the top):</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>git log
commit 613a9eedaa1133fac6fd3325e5ed04e25d7bd025
Author: Vanessa Sochat &lt;vsochat@stanford.edu&gt;
Date:   Tue Mar 26 16:31:30 2019 <span class="nt">-0400</span>

    updating post
</code></pre></div></div>

<p>If you want to look at a specific commit, just change HEAD to that commit. For example:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>git show <span class="nt">-s</span> <span class="nt">--format</span><span class="o">=</span>%ci 75b688717d08d9364b93a4776d7f803b42cd0727
2019-03-25 13:17:47 <span class="nt">-0400</span>
</code></pre></div></div>

<h2 id="export-content">Export Content</h2>

<p>This is the function that makes it possible to turn a tiny git repository into a temporal database,
and this drives the <a href="https://vsoch.github.io/watchme/getting-started/index.html#how-do-i-export-data">export function</a> of the watchme tool.
Here is what that looks like. Seriously, this is so easy and simple it’s going to be your favorite command!</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c"># git show                                 &lt;commit&gt;:&lt;filename&gt;</span>
<span class="nv">$ </span>git show 75b688717d08d9364b93a4776d7f803b42cd0727:Gemfile
<span class="nb">source</span> <span class="s2">"https://rubygems.org"</span>

<span class="c">#gem "rails"</span>
gem <span class="s1">'github-pages'</span>
gem <span class="s1">'jekyll'</span>
</code></pre></div></div>

<p>Yep, it just splots it out into the terminal. You can do whatever you like with it.</p>

<h2 id="all-together-now">All Together Now!</h2>

<p>First, I’ll show you what an application can do to put these commands together. As an example with 
watchme, there is an easy way to export temporal data from a git repository. For each entry, there is a commit,
 a date, and then the content. The example below only shows two commits (entries). You can
tell that the task is set to run on the hour.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c"># watchme export &lt;watcher&gt;   &lt;task&gt;           &lt;filename&gt;</span>
<span class="nv">$ </span>watchme <span class="nb">export </span>system task-memory vanessa-thinkpad-t460s_vanessa.json <span class="nt">--json</span>
</code></pre></div></div>
<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">{</span>
    <span class="s">"commits"</span><span class="p">:</span> <span class="p">[</span>
        <span class="s">"02bccc9b0dbbd885125ae653fa5034dbf1d15eb4"</span><span class="p">,</span>
        <span class="s">"d98aaaae49c2c5106393beff5ebb51225eba8ac6"</span><span class="p">,</span>
        <span class="o">...</span>
    <span class="p">],</span>
    <span class="s">"dates"</span><span class="p">:</span> <span class="p">[</span>
        <span class="s">"2019-04-07 15:00:02 -0400"</span><span class="p">,</span>
        <span class="s">"2019-04-07 14:00:02 -0400"</span><span class="p">,</span>
       <span class="o">...</span>
    <span class="p">],</span>
    <span class="s">"content"</span><span class="p">:</span> <span class="p">[</span>
        <span class="p">{</span>
            <span class="s">"virtual_memory"</span><span class="p">:</span> <span class="p">{</span>
                <span class="s">"total"</span><span class="p">:</span> <span class="mi">20909223936</span><span class="p">,</span>
                <span class="s">"available"</span><span class="p">:</span> <span class="mi">6038528000</span><span class="p">,</span>
                <span class="s">"percent"</span><span class="p">:</span> <span class="mf">71.1</span><span class="p">,</span>
                <span class="s">"used"</span><span class="p">:</span> <span class="mi">13836861440</span><span class="p">,</span>
                <span class="s">"free"</span><span class="p">:</span> <span class="mi">201441280</span><span class="p">,</span>
                <span class="s">"active"</span><span class="p">:</span> <span class="mi">16094294016</span><span class="p">,</span>
                <span class="s">"inactive"</span><span class="p">:</span> <span class="mi">3581304832</span><span class="p">,</span>
                <span class="s">"buffers"</span><span class="p">:</span> <span class="mi">3842781184</span><span class="p">,</span>
                <span class="s">"cached"</span><span class="p">:</span> <span class="mi">3028140032</span><span class="p">,</span>
                <span class="s">"shared"</span><span class="p">:</span> <span class="mi">736833536</span>
            <span class="p">}</span>
        <span class="p">},</span>
        <span class="p">{</span>
            <span class="s">"virtual_memory"</span><span class="p">:</span> <span class="p">{</span>
                <span class="s">"total"</span><span class="p">:</span> <span class="mi">20909223936</span><span class="p">,</span>
                <span class="s">"available"</span><span class="p">:</span> <span class="mi">6103392256</span><span class="p">,</span>
                <span class="s">"percent"</span><span class="p">:</span> <span class="mf">70.8</span><span class="p">,</span>
                <span class="s">"used"</span><span class="p">:</span> <span class="mi">13769531392</span><span class="p">,</span>
                <span class="s">"free"</span><span class="p">:</span> <span class="mi">202334208</span><span class="p">,</span>
                <span class="s">"active"</span><span class="p">:</span> <span class="mi">16014094336</span><span class="p">,</span>
                <span class="s">"inactive"</span><span class="p">:</span> <span class="mi">3663310848</span><span class="p">,</span>
                <span class="s">"buffers"</span><span class="p">:</span> <span class="mi">3859390464</span><span class="p">,</span>
                <span class="s">"cached"</span><span class="p">:</span> <span class="mi">3077967872</span><span class="p">,</span>
                <span class="s">"shared"</span><span class="p">:</span> <span class="mi">755183616</span>
            <span class="p">}</span>
        <span class="p">},</span>
        <span class="o">...</span>
    <span class="p">]</span>
<span class="p">}</span>
</code></pre></div></div>

<p>But let’s walk through how you can do simple, fun things on the command line.</p>

<h3 id="export-list-of-dates">Export List of Dates</h3>

<p>Here is how you could export all the dates for your commits:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">for </span>commit <span class="k">in</span> <span class="k">$(</span>git log <span class="nt">--all</span> <span class="nt">--oneline</span> <span class="nt">--pretty</span><span class="o">=</span>tformat:<span class="s2">"%H"</span><span class="k">)</span>
   <span class="k">do </span>git show <span class="nt">-s</span> <span class="nt">--format</span><span class="o">=</span>%ci <span class="nv">$commit</span>
<span class="k">done

</span>2019-04-08 11:00:02 <span class="nt">-0400</span>
2019-04-08 11:00:02 <span class="nt">-0400</span>
2019-04-08 11:00:02 <span class="nt">-0400</span>
...
</code></pre></div></div>

<h3 id="export-temporal-data">Export Temporal Data</h3>

<p>Or you could do the same, but dump results for each (be careful doing this that
you don’t overwrite data files.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>mkdir <span class="nt">-p</span> <span class="nb">history
</span><span class="k">for </span>commit <span class="k">in</span> <span class="k">$(</span>git log <span class="nt">--all</span> <span class="nt">--oneline</span> <span class="nt">--pretty</span><span class="o">=</span>tformat:<span class="s2">"%H"</span><span class="k">)</span>
   <span class="k">do </span>git show <span class="nv">$commit</span>:README.md <span class="o">&gt;</span> <span class="nb">history</span>/<span class="nv">$commit</span>.txt
<span class="k">done</span>
</code></pre></div></div>
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span><span class="nb">ls history</span>/
0e153d64368a1f9ba55f4c406ae0b0dcd6e55a8f.txt
10dfc058c461152dc4588417a1ee1311423c8dd4.txt
1109ddd7d94051e1e3af3628b09ed2c30bc70696.txt
...
</code></pre></div></div>

<p>There you go! Your little git repo has now served as a database for (whatever your
special files happen to be) and you can go to town doing some analysis with them.</p>

<p>As another quick mention (and final note) a developer that I admire, <a href="https://github.com/cyphar">@cyphar</a>,
created a <a href="https://asciinema.org/a/Cfl6HLqYxpcUfRbBli6SbF5Gg">nice demonstration</a> of how to 
rebase and squash commits. This was another thing that I wasn’t aware was so easy 
to do on the command line. Thanks Aleksa!</p>