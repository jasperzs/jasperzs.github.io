---
layout:     post
title:      Restful API 实践讲解
subtitle:   总结Restful的设计细节和介绍如何设计出易于理解和使用的API
date:       2018-10-29
author:     Mr.Humorous 🥘
header-img: img/restfulAPI/header.jpg
catalog: true
tags:
    - Restful API
---

<div class="asset-content entry-content" id="main-content">
<h2>一、URL 设计</h2>

<h3>1.1 动词 + 宾语</h3>

<p>RESTful 的核心思想就是，客户端发出的数据操作指令都是"动词 + 宾语"的结构。比如，<code>GET /articles</code>这个命令，<code>GET</code>是动词，<code>/articles</code>是宾语。</p>

<p>动词通常就是五种 HTTP 方法，对应 CRUD 操作。</p>

<blockquote>
  <ul>
<li>GET：读取（Read）</li>
<li>POST：新建（Create）</li>
<li>PUT：更新（Update）</li>
<li>PATCH：更新（Update），通常是部分更新</li>
<li>DELETE：删除（Delete）</li>
</ul>
</blockquote>

<p>根据 HTTP 规范，动词一律大写。</p>

<h3>1.2 动词的覆盖</h3>

<p>有些客户端只能使用<code>GET</code>和<code>POST</code>这两种方法。服务器必须接受<code>POST</code>模拟其他三个方法（<code>PUT</code>、<code>PATCH</code>、<code>DELETE</code>）。</p>

<p>这时，客户端发出的 HTTP 请求，要加上<code>X-HTTP-Method-Override</code>属性，告诉服务器应该使用哪一个动词，覆盖<code>POST</code>方法。</p>

<blockquote><pre class=" language-http"><code class=" language-http">
POST /api/Person/4 HTTP/1.1
<span class="token keyword">X-HTTP-Method-Override:</span> PUT
</code></pre></blockquote>

<p>上面代码中，<code>X-HTTP-Method-Override</code>指定本次请求的方法是<code>PUT</code>，而不是<code>POST</code>。</p>

<h3>1.3 宾语必须是名词</h3>

<p>宾语就是 API 的 URL，是 HTTP 动词作用的对象。它应该是名词，不能是动词。比如，<code>/articles</code>这个 URL 就是正确的，而下面的 URL 不是名词，所以都是错误的。</p>

<blockquote>
  <ul>
<li>/getAllCars</li>
<li>/createNewCar</li>
<li>/deleteAllRedCars</li>
</ul>
</blockquote>

<h3>1.4 复数 URL</h3>

<p>既然 URL 是名词，那么应该使用复数，还是单数？</p>

<p>这没有统一的规定，但是常见的操作是读取一个集合，比如<code>GET /articles</code>（读取所有文章），这里明显应该是复数。</p>

<p>为了统一起见，建议都使用复数 URL，比如<code>GET /articles/2</code>要好于<code>GET /article/2</code>。</p>

<h3>1.5 避免多级 URL</h3>

<p>常见的情况是，资源需要多级分类，因此很容易写出多级的 URL，比如获取某个作者的某一类文章。</p>

<blockquote><pre class=" language-http"><code class=" language-http">
GET /authors/12/categories/2
</code></pre></blockquote>

<p>这种 URL 不利于扩展，语义也不明确，往往要想一会，才能明白含义。</p>

<p>更好的做法是，除了第一级，其他级别都用查询字符串表达。</p>

<blockquote><pre class=" language-http"><code class=" language-http">
GET /authors/12?categories=2
</code></pre></blockquote>

<p>下面是另一个例子，查询已发布的文章。你可能会设计成下面的 URL。</p>

<blockquote><pre class=" language-http"><code class=" language-http">
GET /articles/published
</code></pre></blockquote>

<p>查询字符串的写法明显更好。</p>

<blockquote><pre class=" language-http"><code class=" language-http">
GET /articles?published=true
</code></pre></blockquote>

<h2>二、状态码</h2>

<h3>2.1 状态码必须精确</h3>

<p>客户端的每一次请求，服务器都必须给出回应。回应包括 HTTP 状态码和数据两部分。</p>

<p>HTTP 状态码就是一个三位数，分成五个类别。</p>

<blockquote>
  <ul>
<li><code>1xx</code>：相关信息</li>
<li><code>2xx</code>：操作成功</li>
<li><code>3xx</code>：重定向</li>
<li><code>4xx</code>：客户端错误</li>
<li><code>5xx</code>：服务器错误</li>
</ul>
</blockquote>

<p>这五大类总共包含<a href="https://en.wikipedia.org/wiki/List_of_HTTP_status_codes" target="_blank">100多种</a>状态码，覆盖了绝大部分可能遇到的情况。每一种状态码都有标准的（或者约定的）解释，客户端只需查看状态码，就可以判断出发生了什么情况，所以服务器应该返回尽可能精确的状态码。</p>

