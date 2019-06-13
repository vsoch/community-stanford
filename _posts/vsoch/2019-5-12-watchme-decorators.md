---
author: Vanessasaurus
blog_subtitle: dinosaurs, programming, and parsnips
blog_title: VanessaSaurus
blog_url: https://vsoch.github.io//
category: vsoch
date: '2019-05-12 06:30:00'
layout: post
original_url: https://vsoch.github.io//2019/watchme-decorators/
title: Watchme Process Monitoring
---

<p>It’s always been kind of hard to measure resource usage when you are running
a script. What I’d want to do is not get metrics like memory, cpu, and io operations
for an entire host, but rather for a specific process.</p>

<h2 id="the-monitor-process-task">The Monitor Process Task</h2>

<p>What I wanted to do was monitor not an entire node, or a python script, but
a specific function that a user might be running. Toward this goal, I created
the <a href="https://vsoch.github.io/watchme/watchers/psutils/#1-the-monitor-pid-task">monitor pid task</a>,
where pid stands for a process id. It works in a few simple ways. You can
run it as a task - meaning that watchme will schedule it to collect metrics
for some (pre-determined) process id <i>or</i> name. Let’s say we wanted to
create a watcher called “system” and then create a task to monitor slack:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nv">$ </span>watchme create system
<span class="nv">$ </span>watchme add-task system task-monitor-slack <span class="nt">--type</span> psutils func@monitor_pid_task pid@slack

</code></pre></div></div>

<p>and then I would use the <code class="highlighter-rouge">watchme schedule</code> command to specify how often I want to
collect metrics. The schedule will use <a href="https://en.wikipedia.org/wiki/Cron">cron</a> to run the watcher at the
frequency you desired. What kind of result can you get? To give you a sense, here is an example
of a plot for one of the tasks from a <a href="https://github.com/vsoch/watchme-system">a system watcher</a>.
Yep, I created it, scheduled it, and forgot about it. It faithfully generates data for
me, a la cron:</p>

<p><img src="https://vsoch.github.io/watchme-system/task-memory-virtual_memory-total-available-percent-used-free.png" alt="https://vsoch.github.io/watchme-system/task-memory-virtual_memory-total-available-percent-used-free.png" /></p>

<p>That’s the virtual memory that is free on my computer, in bytes, for the span of just over
a month that the task has been running. It’s only a month of data, but there are still
interesting patterns. What are the spikes? It could be that the spikes (more free memory) indicate
a restart of my computer. It might also have to do with whatever I was running, here
is a chart to show cpu percent usage:</p>

<p><img src="https://vsoch.github.io/watchme-system/task-cpu-cpu_freq-cpu_percent.png" alt="https://vsoch.github.io/watchme-system/task-cpu-cpu_freq-cpu_percent.png" /></p>

<p>But I digress! With the monitor process task, you can create similar plots to
these, but instead of for your entire computer, for a Python function or specific
process that you are interested in. See an <a href="https://gist.github.com/vsoch/19957205764ab12a153ddbecd837ffb3#file-result-json">example here</a> 
to see what is exported for each run. By the way, if you do run this task for slack,
take a look at the “cmdline” key. The command used to start up slack is ridiculous.</p>

<h2 id="the-monitor-process-decorator">The Monitor Process Decorator</h2>

<p>The task is pretty cool if you want to schedule monitoring for a process name or
id in advance, but what if you want to run something on the fly? I decided to
solve this issue by way of creating a decorator. Here’s what that looks like:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="kn">from</span> <span class="nn">watchme.watchers.psutils.decorators</span> <span class="kn">import</span> <span class="n">monitor_resources</span>
<span class="kn">from</span> <span class="nn">time</span> <span class="kn">import</span> <span class="n">sleep</span>

