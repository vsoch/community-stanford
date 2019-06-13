---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2019-03-21 07:30:00'
layout: post
original_url: https://vsoch.github.io//2019/collection-tree-filesystem/
title: Collection Tree Filesystems
---

<p>The <a href="https://singularityhub.github.io/container-tree/">containertree</a> software
aims to support and encourage research that can help us to better
optimize container development, distribution, and inheritance.
Specifically, it provides various tree data structures to describe 
different facets of containers. I won’t go into detail because I’ve 
<a href="https://vsoch.github.io/2018/container-tree/">discussed</a> these
data structures, visualizations, and functions previously, but I’ll briefly
review each one:</p>

<h2 id="container-filesystem-trees">Container Filesystem Trees</h2>

<p><a href="https://singularityhub.github.io/container-tree/pages/demo/files_tree/">Container Filesystem Trees</a> are exactly what you think - a tree data structure where nodes are folders and files within a single container. One or more containers or tags can be mapped to the tree to allow for easy search, trace, or comparison of a container or tag across the tree. I’m tickled by the fact that a Container Filesystem tree technically <em>is</em> a container.
The structure you interact with via the library mirrors what you see as the actual filesystem. This data structure should be interesting
to you if you want to study the guts of a single container, or compare guts between one or more containers.</p>

<h2 id="container-package-trees">Container Package Trees</h2>

<p><a href="https://singularityhub.github.io/container-tree/examples/package_tree/">Container Package Trees</a> keep track of packages and versions for one or more containers, and make it easy to <a href="https://singularityhub.github.io/container-tree/examples/export_data/">export data frames</a> of that data. I’ve been doing a cool analysis that shows a nice (expected) relationship between
packages and container inheritance, but I’ll save that for a later post. The software can easily export package metadata by using
<a href="https://github.com/GoogleContainerTools/container-diff">container-diff</a> under the hood.</p>

<p><br /></p>

<h1 id="collection-trees">Collection Trees</h1>

<p>Okay let’s get excited, because I’ve been working heavily on developing the 
<a href="https://singularityhub.github.io/container-tree/examples/collection_tree/">Collection Tree</a> 
data structure, and it’s super cool. A collection tree represents an inheritance structure 
for a family of containers. This means that the root node is <a href="https://hub.docker.com/_/scratch">scratch</a>, 
the first level likely includes containers in the <a href="https://hub.docker.com/u/library/">docker library</a> 
(e.g., think of <code class="highlighter-rouge">library/ubuntu:16.04</code> or similar)
and the children inherit from that. These are of course the defaults (you can enforce your tree
to have a first level of “chewbacca” if that floats your boat) that can be easily modified.
With a collection tree you can:</p>

<ol class="custom-counter">
<li>Trace, search, or find a particular container</li>
<li>Compare containers based on distances in the tree</li>
<li>Export your tree as an <strong>actual</strong> filesystem</li>
</ol>

<p><br /></p>

<p>Let’s walk through an example to show you what I mean! I’ve already shown you
creating trees, and searching, removing, or tracing nodes, so today let’s talk about
the last bullet. The last bullet is fresh off the keyboard, and it’s about
creating collection tree filesystems.</p>

<h2 id="collection-tree-filesystem">Collection Tree Filesystem</h2>

<p>The more I thought about it, akin to the container tree, the Collection Tree itself also 
mapped really well to a filesystem. But instead of actual files and folders, the files
and folders of the Collection Tree Filesystem represent the collection namespaces and tags.
So I wrote a function to export the nodes as paths:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="k">for</span> <span class="n">path</span> <span class="ow">in</span> <span class="n">tree</span><span class="o">.</span><span class="n">paths</span><span class="p">():</span>
    <span class="k">print</span><span class="p">(</span><span class="n">path</span><span class="p">)</span>

