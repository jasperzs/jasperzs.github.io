---
layout:     post
title:      Wildcards In Bash
subtitle:   Understanding Wildcards Usage
date:       2018-10-13
author:     Mr.Humorous 🥘
header-img: img/shell/header.png
catalog: true
tags:
    - Shell
---

<div class="asset-content entry-content" id="main-content">

<p>一次性操作多个文件时，命令行提供通配符（wildcards），用一种很短的文本模式（通常只有一个字符），简洁地代表一组路径。</p>

<p><img src="https://www.wangbase.com/blogimg/asset/201809/bg2018092001.jpg" alt="" title=""></p>

<p>通配符又叫做 globbing patterns。因为 Unix 早期有一个<code>/etc/glob</code>文件保存通配符模板，后来 Bash 内置了这个功能，但是这个名字被保留了下来。</p>

<p>通配符早于正则表达式出现，可以看作是原始的正则表达式。它的功能没有正则那么强大灵活，但是胜在简单和方便。</p>

<p>本文介绍 Bash 的各种通配符。</p>

<h2>一、? 字符</h2>

<p><code>?</code>字符代表单个字符。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
<span class="token comment" spellcheck="true"># 存在文件 a.txt 和 b.txt
</span>$ ls <span class="token operator">?</span><span class="token punctuation">.</span>txt
a<span class="token punctuation">.</span>txt b<span class="token punctuation">.</span>txt
</code></pre></blockquote>

<p>上面命令中，<code>?</code>表示单个字符，所以会同时匹配<code>a.txt</code>和<code>b.txt</code>。</p>

<p>如果匹配多个字符，就需要多个<code>?</code>连用。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
<span class="token comment" spellcheck="true"># 存在文件 a.txt、b.txt 和 ab.txt
</span>$ ls <span class="token operator">?</span><span class="token operator">?</span><span class="token punctuation">.</span>txt
ab<span class="token punctuation">.</span>txt
</code></pre></blockquote>

<p>上面命令中，<code>??</code>匹配了两个字符。</p>

<p>注意，<code>?</code>不能匹配空字符。也就是说，它占据的位置必须有字符存在。</p>

<h2>二、* 字符</h2>

<p><code>*</code>代表任意数量的字符。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
<span class="token comment" spellcheck="true"># 存在文件 a.txt、b.txt 和 ab.txt
</span>$ ls <span class="token operator">*</span><span class="token punctuation">.</span>txt
a<span class="token punctuation">.</span>txt b<span class="token punctuation">.</span>txt ab<span class="token punctuation">.</span>txt

<span class="token comment" spellcheck="true"># 输出所有文件
</span>$ ls <span class="token operator">*</span>
</code></pre></blockquote>

<p>上面代码中，<code>*</code>匹配任意长度的字符。</p>

<p><code>*</code>可以匹配空字符。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
<span class="token comment" spellcheck="true"># 存在文件 a.txt、b.txt 和 ab.txt
</span>$ ls a<span class="token operator">*</span><span class="token punctuation">.</span>txt
a<span class="token punctuation">.</span>txt ab<span class="token punctuation">.</span>txt
</code></pre></blockquote>

<h2>三、[...] 模式</h2>

<p><code>[...]</code>匹配方括号之中的任意一个字符，比如<code>[aeiou]</code>可以匹配五个元音字母。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
<span class="token comment" spellcheck="true"># 存在文件 a.txt 和 b.txt
</span>$ ls <span class="token punctuation">[</span>ab<span class="token punctuation">]</span><span class="token punctuation">.</span>txt
a<span class="token punctuation">.</span>txt b<span class="token punctuation">.</span>txt

$ ls <span class="token operator">*</span><span class="token punctuation">[</span>ab<span class="token punctuation">]</span><span class="token punctuation">.</span>txt
ab<span class="token punctuation">.</span>txt a<span class="token punctuation">.</span>txt b<span class="token punctuation">.</span>txt
</code></pre></blockquote>

<p><code>[start-end]</code>表示一个连续的范围。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
<span class="token comment" spellcheck="true"># 存在文件 a.txt、b.txt 和 c.txt
</span>$ ls <span class="token punctuation">[</span>a<span class="token operator">-</span>c<span class="token punctuation">]</span><span class="token punctuation">.</span>txt
a<span class="token punctuation">.</span>txt b<span class="token punctuation">.</span>txt c<span class="token punctuation">.</span>txt