<span class="nd">@monitor_resources</span><span class="p">(</span><span class="s">'system'</span><span class="p">,</span> <span class="n">seconds</span><span class="o">=</span><span class="mi">3</span><span class="p">)</span>
<span class="k">def</span> <span class="nf">myfunc</span><span class="p">():</span>
    <span class="n">long_list</span> <span class="o">=</span> <span class="p">[]</span>
    <span class="k">for</span> <span class="n">i</span> <span class="ow">in</span> <span class="nb">range</span><span class="p">(</span><span class="mi">100</span><span class="p">):</span>
        <span class="n">long_list</span> <span class="o">=</span> <span class="n">long_list</span> <span class="o">+</span> <span class="p">(</span><span class="n">i</span><span class="o">*</span><span class="mi">10</span><span class="p">)</span><span class="o">*</span><span class="p">[</span><span class="s">'pancakes'</span><span class="p">]</span>
        <span class="k">print</span><span class="p">(</span><span class="s">"i is </span><span class="si">%</span><span class="s">s, sleeping 10 seconds"</span> <span class="o">%</span> <span class="n">i</span><span class="p">)</span>
        <span class="n">sleep</span><span class="p">(</span><span class="mi">10</span><span class="p">)</span>

</code></pre></div></div>

<p>You can add this decorator on the fly, and get results written to a watcher
in your $HOME even if it doesn’t exist. For example, I could add
this decorator to a long running job on my cluster, set a reasonable number of
seconds to measure metrics as it runs (the default is 3), and then get my
version controlled, programatically parseable data ready to go! A <a href="https://github.com/vsoch/watchme/blob/master/watchme/tests/test_psutils_decorator.py">dummy example</a> is provided here to get you started, and meaty example, discussed next,
<a href="https://github.com/vsoch/watchme-sklearn">is also provided</a>.</p>

<h2 id="watchme-sklearn">Watchme Sklearn</h2>

<p>I decided to start with nice tutorial <a href="https://scikit-learn.org/stable/auto_examples/manifold/plot_lle_digits.html#sphx-glr-auto-examples-manifold-plot-lle-digits-py">from here</a> that goes over some classifiers for sklearn.  My goal would be to create
a monitor (a function decorator) for each one. Don’t forget to import the decorator!</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kn">from</span> <span class="nn">watchme.watchers.psutils.decorators</span> <span class="kn">import</span> <span class="n">monitor_resources</span>
</code></pre></div></div>

<p>And here is how I went about doing this.</p>

<h3 id="1-create-function-wrappers">1. Create Function Wrappers</h3>

<p>I decided to plop each training into it’s own function, so a function might look like this:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="c"># ----------------------------------------------------------------------</span>
<span class="c"># MDS  embedding of the digits dataset</span>

<span class="nd">@monitor_resources</span><span class="p">(</span><span class="s">'watchme-sklearn'</span><span class="p">,</span> <span class="n">seconds</span><span class="o">=</span><span class="mf">0.25</span><span class="p">)</span>
<span class="k">def</span> <span class="nf">mds_embedding</span><span class="p">():</span>

    <span class="k">print</span><span class="p">(</span><span class="s">"Computing MDS embedding"</span><span class="p">)</span>
    <span class="n">clf</span> <span class="o">=</span> <span class="n">manifold</span><span class="o">.</span><span class="n">MDS</span><span class="p">(</span><span class="n">n_components</span><span class="o">=</span><span class="mi">2</span><span class="p">,</span> <span class="n">n_init</span><span class="o">=</span><span class="mi">1</span><span class="p">,</span> <span class="n">max_iter</span><span class="o">=</span><span class="mi">100</span><span class="p">)</span>
    <span class="n">t0</span> <span class="o">=</span> <span class="n">time</span><span class="p">()</span>
    <span class="n">X_mds</span> <span class="o">=</span> <span class="n">clf</span><span class="o">.</span><span class="n">fit_transform</span><span class="p">(</span><span class="n">X</span><span class="p">)</span>
    <span class="k">print</span><span class="p">(</span><span class="s">"Done. Stress: </span><span class="si">%</span><span class="s">f"</span> <span class="o">%</span> <span class="n">clf</span><span class="o">.</span><span class="n">stress_</span><span class="p">)</span>
    <span class="n">plot_embedding</span><span class="p">(</span><span class="n">X_mds</span><span class="p">,</span>
                   <span class="s">"MDS embedding of the digits (time </span><span class="si">%.2</span><span class="s">fs)"</span> <span class="o">%</span>
                   <span class="p">(</span><span class="n">time</span><span class="p">()</span> <span class="o">-</span> <span class="n">t0</span><span class="p">))</span>

