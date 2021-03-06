### minisearchengine介绍
minisearchengine是一个很小的搜索引擎示例，它分为3个步骤来构造引擎，分别是爬虫、索引以及排名。利用redis作为页面的缓存队列，为的是能让爬虫与索引并行化运行。

### minisearchengine模块

#### 爬虫
爬虫用的是open-uri与nokogiri库再加上BFS广度优先搜索算法，抓取每个页面后利用rmmseg-cpp包进行中文分词，分词后将单词列表放入redis队列中。

#### 索引
索引的目的是为了将单词与页面做好关联，这样每次搜索就不需要搜索原有的页面，只要搜索关联的表就可以查到所对应的页面。

#### 排名
搜索的排名对于搜索引擎的用户才是最重要的，因为涉及到页面与内容的相关性，minisearchengine采用了四种不同的相关度的计算方法，分别是单词的频度、单词在文档中的位置、单词距离以及PageRank算法。

单词频度：这种方法利用查询条件中的单词在网页中出现的次数来对网页进行评价的。

文档位置：利用单词可能在靠近网页开始处的位置来计算每个单词距离开始位置的距离进行评价。

单词距离：利用多个单词之间最短的距离来评价，因为在某些情形下最接近的单词，或许是个词组，所以页面的相关度也会与此有关。

PageRank算法： 这是谷歌提出的算法，当时google的崛起或多或少的与这个算法有关。它是通过利用外链（其他网页对此网页的链接数目）以及外链的宿主页面（指向搜索的页面链接所在的那个页面）的评价值，来计算需要搜索页面的评价值并进行的排序。

### minisearchengine的使用

#### 安装
安装并运行redis：

	sudo apt-get install redis
	nohup redis-server &

下载minisearchengine并安装包：

	git clone https://github.com/kimboqi/minisearchengine.git
	cd minisearchengine
	bundle install
	
打开minisearchengine/config/database.yml，配置mysql数据库，并运行：

    rake db:create
    rake db:migrate

建立数据库后，在minisearchengine/Rakefile.rb中配置需要抓取的一些起始页面，然后运行：

	rake se:crawl
	rake se:index
	rake db:seed
	rake se:pagerank
	
如果页面多的话这个步骤是可以并行来做的。索引完成后就可以查询了：

	rake se:search
	
打开浏览器输入127.0.0.1:9293，输入你要查询的单词就OK了。
