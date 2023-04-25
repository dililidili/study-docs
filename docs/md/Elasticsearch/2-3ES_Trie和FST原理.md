# 2.3、ES-Trie和FST原理

## 1.前缀树

<img src="https://raw.githubusercontent.com/dililidili/study-docs/main/docs/img/ELasticsearch/es2-3-1.png" style="zoom: 80%;" />

依次输入：msb、msn、msbtech、wltech会产生如上图数据结构。

1、如果出现可以公用的元素，则另开分支将不可以公用的部分进行存储，最后一个节点标记为绿色。

2、在查找时按照从头到尾的顺序进行查找，只有每个节点都符合并且最后一个字母为绿色final节点时代表查询成功。

3、若没有可以公用的部分，则单独开分支进行存储，如wltech。

![](https://raw.githubusercontent.com/dililidili/study-docs/main/docs/img/ELasticsearch/es2-3-2.png)

**但是此时有一问题，msbtech和wltech在前缀上没有可以公用的部分，但是tech可以公用。**

## 2.FSA

​		FSA增加了Entry和Final的概念，也就是由状态转换的不确定性变为了确定，由闭环变为了单向有序，这一点和Trie是类似的，但是不同的是，FSA的Final节点是唯一的，也是因为这个原因，FSA在录入和Trie相同的Term Dictionary数据的时候，从第三步开始才表现出了区别，即尾部复用。如果在第三步的时候还不太明显，那第四步中就可以清楚的看到FSA在后缀的处理上更加高效。

当输入wltech时，tech可以复用，得出FSA将tech直接复用减少存储空间。

![](https://raw.githubusercontent.com/dililidili/study-docs/main/docs/img/ELasticsearch/es2-3-3.png)

**注意：**当输入msn时，如果是前缀树，n节点会单独新增一个节点表示final节点。在使用FSA之后n节点直接指向最后的final节点。

但是这里又会产生一个问题：wl是否存在呢？因为wl的下一个节点为final节点，中止节点，按理说是应该存在的，但是结果是不存在的

## 3.FST

​		FST的压缩率非常高，相比HashMap，FST的性能相差的并不多，但是可以大大的节省空间占用。“搜索引擎”级别的词项字典动辄几亿甚至几十亿的数量级，如果使用FST对其进行存储，其高效的数据存储使得数据被压缩的很小，使其完全缓存在内存中成为了可能。FST在Lucene中的应用非常广泛，比如同义词处理、模糊查询、Suggest等都有应用。


![](https://raw.githubusercontent.com/dililidili/study-docs/main/docs/img/ELasticsearch/es2-3-4.png)

**FST会在final节点中新增Final Output数值，当查找某一字符串的时候，会根据当前字符串的路径相加每个节点中的value得到最终值与初始value对应判断是否一致,从而实现key-value的映射。**