</code></pre></div></div>

<p>Let’s talk about the decorator. The first argument “watchme-sklearn” is the watcher name.
This watcher doesn’t have to exist on my computer. If it doesn’t it will be generated.
The second keyword argument, seconds, indicates how often I want to collect metrics.
Since these functions are really fast, I chose every quarter second. The default would
be 3 seconds. This is just one of the functions - you can see all of the functions <a href="https://github.com/vsoch/watchme-sklearn/blob/master/plot_lle_digits.py">here</a>.</p>

<h3 id="2-prepare-to-run">2. Prepare to Run!</h3>

<p>Then in this simple script, I could basically run all of the various plotting functions
when the script was invoked. See that the function above is called <code class="highlighter-rouge">mds_embedding</code>?
It’s one of many in the list here:</p>

<div class="language-python highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="c"># ensure the function runs when the file is called</span>
<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="s">'__main__'</span><span class="p">:</span>
    <span class="n">plot_digits</span><span class="p">()</span>
    <span class="n">random_2d_projection</span><span class="p">()</span>
    <span class="n">pca_projection</span><span class="p">()</span>
    <span class="n">lda_projection</span><span class="p">()</span>
    <span class="n">isomap_projection</span><span class="p">()</span>
    <span class="n">lle_embedding</span><span class="p">()</span>
    <span class="n">modified_lle_embedding</span><span class="p">()</span>
    <span class="n">hessian_lle_embedding</span><span class="p">()</span>
    <span class="n">ltsa_embedding</span><span class="p">()</span>
    <span class="n">mds_embedding</span><span class="p">()</span>
    <span class="n">spectral_embedding</span><span class="p">()</span>
    <span class="n">tsne_embedding</span><span class="p">()</span>
    <span class="n">plt</span><span class="o">.</span><span class="n">show</span><span class="p">()</span>

</code></pre></div></div>

<h3 id="3-run-away-merrill">3. Run away, Merrill</h3>

<p>For the above, I decided to make life easier and build a container. Building and
then running this Singularity <a href="https://github.com/vsoch/watchme-sklearn/blob/master/Singularity">recipe</a> looked like this:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nb">sudo </span>singularity build watchme-sklearn.sif Singularity
singularity run watchme-sklearn.sif

Adding watcher /home/vanessa/.watchme/watchme-sklearn...
Generating watcher config /home/vanessa/.watchme/watchme-sklearn/watchme.cfg

<span class="o">=============================================================================</span>
Manifold learning on handwritten digits: Locally Linear Embedding, Isomap...
<span class="o">=============================================================================</span>

An illustration of various embeddings on the digits dataset.

The RandomTreesEmbedding, from the :mod:<span class="sb">`</span>sklearn.ensemble<span class="sb">`</span> module, is not
technically a manifold embedding method, as it learn a high-dimensional
representation on which we apply a dimensionality reduction method.
However, it is often useful to cast a dataset into a representation <span class="k">in
</span>which the classes are linearly-separable.

t-SNE will be initialized with the embedding that is generated by PCA <span class="k">in
</span>this example, which is not the default setting. It ensures global stability
of the embedding, i.e., the embedding does not depend on random
initialization.

Linear Discriminant Analysis, from the :mod:<span class="sb">`</span>sklearn.discriminant_analysis<span class="sb">`</span>
module, and Neighborhood Components Analysis, from the :mod:<span class="sb">`</span>sklearn.neighbors<span class="sb">`</span>
module, are supervised dimensionality reduction method, i.e. they make use of
the provided labels, contrary to other methods.

