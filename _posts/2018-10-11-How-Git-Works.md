---
layout:     post
title:      How Git Works?
subtitle:   Understanding Git
date:       2018-10-11
author:     Mr.Humorous 🥘
header-img: img/git/header.jpg
catalog: true
tags:
    - Git
---

<div class="asset-content entry-content" id="main-content">
<h2>一、初始化</h2>

<p>首先，让我们创建一个项目目录，并进入该目录。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ mkdir git<span class="token operator">-</span>demo<span class="token operator">-</span>project
$ cd git<span class="token operator">-</span>demo<span class="token operator">-</span>project
</code></pre></blockquote>

<p>我们打算对该项目进行版本管理，第一件事就是使用<code>git init</code>命令，进行初始化。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ git init
</code></pre></blockquote>

<p><code>git init</code>命令只做一件事，就是在项目根目录下创建一个<code>.git</code>子目录，用来保存版本信息。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ ls <span class="token punctuation">.</span>git

branches<span class="token operator">/</span>
config
description
HEAD
hooks<span class="token operator">/</span>
info<span class="token operator">/</span>
objects<span class="token operator">/</span>
refs<span class="token operator">/</span>
</code></pre></blockquote>

<h2>二、保存对象</h2>

<p>接下来，新建一个空文件<code>test.txt</code>。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ touch test<span class="token punctuation">.</span>txt
</code></pre></blockquote>

<p>然后，把这个文件加入 Git 仓库，也就是为<code>test.txt</code>的当前内容创建一个副本。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ git hash<span class="token operator">-</span>object <span class="token operator">-</span>w test<span class="token punctuation">.</span>txt

e69de29bb2d1d6434b8b29ae775ad8c2e48c5391
</code></pre></blockquote>

<p>上面代码中，<code>git hash-object</code>命令把<code>test.txt</code>的当前内容压缩成二进制文件，存入 Git。压缩后的二进制文件，称为一个 Git 对象，保存在<code>.git/objects</code>目录。</p>

<p>这个命令还会计算当前内容的 SHA1 哈希值（长度40的字符串），作为该对象的文件名。下面看一下这个新生成的 Git 对象文件。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ ls <span class="token operator">-</span>R <span class="token punctuation">.</span>git<span class="token operator">/</span>objects

<span class="token punctuation">.</span>git<span class="token operator">/</span>objects<span class="token operator">/</span>e6<span class="token punctuation">:</span>
9de29bb2d1d6434b8b29ae775ad8c2e48c5391
</code></pre></blockquote>

<p>上面代码可以看到，<code>.git/objects</code>下面多了一个子目录，目录名是哈希值的前2个字符，该子目录下面有一个文件，文件名是哈希值的后38个字符。</p>

<p>再看一下这个文件的内容。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ cat <span class="token punctuation">.</span>git<span class="token operator">/</span>objects<span class="token operator">/</span>e6<span class="token operator">/</span>9de29bb2d1d6434b8b29ae775ad8c2e48c5391
</code></pre></blockquote>

<p>上面代码输出的文件内容，都是一些二进制字符。你可能会问，<code>test.txt</code>是一个空文件，为什么会有内容？这是因为二进制对象里面还保存一些元数据。</p>

<p>如果想看该文件原始的文本内容，要用<code>git cat-file</code>命令。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ git cat<span class="token operator">-</span>file <span class="token operator">-</span>p e69de29bb2d1d6434b8b29ae775ad8c2e48c5391
</code></pre></blockquote>

<p>因为原始文件是空文件，所以上面的命令什么也看不到。现在向<code>test.txt</code>写入一些内容。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ <span class="token keyword">echo</span> <span class="token string">'hello world'</span> <span class="token operator">&gt;</span> test<span class="token punctuation">.</span>txt
</code></pre></blockquote>

<p>因为文件内容已经改变，需要将它再次保存成 Git 对象。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ git hash<span class="token operator">-</span>object <span class="token operator">-</span>w test<span class="token punctuation">.</span>txt

3b18e512dba79e4c8300dd08aeb37f8e728b8dad
</code></pre></blockquote>

<p>上面代码可以看到，随着内容改变，<code>test.txt</code>的哈希值已经变了。同时，新文件<code>.git/objects/3b/18e512dba79e4c8300dd08aeb37f8e728b8dad</code>也已经生成了。现在可以看到文件内容了。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ git cat<span class="token operator">-</span>file <span class="token operator">-</span>p 3b18e512dba79e4c8300dd08aeb37f8e728b8dad