<span class="o">/</span><span class="n">scratch</span>
<span class="o">/</span><span class="n">scratch</span><span class="o">/</span><span class="n">library</span><span class="o">/</span><span class="n">debian</span>
<span class="o">/</span><span class="n">scratch</span><span class="o">/</span><span class="n">library</span><span class="o">/</span><span class="n">python</span>
<span class="o">/</span><span class="n">scratch</span><span class="o">/</span><span class="n">library</span><span class="o">/</span><span class="n">python</span><span class="o">/.</span><span class="n">latest</span><span class="o">/</span><span class="n">continuumio</span><span class="o">/</span><span class="n">miniconda3</span>
<span class="o">/</span><span class="n">scratch</span><span class="o">/</span><span class="n">library</span><span class="o">/</span><span class="n">python</span><span class="o">/.</span><span class="n">latest</span><span class="o">/</span><span class="n">continuumio</span><span class="o">/</span><span class="n">miniconda3</span><span class="o">/.</span><span class="n">latest</span><span class="o">/</span><span class="n">singularityhub</span><span class="o">/</span><span class="n">containertree</span>
<span class="o">/</span><span class="n">scratch</span><span class="o">/</span><span class="n">library</span><span class="o">/</span><span class="n">python</span><span class="o">/.</span><span class="n">latest</span><span class="o">/</span><span class="n">continuumio</span><span class="o">/</span><span class="n">miniconda3</span><span class="o">/.</span><span class="n">latest</span><span class="o">/</span><span class="n">singularityhub</span><span class="o">/</span><span class="n">singularity</span><span class="o">-</span><span class="n">cli</span>
<span class="o">/</span><span class="n">scratch</span><span class="o">/</span><span class="n">library</span><span class="o">/</span><span class="n">python</span><span class="o">/.</span><span class="n">latest</span><span class="o">/</span><span class="n">continuumio</span><span class="o">/</span><span class="n">miniconda3</span><span class="o">/.</span><span class="mf">1.0</span><span class="o">/</span><span class="n">childof</span><span class="o">/</span><span class="n">miniconda3</span>

</code></pre></div></div>

<p>and then from that I could easily create the structure <a href="http://hub.docker.com/r/vanessa/collection-tree-fs">in a container</a>.</p>

<h3 id="1-a-collection-tree-filesystem-container">1. A Collection Tree Filesystem Container</h3>

<p>Let’s shell into this collection tree filesystem container! I thought about creating this as a squashfs filesystem
that could be mounted from a container, but decided to just make another root (/scratch) in a traditional
Docker container to make it most readily usable:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nv">$ </span>docker run <span class="nt">-it</span> vanessa/collection-tree-fs

</code></pre></div></div>

<p>If you run the example above, be warned that I’ve built a tree with inheritance for over 100K 
Docker images I’ve extracted from 2017. The dummy example below is a fake tree that I used 
during testing to exemplify that. First, notice that when we shell inside, the 
working directory is scratch. This is the root of the filesystem tree:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="o">(</span>base<span class="o">)</span> root@c1e3d68bac93:/scratch# tree
<span class="nb">.</span>
└── library
    ├── debian
    │   └── tag-latest
    │       └── vanessa
    │           └── pancakes
    └── python
        └── tag-latest
            └── continuumio
                └── miniconda3
                    ├── tag-1.0
                    │   └── childof
                    │       └── miniconda3
                    └── tag-latest
                        └── singularityhub
                            ├── containertree
                            └── singularity-cli

12 directories, 0 files
</code></pre></div></div>

<p>You can <a href="https://github.com/singularityhub/container-tree/tree/master/examples/collection_tree">see here</a> 
to understand how I created the Collection Tree data structure,
and then exported it into a container. You basically need to know pairs of containers, a single
container, and then whatever it’s base happens to be. I had a set of Dockerfiles (with FROM statements)
that I also knew the unique resource identifiers for, so I was able to do this. This comes
from the <a href="https://vsoch.github.io/datasets/2018/dockerfiles/">Dockerfiles dataset</a> if anyone else is interested.</p>

<h3 id="2-collection-tree-filesystem-interaction">2. Collection Tree Filesystem Interaction</h3>

