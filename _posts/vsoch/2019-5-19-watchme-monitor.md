---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2019-05-19 06:30:00'
layout: post
original_url: https://vsoch.github.io//2019/watchme-monitor/
title: Watchme Terminal Monitor
---

<p>We don’t care enough about resource usage. If there could be unique patterns associated with
running different software, wouldn’t it be lucrative to study them, and then classify an unknown process?
Or to predict resource usage given the programs involved? It’s a hard problem, but it’s cool enough
that I want to talk about it, and show you some fun I had today thinking about it.</p>

<p>I’ve been working on a tool to monitor resource usage called <a href="https://vsoch.github.io/watchme/watchers/psutils">watchme</a>,
and on Friday I released a version with not only a Python decorator and task, but also a terminal monitor 
that will allow you to run watchme on the fly for <strong>any</strong> process that you launch. You can still
specify an interval t o record at, and filter the metrics however you please.
If you’ve used <a href="https://www.gnu.org/software/time/">GNU time</a>, it’s similar in usage to that. For example, here I am going to
monitor the sleep command, and take a recording every second:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>watchme monitor sleep 10 <span class="nt">--seconds</span> 1
</code></pre></div></div>

<p>If you are interested, <a href="https://asciinema.org/a/247178?speed=2">here is an asciinema</a> video of that in action.
But let’s skip over the dummy examples and jump into something a little more fun - using
watchme to:</p>

<ul>
  <li><a href="#monitoring-container-pulls">Monitor Container Pulls</a> on the Sherlock cluster using Singularity</li>
  <li><a href="#measure-memory-usage">Measure Memory Usage</a> for a containerized sklearn model.</li>
  <li><a href="#why-should-i-care">Why Should I Care?</a> and then talk about why in the world you should care at all.</li>
</ul>

<p><br /></p>

<p>Feel free to jump around if one is more interesting to you.</p>

<p><br /></p>

<h1 id="monitoring-container-pulls">Monitoring Container Pulls</h1>

<p>I wanted to collect resource usage during a Singularity pull of several
containers including ubuntu, busybox, centos, alpine, and nginx. I chose these fairly randomly.
The goal was to create plots, taking a measurement each second, and
asking a very basic question:</p>

<blockquote>
  <p>Is there varying performance based on the amount of memory available?</p>
</blockquote>

<p>This meant that I launched a job, and manipulated only the amount of memory. Here
is my quick submission loop (the sbatch command submits the job in the file pull-job.sh):</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="k">for </span>iter <span class="k">in </span>1 2 3 4 5<span class="p">;</span> <span class="k">do
    for </span>name <span class="k">in </span>ubuntu busybox centos alpine nginx<span class="p">;</span> <span class="k">do
        for </span>mem <span class="k">in </span>4 6 8 12 16 18 24 32 64 128<span class="p">;</span> <span class="k">do
            </span><span class="nv">output</span><span class="o">=</span><span class="s2">"</span><span class="k">${</span><span class="nv">outdir</span><span class="k">}</span><span class="s2">/</span><span class="k">${</span><span class="nv">name</span><span class="k">}</span><span class="s2">-iter</span><span class="k">${</span><span class="nv">iter</span><span class="k">}</span><span class="s2">-</span><span class="k">${</span><span class="nv">mem</span><span class="k">}</span><span class="s2">gb.json"</span>
            <span class="nb">echo</span> <span class="s2">"sbatch --mem=</span><span class="k">${</span><span class="nv">mem</span><span class="k">}</span><span class="s2">GB pull-job.sh </span><span class="k">${</span><span class="nv">mem</span><span class="k">}</span><span class="s2"> </span><span class="k">${</span><span class="nv">iter</span><span class="k">}</span><span class="s2"> </span><span class="k">${</span><span class="nv">name</span><span class="k">}</span><span class="s2"> </span><span class="k">${</span><span class="nv">output</span><span class="k">}</span><span class="s2">"</span>            
            sbatch <span class="nt">--mem</span><span class="o">=</span><span class="k">${</span><span class="nv">mem</span><span class="k">}</span>GB pull-job.sh <span class="s2">"</span><span class="k">${</span><span class="nv">mem</span><span class="k">}</span><span class="s2">"</span> <span class="s2">"</span><span class="k">${</span><span class="nv">iter</span><span class="k">}</span><span class="s2">"</span> <span class="s2">"</span><span class="k">${</span><span class="nv">name</span><span class="k">}</span><span class="s2">"</span> <span class="k">${</span><span class="nv">output</span><span class="k">}</span>
        <span class="k">done
    done