hello world
</code></pre></blockquote>

<h2>三、暂存区</h2>

<p>文件保存成二进制对象以后，还需要通知 Git 哪些文件发生了变动。所有变动的文件，Git 都记录在一个区域，叫做"暂存区"（英文叫做 index 或者 stage）。等到变动告一段落，再统一把暂存区里面的文件写入正式的版本历史。</p>

<p><code>git update-index</code>命令用于在暂存区记录一个发生变动的文件。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ git update<span class="token operator">-</span>index <span class="token operator">--</span>add <span class="token operator">--</span>cacheinfo <span class="token number">100644</span> \
3b18e512dba79e4c8300dd08aeb37f8e728b8dad test<span class="token punctuation">.</span>txt
</code></pre></blockquote>

<p>上面命令向暂存区写入文件名<code>test.txt</code>、二进制对象名（哈希值）和文件权限。</p>

<p><code>git ls-files</code>命令可以显示暂存区当前的内容。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ git ls<span class="token operator">-</span>files <span class="token operator">--</span>stage

<span class="token number">100644</span> 3b18e512dba79e4c8300dd08aeb37f8e728b8dad <span class="token number">0</span>   test<span class="token punctuation">.</span>txt
</code></pre></blockquote>

<p>上面代码表示，暂存区现在只有一个文件<code>test.txt</code>，以及它的二进制对象名和权限。知道了二进制对象名，就可以在<code>.git/objects</code>子目录里面读出这个文件的内容。</p>

<p><code>git status</code>命令会产生更可读的结果。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ git status

要提交的变更：
    新文件：   test<span class="token punctuation">.</span>txt
</code></pre></blockquote>

<p>上面代码表示，暂存区里面只有一个新文件<code>test.txt</code>，等待写入历史。</p>

<h2>四、git add 命令</h2>

<p>上面两步（保存对象和更新暂存区），如果每个文件都做一遍，那是很麻烦的。Git 提供了<code>git add</code>命令简化操作。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ git add <span class="token operator">--</span>all
</code></pre></blockquote>

<p>上面命令相当于，对当前项目所有变动的文件，执行前面的两步操作。</p>

<h2>五、commit 的概念</h2>

<p>暂存区保留本次变动的文件信息，等到修改了差不多了，就要把这些信息写入历史，这就相当于生成了当前项目的一个快照（snapshot）。</p>

<p>项目的历史就是由不同时点的快照构成。Git 可以将项目恢复到任意一个快照。快照在 Git 里面有一个专门名词，叫做 commit，生成快照又称为完成一次提交。</p>

<p>下文所有提到"快照"的地方，指的就是 commit。</p>

<h2>六、完成提交</h2>

<p>首先，设置一下用户名和 Email，保存快照的时候，会记录是谁提交的。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ git config user<span class="token punctuation">.</span>name <span class="token string">"用户名"</span> 
$ git config user<span class="token punctuation">.</span>email <span class="token string">"Email 地址"</span>
</code></pre></blockquote>

<p>接下来，要保存当前的目录结构。前面保存对象的时候，只是保存单个文件，并没有记录文件之间的目录关系（哪个文件在哪里）。</p>

<p><code>git write-tree</code>命令用来将当前的目录结构，生成一个 Git 对象。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ git write<span class="token operator">-</span>tree

c3b8bb102afeca86037d5b5dd89ceeb0090eae9d
</code></pre></blockquote>

<p>上面代码中，目录结构也是作为二进制对象保存的，也保存在<code>.git/objects</code>目录里面，对象名就是哈希值。</p>

<p>让我们看一下这个文件的内容。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ git cat<span class="token operator">-</span>file <span class="token operator">-</span>p c3b8bb102afeca86037d5b5dd89ceeb0090eae9d

<span class="token number">100644</span> blob 3b18e512dba79e4c8300dd08aeb37f8e728b8dad    test<span class="token punctuation">.</span>txt
</code></pre></blockquote>

<p>可以看到，当前的目录里面只有一个<code>test.txt</code>文件。</p>

<p>所谓快照，就是保存当前的目录结构，以及每个文件对应的二进制对象。上一个操作，目录结构已经保存好了，现在需要将这个目录结构与一些元数据一起写入版本历史。</p>

