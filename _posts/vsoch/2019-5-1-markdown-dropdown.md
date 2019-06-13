---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2019-05-01 08:30:00'
layout: post
original_url: https://vsoch.github.io//2019/markdown-dropdown/
title: Markdown Details
---

<p>This is a quick post to share a highly useful trick for posting long
error logs or similar on GitHub issues or any spot with markdown. The amazing
discovery <a href="https://github.com/RunestoneInteractive/RunestoneServer/pull/1229#issuecomment-488315755">comes by way</a> 
of my colleage, <a href="https://github.com/yarikoptic">@yarikoptic</a>. Here is a 
<a href="https://github.com/RunestoneInteractive/RunestoneServer/issues/1204#issue-407541603">GitHub example</a> out in the wild, again from my colleage,
and here is a live example of what it looks like in this blog post:</p>

<details>

This is top secret text! Or more likely, some really verbose error log that<br />
only a tiny fraction of us need to see. Inspect a container? Sure, why not!<br />

<pre><code>
$ singularity inspect salad_latest.sif
==labels==
org.label-schema.build-date: Thursday_11_April_2019_9:13:20_EDT
org.label-schema.schema-version: 1.0
org.label-schema.usage.singularity.deffile.bootstrap: docker
org.label-schema.usage.singularity.deffile.from: vanessa/salad
org.label-schema.usage.singularity.version: 3.1.0-rc2.1154.g479352901
</code></pre>
</details>

<h3 id="basic-example">Basic Example</h3>
<p>What would the code look like to do this?</p>

<div class="language-html highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nt">&lt;details&gt;</span>

This is top secret text! Or more likely, some really verbose error log that
only a tiny fraction of us need to see. Inspect a container? Sure, why not!

$ singularity inspect salad_latest.sif
==labels==
org.label-schema.build-date: Thursday_11_April_2019_9:13:20_EDT
org.label-schema.schema-version: 1.0
org.label-schema.usage.singularity.deffile.bootstrap: docker
org.label-schema.usage.singularity.deffile.from: vanessa/salad
org.label-schema.usage.singularity.version: 3.1.0-rc2.1154.g479352901

<span class="nt">&lt;/details&gt;</span>
</code></pre></div></div>

<h3 id="add-a-title">Add a Title</h3>

<p>You can add a title with the <code class="highlighter-rouge">&lt;summary&gt;&lt;/summary&gt;</code> set of tags.</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>&lt;details&gt;
  &lt;summary&gt;Error Log&lt;/summary&gt;

   more...

&lt;/details&gt;
</code></pre></div></div>

<h3 id="open-by-default">Open by Default</h3>

<p>You can also make the dropdown box “open” by default.</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>&lt;details open&gt;
  &lt;summary&gt;Error Log&lt;/summary&gt;

   YOU MUST READ THIS TEXT :X
   more...

&lt;/details&gt;
</code></pre></div></div>

<h3 id="formatting">Formatting</h3>

<p>Writing this into an html page, you have to include the contents of the details
box in paragraphs or with line breaks. However on GitHub, the lines of markdown
are formatted as such, and so you don’t need these extra tags.
For example, you can add formatting for your code, of course.</p>

<h2 id="how-do-i-remember-this">How do I remember this?</h2>

<p>Just remember <code class="highlighter-rouge">&lt;details&gt;&lt;/details&gt;</code> and write code and content between these tags.
My colleague mentioned that it’s good to have an empty line at the top, so if you run
into issues try that. <a href="https://gist.github.com/vsoch/1235f639d50d358a017abce651580435">Here is a gist</a>
I put together so you can see both rendered and code examples.</p>

<h2 id="why-does-this-work">Why does this work?</h2>

<p>Details isn’t a markdown trick, or a GitHub (or similar) feature, it’s
actually a full fledged <a href="https://www.w3schools.com/tags/tag_details.asp">html tag</a>
that has almost full browser support (it doesn’t work Internet Explorer / Edge).
The initial tag was added to the HTML 5.1 specification, and is cutely
referred to as “a disclosure box.” Read more <a href="https://developer.mozilla.org/en-US/docs/Web/HTML/Element/details">about the details tag here</a>,
and let’s start seeing these handy boxes used in GitHub issues to clean up the threads,
and make them easier to navigate.</p>