<p>This is another really cool part. Once you have your Container namespaces exported to
an actual filesystem, you can interact with them using standard linux command line tools! 
For example, here we are searching for a tag of interest across all collections:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nv">$ </span>find <span class="nb">.</span> <span class="nt">-name</span> tag-latest
./library/debian/tag-latest
./library/python/tag-latest
./library/python/tag-latest/continuumio/miniconda3/tag-latest

</code></pre></div></div>

<p>Or finding a collection namespace with a particular string (singularityhub):</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nv">$ </span>find <span class="nb">.</span> <span class="nt">-name</span> singularityhub
./library/python/tag-latest/continuumio/miniconda3/tag-latest/singularityhub

</code></pre></div></div>

<p>As a more substantial example, here I am using the (bigger) container <code class="highlighter-rouge">vanessa/collection-tree-fs</code>
and looking at all the ubuntu tags! Since the defaults exports tags as hidden folders (starting with “.”)
for this container I changed the “tag_prefix” variable so they would all be prefixed with “tag-“. I knew
that containers have a lot of tags, but I never imagined this many!</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="o">(</span>base<span class="o">)</span> root@60a8aa15ef1c:/scratch# <span class="nb">ls </span>library/ubuntu
tag-10.04    tag-15.04		  tag-bionic-20180426  tag-trusty-20150814  tag-xenial-20160125    tag-x...
tag-12.04    tag-15.10		  tag-devel	       tag-trusty-20160323  tag-xenial-20160331.1  tag-x...
tag-12.04.5  tag-16.04		  tag-latest	       tag-trusty-20160624  tag-xenial-20160809    tag-x...
tag-12.10    tag-16.10		  tag-lucid	       tag-trusty-20160711  tag-xenial-20160818    tag-x...
tag-13.04    tag-17.04		  tag-precise	       tag-trusty-20161214  tag-xenial-20160923.1  tag-x...
tag-13.10    tag-17.10		  tag-quantal	       tag-trusty-20170602  tag-xenial-20161010    tag-x...
tag-14.04    tag-18.04		  tag-raring	       tag-trusty-20170728  tag-xenial-20161213    tag-y...
tag-14.04.1  tag-artful		  tag-rolling	       tag-trusty-20170817  tag-xenial-20170119    tag-z...
tag-14.04.2  tag-artful-20170916  tag-saucy	       tag-utopic	    tag-xenial-20170214    tag-z...
tag-14.04.3  tag-artful-20171116  tag-trusty	       tag-vivid	    tag-xenial-20170510    tag-z...
tag-14.04.4  tag-artful-20180417  tag-trusty-20150427  tag-wily		    tag-xenial-20170619
tag-14.04.5  tag-bionic		  tag-trusty-20150528  tag-wily-20150829    tag-xenial-20170802
tag-14.10    tag-bionic-20180125  tag-trusty-20150612  tag-xenial	    tag-xenial-20170915

</code></pre></div></div>

<p>Here is how we can just count them:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="o">(</span>base<span class="o">)</span> root@60a8aa15ef1c:/scratch# <span class="nb">ls </span>library/ubuntu | wc <span class="nt">-l</span>
75

</code></pre></div></div>

<p>Here is a peek into one of the smaller tag folders, raring, which I believe is version 13.04 (a long time ago!)</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="o">(</span>base<span class="o">)</span> root@60a8aa15ef1c:/scratch# tree library/ubuntu/tag-raring/
library/ubuntu/tag-raring/
├── ewindisch
│   └── docker-bomb
└── leifw
    └── tokumx-builder-ubuntu-raring

4 directories, 0 files

</code></pre></div></div>

<p>What could you do with this filesystem? You could do something as simple as using it as a storage structure.
Some related file content could be stored in the correct container namespace folder, or possibly even
the container itself. If you are interested in calculating metrics, you might want to stick with
the collection tree data structure itself. But another cool thing you can do is to write metadata
to the filesystem directly. Let’s talk about that next.</p>

<h3 id="3-collection-tree-filesystem-metadata">3. Collection Tree Filesystem Metadata</h3>