<span class="token comment" spellcheck="true"># 存在文件 report1.txt、report2.txt 和 report3.txt
</span>$ ls report<span class="token punctuation">[</span><span class="token number">0</span><span class="token operator">-</span><span class="token number">9</span><span class="token punctuation">]</span><span class="token punctuation">.</span>txt
report1<span class="token punctuation">.</span>txt report2<span class="token punctuation">.</span>txt report3<span class="token punctuation">.</span>txt
</code></pre></blockquote>

<h2>四、<code>[^...]</code> 和 <code>[!...]</code></h2>

<p><code>[^...]</code>和<code>[!...]</code>表示匹配不在方括号里面的字符（不包括空字符）。这两种写法是等价的。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
<span class="token comment" spellcheck="true"># 存在文件 a.txt、b.txt 和 c.txt
</span>$ ls <span class="token punctuation">[</span><span class="token operator">^</span>a<span class="token punctuation">]</span><span class="token punctuation">.</span>txt
b<span class="token punctuation">.</span>txt c<span class="token punctuation">.</span>txt
</code></pre></blockquote>

<p>这种模式下也可以使用连续范围的写法<code>[!start-end]</code>。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ <span class="token keyword">echo</span> report<span class="token punctuation">[</span><span class="token operator">!</span><span class="token number">1</span><span class="token operator">-</span><span class="token number">3</span><span class="token punctuation">]</span><span class="token punctuation">.</span>txt
report4<span class="token punctuation">.</span>txt report5<span class="token punctuation">.</span>txt
</code></pre></blockquote>

<p>上面代码中，<code>[!1-3]</code>表示排除1、2和3。</p>

<h2>五、{...} 模式</h2>

<p><code>{...}</code> 表示匹配大括号里面的所有模式，模式之间使用逗号分隔。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ <span class="token keyword">echo</span> d<span class="token punctuation">{</span>a<span class="token punctuation">,</span>e<span class="token punctuation">,</span>i<span class="token punctuation">,</span>u<span class="token punctuation">,</span>o<span class="token punctuation">}</span>g
dag deg dig dug dog
</code></pre></blockquote>

<p>它可以用于多字符的模式。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ <span class="token keyword">echo</span> <span class="token punctuation">{</span>cat<span class="token punctuation">,</span>dog<span class="token punctuation">}</span>
cat dog
</code></pre></blockquote>

<p><code>{...}</code>与<code>[...]</code>有一个很重要的区别。如果匹配的文件不存在，<code>[...]</code>会失去模式的功能，变成一个单纯的字符串，而<code>{...}</code>依然可以展开。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
<span class="token comment" spellcheck="true"># 不存在 a.txt 和 b.txt
</span>$ ls <span class="token punctuation">[</span>ab<span class="token punctuation">]</span><span class="token punctuation">.</span>txt
ls<span class="token punctuation">:</span> <span class="token punctuation">[</span>ab<span class="token punctuation">]</span><span class="token punctuation">.</span>txt<span class="token punctuation">:</span> No such file or directory

$ ls <span class="token punctuation">{</span>a<span class="token punctuation">,</span>b<span class="token punctuation">}</span><span class="token punctuation">.</span>txt
ls<span class="token punctuation">:</span> a<span class="token punctuation">.</span>txt<span class="token punctuation">:</span> No such file or directory
ls<span class="token punctuation">:</span> b<span class="token punctuation">.</span>txt<span class="token punctuation">:</span> No such file or directory
</code></pre></blockquote>

<p>上面代码中，如果不存在<code>a.txt</code>和<code>b.txt</code>，那么<code>[ab].txt</code>就会变成一个普通的文件名，而<code>{a,b}.txt</code>可以照样展开。</p>

<p>大括号可以嵌套。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ <span class="token keyword">echo</span> <span class="token punctuation">{</span>j<span class="token punctuation">{</span>p<span class="token punctuation">,</span>pe<span class="token punctuation">}</span>g<span class="token punctuation">,</span>png<span class="token punctuation">}</span>
jpg jpeg png
</code></pre></blockquote>

<p>大括号也可以与其他模式联用。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ <span class="token keyword">echo</span> <span class="token punctuation">{</span>cat<span class="token punctuation">,</span>d<span class="token operator">*</span><span class="token punctuation">}</span>
cat dawg dg dig dog doug dug
</code></pre></blockquote>

<p>上面代码中，会先进行大括号扩展，然后进行<code>*</code>扩展。</p>

<h2>六、{start..end} 模式</h2>

<p><code>{start..end}</code>会匹配连续范围的字符。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ <span class="token keyword">echo</span> d<span class="token punctuation">{</span>a<span class="token punctuation">.</span><span class="token punctuation">.</span>d<span class="token punctuation">}</span>g
dag dbg dcg ddg