<p><code>git commit-tree</code>命令用于将目录树对象写入版本历史。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ <span class="token keyword">echo</span> <span class="token string">"first commit"</span> <span class="token operator">|</span> git commit<span class="token operator">-</span>tree c3b8bb102afeca86037d5b5dd89ceeb0090eae9d

c9053865e9dff393fd2f7a92a18f9bd7f2caa7fa
</code></pre></blockquote>

<p>上面代码中，提交的时候需要有提交说明，<code>echo "first commit"</code>就是给出提交说明。然后，<code>git commit-tree</code>命令将元数据和目录树，一起生成一个 Git 对象。现在，看一下这个对象的内容。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ git cat<span class="token operator">-</span>file <span class="token operator">-</span>p c9053865e9dff393fd2f7a92a18f9bd7f2caa7fa

tree c3b8bb102afeca86037d5b5dd89ceeb0090eae9d
author ruanyf  <span class="token number">1538889134</span> <span class="token operator">+</span><span class="token number">0800</span>
committer ruanyf  <span class="token number">1538889134</span> <span class="token operator">+</span><span class="token number">0800</span>

first commit
</code></pre></blockquote>

<p>上面代码中，输出结果的第一行是本次快照对应的目录树对象（tree），第二行和第三行是作者和提交人信息，最后是提交说明。</p>

<p><code>git log</code>命令也可以用来查看某个快照信息。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ git log <span class="token operator">--</span>stat c9053865e9dff393fd2f7a92a18f9bd7f2caa7fa

commit c9053865e9dff393fd2f7a92a18f9bd7f2caa7fa
Author<span class="token punctuation">:</span> ruanyf 
Date<span class="token punctuation">:</span>   Sun Oct <span class="token number">7</span> <span class="token number">13</span><span class="token punctuation">:</span><span class="token number">12</span><span class="token punctuation">:</span><span class="token number">14</span> <span class="token number">2018</span> <span class="token operator">+</span><span class="token number">0800</span>

    first commit

 test<span class="token punctuation">.</span>txt <span class="token operator">|</span> <span class="token number">1</span> <span class="token operator">+</span>
 <span class="token number">1</span> file changed<span class="token punctuation">,</span> <span class="token number">1</span> <span class="token function">insertion<span class="token punctuation">(</span></span><span class="token operator">+</span><span class="token punctuation">)</span>
</code></pre></blockquote>

<h2>七、git commit 命令</h2>

<p>Git 提供了<code>git commit</code>命令，简化提交操作。保存进暂存区以后，只要<code>git commit</code>一个命令，就同时提交目录结构和说明，生成快照。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ git commit <span class="token operator">-</span>m <span class="token string">"first commit"</span>
</code></pre></blockquote>

<p>此外，还有两个命令也很有用。</p>

<p><code>git checkout</code>命令用于切换到某个快照。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ git checkout c9053865e9dff393fd2f7a92a18f9bd7f2caa7fa
</code></pre></blockquote>

<p><code>git show</code>命令用于展示某个快照的所有代码变动。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ git show c9053865e9dff393fd2f7a92a18f9bd7f2caa7fa
</code></pre></blockquote>

<h2>八、branch 的概念</h2>

<p>到了这一步，还没完。如果这时用<code>git log</code>命令查看整个版本历史，你看不到新生成的快照。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ git log
</code></pre></blockquote>

<p>上面命令没有任何输出，这是为什么呢？快照明明已经写入历史了。</p>

<p>原来<code>git log</code>命令只显示当前分支的变动，虽然我们前面已经提交了快照，但是还没有记录这个快照属于哪个分支。</p>

<p>所谓分支（branch）就是指向某个快照的指针，分支名就是指针名。哈希值是无法记忆的，分支使得用户可以为快照起别名。而且，分支会自动更新，如果当前分支有新的快照，指针就会自动指向它。比如，master 分支就是有一个叫做 master 指针，它指向的快照就是 master 分支的当前快照。</p>

<p>用户可以对任意快照新建指针。比如，新建一个 fix-typo 分支，就是创建一个叫做 fix-typo 的指针，指向某个快照。所以，Git 新建分支特别容易，成本极低。</p>

<p>Git 有一个特殊指针<code>HEAD</code>， 总是指向当前分支的最近一次快照。另外，Git 还提供简写方式，<code>HEAD^</code>指向 <code>HEAD</code>的前一个快照（父节点），<code>HEAD~6</code>则是<code>HEAD</code>之前的第6个快照。</p>