done</span>

</code></pre></div></div>

<p>and then “pull-job.sh” collected the input arguments, and pulled the container on
the node:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nv">mem</span><span class="o">=</span><span class="k">${</span><span class="nv">1</span><span class="k">}</span>
<span class="nv">iter</span><span class="o">=</span><span class="k">${</span><span class="nv">2</span><span class="k">}</span>
<span class="nv">name</span><span class="o">=</span><span class="k">${</span><span class="nv">3</span><span class="k">}</span>
<span class="nv">output</span><span class="o">=</span><span class="k">${</span><span class="nv">4</span><span class="k">}</span>

<span class="c"># Add variables for host, cpu, etc.</span>
<span class="nb">export </span><span class="nv">WATCHMEENV_HOSTNAME</span><span class="o">=</span><span class="k">$(</span>hostname<span class="k">)</span>
<span class="nb">export </span><span class="nv">WATCHMEENV_NPROC</span><span class="o">=</span><span class="k">$(</span>nproc<span class="k">)</span>
<span class="nb">export </span><span class="nv">WATCHMEENV_MAXMEMORY</span><span class="o">=</span><span class="k">${</span><span class="nv">mem</span><span class="k">}</span>
watchme monitor singularity pull <span class="nt">--force</span> docker://<span class="nv">$name</span> <span class="nt">--name</span> <span class="nv">$name</span>-<span class="nv">$iter</span> <span class="nt">--seconds</span> 1 <span class="o">&gt;</span> <span class="k">${</span><span class="nv">output</span><span class="k">}</span>

</code></pre></div></div>

<p>Notice how the “singularity pull” command is wrapped with “watchme monitor” - this is
how I’m handing off the process for watchme to run and watch. For this approach, 
I installed watchme, and opted to pipe results directly into files named according to the parameters.
The full set of output files are <a href="https://github.com/singularityhub/watchme-singularity-pull/tree/master/data">here</a>.
Most of these pulls are between 4 and 10 seconds, so there isn’t a ton of data recorded, but I’ll quickly show an example 
of what I found. First, let’s look at cpu time in user space during the pull of alpine. 
What is cpu time in user space, as opposed to system / kernel space? It’s the amount of time 
[<a href="https://psutil.readthedocs.io/en/latest/#psutil.cpu_times">1</a>][<a href="https://en.wikipedia.org/wiki/CPU_time">2</a>] that
the processer spends pulling our container. A higher value for this metric means that
the process is taking more time. I would expect that asking for less memory for a job 
corresponds with getting less user CPU time. And this (might be?) what we see - here is a pull
for alpine.</p>

<p><img src="/assets/images/posts/watchme/alpine-cpu-times-user.png" alt="/assets/images/posts/watchme/alpine-cpu-times-user.png" /></p>

<p>Yeah, I was a bit lazy to just show all the iterations on the same plot. It’s a bit all over
the place, and hard to make any sort of conclusion. But what I do find interesting is the kink
in the plot at around 2 seconds. I would guess that Singularity starts running, and at around 2 seconds starts to do something
(slightly) more CPU intensive, like extraction of layers and then building the SIF binary.
Sure, the units of change are very small, but we can watch a pull to see how behavior
(represented by the terminal logging) corresponds with what we’ve measured:</p>

<script id="asciicast-247185" src="https://asciinema.org/a/247185.js" async=""></script>