$ <span class="token keyword">echo</span> <span class="token punctuation">{</span><span class="token number">11</span><span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token number">15</span><span class="token punctuation">}</span>
<span class="token number">11</span> <span class="token number">12</span> <span class="token number">13</span> <span class="token number">14</span> <span class="token number">15</span>
</code></pre></blockquote>

<p>如果遇到无法解释的扩展，模式会原样输出。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ <span class="token keyword">echo</span> <span class="token punctuation">{</span>a1<span class="token punctuation">.</span><span class="token punctuation">.</span>3c<span class="token punctuation">}</span>
<span class="token punctuation">{</span>a1<span class="token punctuation">.</span><span class="token punctuation">.</span>3c<span class="token punctuation">}</span>
</code></pre></blockquote>

<p>这种模式与逗号联用，可以写出复杂的模式。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ <span class="token keyword">echo</span> <span class="token punctuation">.</span><span class="token punctuation">{</span>mp<span class="token punctuation">{</span><span class="token number">3</span><span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token number">4</span><span class="token punctuation">}</span><span class="token punctuation">,</span>m4<span class="token punctuation">{</span>a<span class="token punctuation">,</span>b<span class="token punctuation">,</span>p<span class="token punctuation">,</span>v<span class="token punctuation">}</span><span class="token punctuation">}</span>
<span class="token punctuation">.</span>mp3 <span class="token punctuation">.</span>mp4 <span class="token punctuation">.</span>m4a <span class="token punctuation">.</span>m4b <span class="token punctuation">.</span>m4p <span class="token punctuation">.</span>m4v
</code></pre></blockquote>

<h2>七、注意点</h2>

<p>通配符有一些使用注意点，不可不知。</p>

<p><strong>（1）通配符是先解释，再执行。</strong></p>

<p>Bash 接收到命令以后，发现里面有通配符，会进行通配符扩展，然后再执行命令。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ ls a<span class="token operator">*</span><span class="token punctuation">.</span>txt
ab<span class="token punctuation">.</span>txt
</code></pre></blockquote>

<p>上面命令的执行过程是，Bash 先将<code>a*.txt</code>扩展成<code>ab.txt</code>，然后再执行<code>ls ab.txt</code>。</p>

<p><strong>（2）通配符不匹配，会原样输出。</strong></p>

<p>Bash 扩展通配符的时候，发现不存在匹配的文件，会将通配符原样输出。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
<span class="token comment" spellcheck="true"># 不存在 r 开头的文件名
</span>$ <span class="token keyword">echo</span> r<span class="token operator">*</span>
r<span class="token operator">*</span>
</code></pre></blockquote>

<p>上面代码中，由于不存在<code>r</code>开头的文件名，<code>r*</code>会原样输出。</p>

<p>下面是另一个例子。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ ls <span class="token operator">*</span><span class="token punctuation">.</span>csv
ls<span class="token punctuation">:</span> <span class="token operator">*</span><span class="token punctuation">.</span>csv<span class="token punctuation">:</span> No such file or directory
</code></pre></blockquote> 

<p>另外，前面已经说过，这条规则对<code>{...}</code>不适用</p>

<p><strong>（3）只适用于单层路径。</strong></p>

<p>上面所有通配符只匹配单层路径，不能跨目录匹配，即无法匹配子目录里面的文件。或者说，<code>?</code>或<code>*</code>这样的通配符，不能匹配路径分隔符（<code>/</code>）。</p>

<p>如果要匹配子目录里面的文件，可以写成下面这样。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ ls <span class="token operator">*</span><span class="token operator">/</span><span class="token operator">*</span><span class="token punctuation">.</span>txt
</code></pre></blockquote>

<p><strong>（4）可用于文件名。</strong></p>

<p>Bash 允许文件名使用通配符。这时，引用文件名的时候，需要把文件名放在单引号里面。</p>

<blockquote><pre class=" language-bash"><code class=" language-bash">
$ touch <span class="token string">'fo*'</span>
$ ls
fo<span class="token operator">*</span>
</code></pre></blockquote>

<p>上面代码创建了一个<code>fo*</code>文件，这时<code>*</code>就是文件名的一部分。</p>

<h2>八、参考链接</h2>

<ul>
<li><a href="https://medium.com/@leedowthwaite/why-most-people-only-think-they-understand-wildcards-63bb9c2024ab" target="_blank">Think You Understand Wildcards? Think Again</a></li>
<li><a href="https://appcodelabs.com/advanced-wildcard-patterns-most-people-dont-know" target="_blank">Advanced Wildcard Patterns Most People Don't Know</a></li>
</ul>