<p>API 不需要<code>1xx</code>状态码，下面介绍其他四类状态码的精确含义。</p>

<h3>2.2 2xx 状态码</h3>

<p><code>200</code>状态码表示操作成功，但是不同的方法可以返回更精确的状态码。</p>

<blockquote>
  <ul>
<li>GET: 200 OK</li>
<li>POST: 201 Created</li>
<li>PUT: 200 OK</li>
<li>PATCH: 200 OK</li>
<li>DELETE: 204 No Content</li>
</ul>
</blockquote>

<p>上面代码中，<code>POST</code>返回<code>201</code>状态码，表示生成了新的资源；<code>DELETE</code>返回<code>204</code>状态码，表示资源已经不存在。</p>

<p>此外，<code>202 Accepted</code>状态码表示服务器已经收到请求，但还未进行处理，会在未来再处理，通常用于异步操作。下面是一个例子。</p>

<blockquote><pre class=" language-http"><code class=" language-http">
HTTP/1.1 202 Accepted

{
  "task": {
    "href": "/api/company/job-management/jobs/2130040",
    "id": "2130040"
  }
}
</code></pre></blockquote>

<h3>2.3 3xx 状态码</h3>

<p>API 用不到<code>301</code>状态码（永久重定向）和<code>302</code>状态码（暂时重定向，<code>307</code>也是这个含义），因为它们可以由应用级别返回，浏览器会直接跳转，API 级别可以不考虑这两种情况。</p>

<p>API 用到的<code>3xx</code>状态码，主要是<code>303 See Other</code>，表示参考另一个 URL。它与<code>302</code>和<code>307</code>的含义一样，也是"暂时重定向"，区别在于<code>302</code>和<code>307</code>用于<code>GET</code>请求，而<code>303</code>用于<code>POST</code>、<code>PUT</code>和<code>DELETE</code>请求。收到<code>303</code>以后，浏览器不会自动跳转，而会让用户自己决定下一步怎么办。下面是一个例子。</p>

<blockquote><pre class=" language-http"><code class=" language-http">
HTTP/1.1 303 See Other
<span class="token keyword">Location:</span> /api/orders/12345
</code></pre></blockquote>

<h3>2.4 4xx 状态码</h3>

<p><code>4xx</code>状态码表示客户端错误，主要有下面几种。</p>

<p><code>400 Bad Request</code>：服务器不理解客户端的请求，未做任何处理。</p>

<p><code>401 Unauthorized</code>：用户未提供身份验证凭据，或者没有通过身份验证。</p>

<p><code>403 Forbidden</code>：用户通过了身份验证，但是不具有访问资源所需的权限。</p>

<p><code>404 Not Found</code>：所请求的资源不存在，或不可用。</p>

<p><code>405 Method Not Allowed</code>：用户已经通过身份验证，但是所用的 HTTP 方法不在他的权限之内。</p>

<p><code>410 Gone</code>：所请求的资源已从这个地址转移，不再可用。</p>

<p><code>415 Unsupported Media Type</code>：客户端要求的返回格式不支持。比如，API 只能返回 JSON 格式，但是客户端要求返回 XML 格式。</p>

<p><code>422 Unprocessable Entity</code> ：客户端上传的附件无法处理，导致请求失败。</p>

<p><code>429 Too Many Requests</code>：客户端的请求次数超过限额。</p>

<h3>2.5 5xx 状态码</h3>

<p><code>5xx</code>状态码表示服务端错误。一般来说，API 不会向用户透露服务器的详细信息，所以只要两个状态码就够了。</p>

<p><code>500 Internal Server Error</code>：客户端请求有效，服务器处理时发生了意外。</p>

<p><code>503 Service Unavailable</code>：服务器无法处理请求，一般用于网站维护状态。</p>

<h2>三、服务器回应</h2>

<h3>3.1 不要返回纯本文</h3>

<p>API 返回的数据格式，不应该是纯文本，而应该是一个 JSON 对象，因为这样才能返回标准的结构化数据。所以，服务器回应的 HTTP 头的<code>Content-Type</code>属性要设为<code>application/json</code>。</p>

<p>客户端请求时，也要明确告诉服务器，可以接受 JSON 格式，即请求的 HTTP 头的<code>ACCEPT</code>属性也要设成<code>application/json</code>。下面是一个例子。</p>

<blockquote><pre class=" language-http"><code class=" language-http">
GET /orders/2 HTTP/1.1
<span class="token keyword">Accept:</span> application/json
</code></pre></blockquote>

<h3>3.2 发生错误时，不要返回 200 状态码</h3>