<p>Did you see that? The first two seconds when we were “Starting Build” likely correspond with
the first rise in the graph. Then when we pull and extract layers, albeit the change being small,
we demand (and get) more user CPU time. You can also see higher user CPU times for a
beefier image like <a target="_blank" href="/assets/images/posts/watchme/ubuntu-cpu-times-user.png">ubuntu</a>.</p>

<p><br /></p>

<h1 id="measure-memory-usage">Measure Memory Usage</h1>

<p>Let’s step it up a notch, and try measuring the training of a model. I’ve put it in a container. I want to run it
in parallel on my cluster, but I have no idea how much memory to ask for!</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nv">$ </span>sbatch <span class="nt">--partition</span> owners <span class="nt">--mem</span><span class="o">=</span>??? job.sh

</code></pre></div></div>

<h2 id="1-prepare-your-analysis">1. Prepare your Analysis</h2>

<p>Can watchme help? Yes, I think so! I will sheepishly admit that I had maybe a couple
of job submission scripts in graduate school, and I rarely changed the amount of memory
that I asked for. I always set it at some high value that I was sure wouldn’t poop out.
But actually, I’d have been able to run more jobs and to use the cluster resources
more optimally if I had just spent a little time to accurately estimate memory for
my jobs. Let’s do this now. I started with this 
<a href="https://scikit-learn.org/stable/auto_examples/neural_networks/plot_mnist_filters.html#sphx-glr-auto-examples-neural-networks-plot-mnist-filters-py">sklearn mnist example</a>,
and built it into a container:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
FROM continuumio/miniconda3

<span class="c"># docker build -t vanessa/watchme-mnist .</span>
<span class="c"># docker push vanessa/watchme-mnist</span>

RUN apt-get update <span class="o">&amp;&amp;</span> apt-get install <span class="nt">-y</span> git
RUN conda install scikit-learn matplotlib
ADD run.py /run.py
ENTRYPOINT <span class="o">[</span><span class="s2">"python"</span>, <span class="s2">"/run.py"</span><span class="o">]</span>

</code></pre></div></div>

<p>and served it at <a href="https://hub.docker.com/r/vanessa/watchme-mnist">vanessa/watchme-mnist</a>.</p>

<h2 id="2-testing-environment">2. Testing Environment</h2>

<p>First, I’m going to grab an interactive node on my cluster. I could use sdev, but
I want to ask for a bit more time and memory than comes by default.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nv">$ </span>srun <span class="nt">--mem</span><span class="o">=</span>32GB <span class="nt">--time</span><span class="o">=</span>24:00:00 <span class="nt">--pty</span> bash

</code></pre></div></div>

<p>and pull the container.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">$ </span>singularity pull docker://vanessa/watchme-mnist
</code></pre></div></div>

<p>I first tried running watchme on the container to collect metrics:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nv">$ </span>watchme monitor <span class="nt">--seconds</span> 1 singularity run watchme-mnist_latest.sif plots.png <span class="o">&gt;</span> mnist-external.json

</code></pre></div></div>

<p>I strangely found in the <a href="https://raw.githubusercontent.com/vsoch/watchme-mnist/master/mnist-external.json">data export</a> 
that after the first call to <code class="highlighter-rouge">singularity</code>, we weren’t able to derive much from the process that we execv’d to 
named <code class="highlighter-rouge">starter-suid</code>. This means that we need to install the monitor inside the container:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
FROM continuumio/miniconda3

<span class="c"># docker build -t vanessa/watchme-mnist .</span>
<span class="c"># docker push vanessa/watchme-mnist</span>

RUN apt-get update <span class="o">&amp;&amp;</span> apt-get install <span class="nt">-y</span> git
RUN conda install scikit-learn matplotlib memory_profiler
RUN pip install watchme
ADD run.py /run.py
ENTRYPOINT <span class="o">[</span><span class="s2">"python"</span>, <span class="s2">"/run.py"</span><span class="o">]</span>

</code></pre></div></div>

<p>Re-pull our container</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nv">$ </span>singularity pull docker://vanessa/watchme

</code></pre></div></div>