<p>每一个分支指针都是一个文本文件，保存在<code>.git/refs/heads/</code>目录，该文件的内容就是它所指向的快照的二进制对象名（哈希值）。</p>

<h2>九、更新分支</h2>

<p>下面演示更新分支是怎么回事。首先，修改一下<code>test.txt</code>。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ <span class="token keyword">echo</span> <span class="token string">"hello world again"</span> <span class="token operator">&gt;</span> test<span class="token punctuation">.</span>txt
</code></pre></blockquote>

<p>然后，保存二进制对象。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ git hash<span class="token operator">-</span>object <span class="token operator">-</span>w test<span class="token punctuation">.</span>txt

c90c5155ccd6661aed956510f5bd57828eec9ddb
</code></pre></blockquote>

<p>接着，将这个对象写入暂存区，并保存目录结构。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ git update<span class="token operator">-</span>index test<span class="token punctuation">.</span>txt
$ git write<span class="token operator">-</span>tree

1552fd52bc14497c11313aa91547255c95728f37
</code></pre></blockquote>

<p>最后，提交目录结构，生成一个快照。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ <span class="token keyword">echo</span> <span class="token string">"second commit"</span> <span class="token operator">|</span> git commit<span class="token operator">-</span>tree 1552fd52bc14497c11313aa91547255c95728f37 <span class="token operator">-</span>p c9053865e9dff393fd2f7a92a18f9bd7f2caa7fa

785f188674ef3c6ddc5b516307884e1d551f53ca
</code></pre></blockquote>

<p>上面代码中，<code>git commit-tree</code>的<code>-p</code>参数用来指定父节点，也就是本次快照所基于的快照。</p>

<p>现在，我们把本次快照的哈希值，写入<code>.git/refs/heads/master</code>文件，这样就使得<code>master</code>指针指向这个快照。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ <span class="token keyword">echo</span> 785f188674ef3c6ddc5b516307884e1d551f53ca <span class="token operator">&gt;</span> <span class="token punctuation">.</span>git<span class="token operator">/</span>refs<span class="token operator">/</span>heads<span class="token operator">/</span>master
</code></pre></blockquote>

<p>现在，<code>git log</code>就可以看到两个快照了。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ git log

commit 785f188674ef3c6ddc5b516307884e1d551f53ca <span class="token punctuation">(</span>HEAD <span class="token operator">-</span><span class="token operator">&gt;</span> master<span class="token punctuation">)</span>
Author<span class="token punctuation">:</span> ruanyf 
Date<span class="token punctuation">:</span>   Sun Oct <span class="token number">7</span> <span class="token number">13</span><span class="token punctuation">:</span><span class="token number">38</span><span class="token punctuation">:</span><span class="token number">00</span> <span class="token number">2018</span> <span class="token operator">+</span><span class="token number">0800</span>

    second commit

commit c9053865e9dff393fd2f7a92a18f9bd7f2caa7fa
Author<span class="token punctuation">:</span> ruanyf 
Date<span class="token punctuation">:</span>   Sun Oct <span class="token number">7</span> <span class="token number">13</span><span class="token punctuation">:</span><span class="token number">12</span><span class="token punctuation">:</span><span class="token number">14</span> <span class="token number">2018</span> <span class="token operator">+</span><span class="token number">0800</span>

    first commit
</code></pre></blockquote>

<p><code>git log</code>的运行过程是这样的：</p>

<blockquote>
  <ol start="1">
<li>查找<code>HEAD</code>指针对应的分支，本例是<code>master</code></li>
<li>找到<code>master</code>指针指向的快照，本例是<code>785f188674ef3c6ddc5b516307884e1d551f53ca</code></li>
<li>找到父节点（前一个快照）<code>c9053865e9dff393fd2f7a92a18f9bd7f2caa7fa</code></li>
<li>以此类推，显示当前分支的所有快照</li>
</ol>
</blockquote>

<p>最后，补充一点。前面说过，分支指针是动态的。原因在于，下面三个命令会自动改写分支指针。</p>

<blockquote>
  <ul>
<li><code>git commit</code>：当前分支指针移向新创建的快照。</li>
<li><code>git pull</code>：当前分支与远程分支合并后，指针指向新创建的快照。</li>
<li><code>git reset [commit_sha]</code>：当前分支指针重置为指定快照。</li>
</ul>
</blockquote>