<p>有一种不恰当的做法是，即使发生错误，也返回<code>200</code>状态码，把错误信息放在数据体里面，就像下面这样。</p>

<blockquote><pre class=" language-http"><code class=" language-http">
HTTP/1.1 200 OK
<span class="token keyword">Content-Type:</span> application/json<span class="token application/json">

<span class="token punctuation">{</span>
  <span class="token string">"status"</span><span class="token punctuation">:</span> <span class="token string">"failure"</span><span class="token punctuation">,</span>
  <span class="token string">"data"</span><span class="token punctuation">:</span> <span class="token punctuation">{</span>
    <span class="token string">"error"</span><span class="token punctuation">:</span> <span class="token string">"Expected at least two items in list."</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</span></code></pre></blockquote>

<p>上面代码中，解析数据体以后，才能得知操作失败。</p>

<p>这张做法实际上取消了状态码，这是完全不可取的。正确的做法是，状态码反映发生的错误，具体的错误信息放在数据体里面返回。下面是一个例子。</p>

<blockquote><pre class=" language-http"><code class=" language-http">
HTTP/1.1 400 Bad Request
<span class="token keyword">Content-Type:</span> application/json<span class="token application/json">

<span class="token punctuation">{</span>
  <span class="token string">"error"</span><span class="token punctuation">:</span> <span class="token string">"Invalid payoad."</span><span class="token punctuation">,</span>
  <span class="token string">"detail"</span><span class="token punctuation">:</span> <span class="token punctuation">{</span>
     <span class="token string">"surname"</span><span class="token punctuation">:</span> <span class="token string">"This field is required."</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</span></code></pre></blockquote>

<h3>3.3 提供链接</h3>

<p>API 的使用者未必知道，URL 是怎么设计的。一个解决方法就是，在回应中，给出相关链接，便于下一步操作。这样的话，用户只要记住一个 URL，就可以发现其他的 URL。这种方法叫做 HATEOAS。</p>

<p>举例来说，GitHub 的 API 都在 <a href="https://api.github.com/" target="_blank">api.github.com</a> 这个域名。访问它，就可以得到其他 URL。</p>

<blockquote><pre class=" language-http"><code class=" language-http">
{
  ...
  "feeds_url": "<a class="token url-link" href="https://api.github.com/feeds">https://api.github.com/feeds</a>",
  "followers_url": "<a class="token url-link" href="https://api.github.com/user/followers">https://api.github.com/user/followers</a>",
  "following_url": "<a class="token url-link" href="https://api.github.com/user/following">https://api.github.com/user/following</a>{/target}",
  "gists_url": "<a class="token url-link" href="https://api.github.com/gists">https://api.github.com/gists</a>{/gist_id}",
  "hub_url": "<a class="token url-link" href="https://api.github.com/hub">https://api.github.com/hub</a>",
  ...
}
</code></pre></blockquote>

<p>上面的回应中，挑一个 URL 访问，又可以得到别的 URL。对于用户来说，不需要记住  URL 设计，只要从 api.github.com 一步步查找就可以了。</p>

<p>HATEOAS 的格式没有统一规定，上面例子中，GitHub 将它们与其他属性放在一起。更好的做法应该是，将相关链接与其他属性分开。</p>

<blockquote><pre class=" language-http"><code class=" language-http">
HTTP/1.1 200 OK
<span class="token keyword">Content-Type:</span> application/json<span class="token application/json">

<span class="token punctuation">{</span>
  <span class="token string">"status"</span><span class="token punctuation">:</span> <span class="token string">"In progress"</span><span class="token punctuation">,</span>
   <span class="token string">"links"</span><span class="token punctuation">:</span> <span class="token punctuation">{</span><span class="token punctuation">[</span>
    <span class="token punctuation">{</span> <span class="token string">"rel"</span><span class="token punctuation">:</span><span class="token string">"cancel"</span><span class="token punctuation">,</span> <span class="token string">"method"</span><span class="token punctuation">:</span> <span class="token string">"delete"</span><span class="token punctuation">,</span> <span class="token string">"href"</span><span class="token punctuation">:</span><span class="token string">"/api/status/12345"</span> <span class="token punctuation">}</span> <span class="token punctuation">,</span>
    <span class="token punctuation">{</span> <span class="token string">"rel"</span><span class="token punctuation">:</span><span class="token string">"edit"</span><span class="token punctuation">,</span> <span class="token string">"method"</span><span class="token punctuation">:</span> <span class="token string">"put"</span><span class="token punctuation">,</span> <span class="token string">"href"</span><span class="token punctuation">:</span><span class="token string">"/api/status/12345"</span> <span class="token punctuation">}</span>
  <span class="token punctuation">]</span><span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</span></code></pre></blockquote>