<p>and try again! This is a learning experience for both of us. I didn’t
anticipate that I wouldn’t be able to measure inside the container. In retrospect, 
it makes sense. Here is our updated command - notice that singularity is run first, and 
the process we exec is for watchme to monitor our script.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nv">$ </span>singularity <span class="nb">exec </span>watchme-mnist_latest.sif watchme monitor <span class="nt">--seconds</span> 1 python /run.py plots.png <span class="o">&gt;</span> mnist.json

</code></pre></div></div>

<p>In the above, I monitored the command to run the container, <code class="highlighter-rouge">singularity run watchme-mnist_latest.sif</code>,
from inside of the container. I asked watchme to record all metrics every 1 second, and I piped the json
result into <a href="https://raw.githubusercontent.com/vsoch/watchme-mnist/master/mnist.json">a file</a>.
Thank goodness that Singularity has seamless connection to the host (binds, environment), because
I could easily do this. I could then make a few simple plots to look at memory.</p>

<p><br /></p>

<p><img src="/assets/images/posts/watchme/mnist-all.png" alt="/assets/images/posts/watchme/mnist-all.png" /></p>

<p>Oh this is so neat! Well, what do we see off the bat?</p>

<h3 id="how-much-memory-is-the-process-using">How much memory is the process using?</h3>

<p>Out of the <a href="https://psutil.readthedocs.io/en/latest/#psutil.Process.memory_full_info">memory metrics</a> that psutils
can measure, only a few of them are non-zero. According to the docs and <a href="http://grodola.blogspot.com/2016/02/psutil-4-real-process-memory-and-environ.html">here</a>, unique set size is probably the best representative of the process memory usage.</p>

<blockquote>
  <p><em>Unique set size (uss)</em>. In computing, unique set size (USS) is the portion of main memory (RAM) occupied by a process which is guaranteed to be private to that process.</p>
</blockquote>

<h3 id="how-much-memory-is-available-total">How much memory is available, total?</h3>

<p>I was confused to see only 1.7GB for virtual memory size, because I thought I had asked for more.
First, I decided to look at the maximum value of the virtual memory size, “1722089472” (this is in bytes). Let’s zoom in on the chart above.</p>

<p><br /></p>

<p><img src="/assets/images/posts/watchme/mnist.png" alt="/assets/images/posts/watchme/mnist.png" /></p>

<p>As we can see, the maximum is around 1.7, which is 1.7GB. But… didn’t I ask for more?
Let’s look more closely at the node we were on. I didn’t specify the number of processing units that I got, so I got…</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="o">[</span>vsochat@sh-108-42 ~/.watchme/mnist]<span class="nv">$ </span>nproc
1

</code></pre></div></div>

<p>Just 1! And then to confirm what we see in the plot, we can look at <code class="highlighter-rouge">/proc/meminfo</code>:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="o">[</span>vsochat@sh-108-42 ~/.watchme/mnist]<span class="nv">$ </span><span class="nb">cat</span> /proc/meminfo
MemTotal:       196438172 kB
MemFree:        164311444 kB
MemAvailable:   169037160 kB
Buffers:             492 kB
Cached:          4682540 kB
SwapCached:         2660 kB
Active:          3281700 kB
Inactive:        3413948 kB
Active<span class="o">(</span>anon<span class="o">)</span>:    2183656 kB
Inactive<span class="o">(</span>anon<span class="o">)</span>:   204200 kB
Active<span class="o">(</span>file<span class="o">)</span>:    1098044 kB
Inactive<span class="o">(</span>file<span class="o">)</span>:  3209748 kB
Unevictable:       89784 kB
Mlocked:           89796 kB
SwapTotal:       4194300 kB
SwapFree:        4170720 kB
Dirty:                20 kB
Writeback:             0 kB
AnonPages:       2099752 kB
Mapped:            38748 kB
Shmem:            362580 kB
Slab:           22616180 kB
SReclaimable:    1168932 kB
SUnreclaim:     21447248 kB
KernelStack:       17760 kB
PageTables:        10796 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:    102413384 kB
Committed_AS:    2898952 kB
VmallocTotal:   34359738367 kB
VmallocUsed:     1669384 kB
VmallocChunk:   34257489200 kB
HardwareCorrupted:     0 kB
AnonHugePages:         0 kB
CmaTotal:              0 kB
CmaFree:               0 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
DirectMap4k:      624448 kB
DirectMap2M:    26251264 kB
DirectMap1G:    175112192 kB