Computing random projection
Computing PCA projection
Computing Linear Discriminant Analysis projection
Computing Isomap projection
Done.
Computing LLE embedding
Done. Reconstruction error: 1.63546e-06
Computing modified LLE embedding
Done. Reconstruction error: 0.360659
Computing Hessian LLE embedding
Done. Reconstruction error: 0.212804
Computing LTSA embedding
Done. Reconstruction error: 0.212804
Computing MDS embedding
Done. Stress: 157308701.864713
Computing Spectral embedding
Computing t-SNE embedding

</code></pre></div></div>

<p>Ta da! Done.</p>

<h3 id="3-oggle-at-results">3. Oggle at Results</h3>

<p>Here is a glimpse at what was created in my watchme home. Each function gets
a decorator folder, and within each folder is a result.json file and a TIMESTAMP.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nv">$ </span>tree
<span class="nb">.</span>
├── decorator-psutils-hessian_lle_embedding
│   ├── result.json
│   └── TIMESTAMP
├── decorator-psutils-isomap_projection
│   ├── result.json
│   └── TIMESTAMP
...
├── decorator-psutils-tsne_embedding
│   ├── result.json
│   └── TIMESTAMP
└── watchme.cfg

</code></pre></div></div>

<p>Let’s step back and remind ourselves how watchme stores its data. It’s going
to use the .git repository to store each data entry, where one entry might 
correspond with one function run. We would then use “watchme export” to
generate a json export of this temporal data. For example, Here is how I would export
data for just one of the decorators result files:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>watchme <span class="nb">export </span>watchme-sklearn decorator-psutils-tsne_embedding result.json <span class="nt">--json</span>
</code></pre></div></div>