<p>What about metadata? What got me really excited was discovering that
using filesystem <a href="https://en.wikipedia.org/wiki/Extended_file_attributes#Linux">xattrs</a>
I can add metadata to the actual nodes in the filesystem. First I tried doing it manually,
inside my container (I was too afraid to do this on my host). Here is the usage”</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
Usage: attr <span class="o">[</span><span class="nt">-LRSq</span><span class="o">]</span> <span class="nt">-s</span> attrname <span class="o">[</span><span class="nt">-V</span> attrvalue] pathname  <span class="c"># set value</span>
       attr <span class="o">[</span><span class="nt">-LRSq</span><span class="o">]</span> <span class="nt">-g</span> attrname pathname                 <span class="c"># get value</span>
       attr <span class="o">[</span><span class="nt">-LRSq</span><span class="o">]</span> <span class="nt">-r</span> attrname pathname                 <span class="c"># remove attr</span>
       attr <span class="o">[</span><span class="nt">-LRq</span><span class="o">]</span>  <span class="nt">-l</span> pathname                          <span class="c"># list attrs </span>
      <span class="nt">-s</span> reads a value from stdin and <span class="nt">-g</span> writes a value to stdout

</code></pre></div></div>

<p>We could do something like add a count at each node, meaning the number of times the container was added as a
parent or a child. You can do this in python or with the xattr library on the system.  Here is
how to add an attribute:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="o">(</span>base<span class="o">)</span> root@38ecc937efda:/scratch# attr <span class="nt">-s</span> maintainer <span class="nt">-V</span> vanessasaur library/debian/
Attribute <span class="s2">"maintainer"</span> <span class="nb">set </span>to a 11 byte value <span class="k">for </span>library/debian:
vanessasaur

</code></pre></div></div>
<p>The “V” means the value, and “-s” means set, so we use the general form <code class="highlighter-rouge">attr -s &lt;key&gt; -V &lt;value&gt; &lt;path&gt;</code>.
And then list the attribute:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="o">(</span>base<span class="o">)</span> root@38ecc937efda:/scratch# attr <span class="nt">-g</span> maintainer library/debian/
Attribute <span class="s2">"maintainer"</span> had a 11 byte value <span class="k">for </span>library/debian/:
vanessasaur

</code></pre></div></div>

<p>As another example, here is how to list attributes for an example library/debian folder that has
had a count added:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="o">(</span>base<span class="o">)</span> root@f379adcb3cb7:/scratch# attr <span class="nt">-l</span> library/debian/
Attribute <span class="s2">"count"</span> has a 3 byte value <span class="k">for </span>library/debian/

</code></pre></div></div>

<p>To actually get the count:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="o">(</span>base<span class="o">)</span> root@f379adcb3cb7:/scratch# attr <span class="nt">-g</span> count library/debian/
Attribute <span class="s2">"count"</span> has a 3 byte value <span class="k">for </span>library/debian/
3

</code></pre></div></div>

<p>How awesomely wicked cool is this! We can put metadata with our files! 
For those interested in using this with containertree, there is 
a <a href="https://github.com/xattr/xattr">python package</a> that would make integration
easy.</p>

<h2 id="final-thoughts">Final Thoughts</h2>

<p>I <strong>really</strong> want you to get excited about studying containers. 
While you are distracted with AI taking over the world, and everything touted as cool
on social media, I’m over here jumping up and down trying to get you interested in data
structures and organizational standards. Think about it -
containers are the new unit of reproducible analysis, and enterprise deployment. We need
to be thinking hard and carefully about mapping 
this universe of containers, or developing a rigorous method to not just randomly use a machine
learning container to achieve a goal, but how to optimize your choice. And nobody is studying
how different container bases map to different use cases. Do more people use alpine with 
GPUs, or something else? If I want to do some genomic analysis, can I quickly find
the best container to do that?</p>

<p>If anyone wants to do a fun project to ask some of these questions, I’m a dynamo dinosaur programmer, 
and I’m in! I submit a small <a href="https://joss.theoj.org/papers/f7b46a7f922b468e535adabc2337b330">paper</a>
for the software to be reviewed, and will follow up with a small analysis with package trees.
In the meantime, please contribute or provide feedback - the library is under development,
and I’m continually hoping to improve it.</p>