</code></pre></div></div>

<p>See at the top, how “MemFree” and “MemTotal” is between 164311444 and 196438172 kB? It
would sort of make sense to get a maximum virtual memory somewhere between those two, as we did.
So for the node that I ran the container on, although I asked for 32GB, I got about half
of that. Strange.</p>

<h2 id="what-did-i-learn">What did I learn?</h2>

<p><strong>SLURM Seems Messy</strong></p>

<p>A lot of times when you <em>think</em> you are asking
for a specific resource allocation, if you aren’t specific about everything from memory to number
of processes, you are likely to not get exactly what you think. In my case, the memory argument was
totally useless because I got half of what I asked for. Further, it sort of seems like
the nodes vary widely in their actual configurations, and when I think about it, this makes sense too. 
They are added slowly over time, with varying models and configurations depending on the labs that funded them.
I would even bet that the memory argument isn’t enforced beyond SLURM possibly watching the process, and just killing it if it goes over. I wonder
what this means for shared jobs on a node? The whole setup just seems messy, especially if you
are used to bringing up a cloud instance, and generally knowing that you have the entire thing.
I remember that SLURM (used to?) have an exclusive flag, now it makes sense why someone would use it.
After this exercise, I strangely have more sympathy for graduate school version of me that didn’t
spend too much time optimizing job submissions. It seems to be messy anyway, might as well ask
for more than you need and pray.</p>

<p><strong>Containers Isolate some Metrics</strong></p>

<p>As we mentioned earlier, containers isolate some metrics from the host. I should have remembered
this would be the case, but I didn’t! To make using watchme easier for you, I’ve provided
 <a href="https://hub.docker.com/r/vanessa/watchme">Docker bases</a> that come ready to go with watchme so you
can easily monitor processes inside of containers, also from inside of the container.</p>

<p><strong>Tracking Actual Memory is Useful</strong></p>

<p>Regardless of what SLURM gives me, tracking actual resources is an interesting practice.
What I learned today is that If I want to track memory usage for a process, I should look at “memory_full_info” -&gt; “uss” and compare to
what (the container sees) as the total available, “memory_full_info” -&gt; “vms.” If you want
some (rough) code to do this, see <a href="https://github.com/vsoch/watchme-mnist/blob/master/mnist.ipynb">my notebook here</a>.
One thing I’m thinking of now is that it would be useful to have some ready-to-go scripts
to parse the output. If you are interested in this, please <a href="https://www.github.com/vsoch/watchme/issues">open an issue</a>.</p>

<h1 id="why-should-i-care">Why should I care?</h1>

<p><strong>Asking for Resources</strong></p>

<p>I was originally going to say that you should care because it would
allow you to more efficiently ask for resources, but I no longer find that a compelling answer.</p>

<p><strong>Machine Learning to Categorize Software</strong></p>

<p>If you are someone that is interested in data science or machine learning, there is a trove
of work to be done in this area. Take a look at these plots. Or better yet, take a look
at the <a href="https://raw.githubusercontent.com/vsoch/watchme-mnist/master/mnist.json">metrics</a> 
that I didn’t get to touch on, including io operations, cpu, connections, and many more I have
yet to read about. In the same way that we saw a logical pattern in the Singularity pull data,
I would hypothesize that we could associate patterns with different software or programs, and then
be able to see an unknown entity running, and detect if any of the patterns that
we know are present. For example, let’s say that a script starts with pulling a Singularity
container. Might it be possible to detect the pattern? It’s akin to how YouTube analyzes
copyright music from your video audio track. It’s a really cool idea, and I think with
software put in place to collect metrics, and then a large collection of data, this would
be an awesomet thing to do.</p>