<p>And this is a subset of what gets splot on my screen, or directly to a file with the <code class="highlighter-rouge">--out</code>
parameter:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="o">{</span>
    <span class="s2">"commits"</span>: <span class="o">[</span>
        <span class="s2">"72d6a9d1fc4b574e4d4063324b8a9dcb19b1b22e"</span>
    <span class="o">]</span>,
    <span class="s2">"dates"</span>: <span class="o">[</span>
        <span class="s2">"2019-05-12 16:24:03 -0400"</span>
    <span class="o">]</span>,
    <span class="s2">"content"</span>: <span class="o">[</span>
        <span class="o">{</span>
            <span class="s2">"create_time"</span>: 1557692636.69,
            <span class="s2">"cmdline"</span>: <span class="o">[</span>
                <span class="s2">"/opt/conda/bin/python"</span>,
                <span class="s2">"/plot_lle_digits.py"</span>
            <span class="o">]</span>,
            <span class="s2">"LABEL"</span>: <span class="s2">"singularity-container"</span>,
            <span class="s2">"SECONDS"</span>: <span class="s2">"0.25"</span>
        ...
        <span class="o">}</span>,
    ...

</code></pre></div></div>

<p>As with all watchme exports, since we are using git as a temporal database,
we get a json structure with commits, dates, and content. The function
was monitored multiple times, and each timepoint is an entry in the “content”
list, all stored under one commit.  Also notice that the interval (SECONDS) is a variable in the result,
along with a custom label “LABEL.”</p>

<blockquote>
  <p>What is a custom label?</p>
</blockquote>

<p>To allow the user flexibility in adding metadata to the result, any <code class="highlighter-rouge">WATCHMEENV_*</code>
prefixed environment variable is automatically added. For the variable above, I
exported the following in the Singularity container in the <a href="https://github.com/vsoch/watchme-sklearn/blob/master/Singularity#L15">environment
section</a>.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nv">WATCHMEENV_LABEL</span><span class="o">=</span>singularity-container
<span class="nb">export </span>WATCHMEENV_LABEL

</code></pre></div></div>

<p>You can see all of the exports in completion <a href="https://github.com/vsoch/watchme-sklearn/tree/master/data">here</a> -
I put them in a data folder in the respository. Here is a programmatic way that I have used to
 export all results to a “data” folder in the repository:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
mkdir <span class="nt">-p</span> data
<span class="k">for </span>folder <span class="k">in</span> <span class="k">$(</span>find <span class="nb">.</span> <span class="nt">-maxdepth</span> 1 <span class="nt">-type</span> d <span class="nt">-name</span> <span class="s1">'decorator*'</span> <span class="nt">-print</span><span class="k">)</span><span class="p">;</span> <span class="k">do
    </span><span class="nv">folder</span><span class="o">=</span><span class="s2">"</span><span class="k">${</span><span class="nv">folder</span><span class="p">//.\/</span><span class="k">}</span><span class="s2">"</span>
    watchme <span class="nb">export </span>watchme-sklearn <span class="nv">$folder</span> <span class="nt">--out</span> data/<span class="nv">$folder</span>.json result.json <span class="nt">--json</span> <span class="nt">--force</span>
<span class="k">done</span>

</code></pre></div></div>

<p>After I generated this folder, I pushed everything up to GitHub. WatchMe handles
adding the result files, so I just needed to commit the exports that I created.</p>

<h3 id="4-run-it-yourself">4. Run it Yourself!</h3>

<p>Here is a final example for how easy it is to share your watchme
decorated functions with others. I built this same container on <a href="https://www.singularity-hub.org/collections/2956">Singularity Hub</a>
so you can pull it, and run it. And just for kicks and giggles, we will add an 
extra variable for the Singularity container to find.</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
singularity pull shub://vsoch/watchme-sklearn
<span class="nb">export </span><span class="nv">SINGULARITYENV_WATCHMEENV_avocados</span><span class="o">=</span>aregreat
./watchme-sklearn_latest.sif

</code></pre></div></div>

<p>And seriously that’s it - go to your $HOME/.watchme/watchme-sklearn folder
to inspect the results!</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
<span class="nb">cd</span> <span class="nv">$HOME</span>/.watchme/watchme-sklearn
watchme <span class="nb">export </span>watchme-sklearn decorator-psutils-tsne_embedding result.json <span class="nt">--json</span>

</code></pre></div></div>

<p>And yes, you will even see our avocados variable!</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
watchme <span class="nb">export </span>watchme-sklearn decorator-psutils-tsne_embedding result.json <span class="nt">--json</span> | <span class="nb">grep </span>avocados
            <span class="s2">"avocados"</span>: <span class="s2">"aregreat"</span>,
            ...

</code></pre></div></div>

<p>Check out the git log to see everything recorded for you:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
git log
commit 6f10453a429ab3e2ad835520443bf127c466ac40 <span class="o">(</span>HEAD -&gt; master<span class="o">)</span>
Author: Vanessa Sochat &lt;vsochat@stanford.edu&gt;
Date:   Sun May 12 18:02:08 2019 <span class="nt">-0400</span>

    watchme watchme-sklearn ADD results decorator-psutils-tsne_embedding

commit b53b4ab7896b1668fa3562334db633431170bb6f
Author: Vanessa Sochat &lt;vsochat@stanford.edu&gt;
Date:   Sun May 12 18:02:08 2019 <span class="nt">-0400</span>

    watchme watchme-sklearn ADD results decorator-psutils-plot_embedding

commit 9aa96290f4bd1b45e33f54a050899f1adaf308f1
Author: Vanessa Sochat &lt;vsochat@stanford.edu&gt;
Date:   Sun May 12 18:02:02 2019 <span class="nt">-0400</span>

    watchme watchme-sklearn ADD results decorator-psutils-spectral_embedding

commit e0ad2d079a74794e0ccb2c14e9cd368356074774
Author: Vanessa Sochat &lt;vsochat@stanford.edu&gt;
Date:   Sun May 12 18:02:02 2019 <span class="nt">-0400</span>

    watchme watchme-sklearn ADD results decorator-psutils-plot_embedding

commit efbccc4bbe58d81597588cc268f91e5809986122
Author: Vanessa Sochat &lt;vsochat@stanford.edu&gt;
Date:   Sun May 12 18:02:01 2019 <span class="nt">-0400</span>
...

</code></pre></div></div>

<p>And at this point you would add a README, and then just create a GitHub repository
to push to, and push. Is that cool or what? Obviously, we’d want to use the decorators for more interesting
(longer) tasks, possibly on HPC. If you have something in mind, please reach
out and we can put together an example to run!</p>