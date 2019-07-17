## 谈到hashMap，先来说说hashMap的数据结构
### 在阅读之前，需要带着几个问题走：
- hashMap数据结构，怎么保证增删改查的效率？
- hashMap用什么储存元素？
- hashMap怎么储存元素？(容量，寻址等等)
![结构示意图](https://img-blog.csdnimg.cn/20190711134303162.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMyODA4Mzk5,size_16,color_FFFFFF,t_70)
1. hashMap是由数组加链表，jkd8中链表长度超过8时会转成红黑树
2. 通过put方存入元素，这里我们可以  、ctrl+鼠标左键  点进去源码里面看(博主用的是idea)，会有这样一个put方法，具体后面会说到
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190711135706327.png)
3. 存进去的单个元素，是由hasMap里面的Node<K,V>这个静态内部类来储存的，然后把每个Node<K,V>存到Node<K,V>[]数组里
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190711134714594.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190711135410835.png)
## 现在HashMap的储存结构大概的了解了，就来看看具体怎么存的把
- put方法里面有5个参数，目前我们只讨论前三个
- 第一个是  hash(key)，这个是做什么的呢？我们先进去源码看看：
- ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190711140025947.png)
- 这里是做了一个哈希值的计算，然后跟自身右移16位后做异或(这里基础的计算就不多说了)，至于为什么要这样做呢？
1. 我们知道int类型是总共有32位，而右移16位后 刚好是取了整个int的高16位 ，再拿高16位和自身异或，这样就可以充分的把int的32位都用起来，会更加散列。这时带着散列有什么用？的问题继续往下看--->
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190711141035885.png)
2. 然后再回到最前面hashMap的数据结构，我们可以从源码中看到HashMAp的初始容量是16
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190711142120777.png)
3. 那么我们添加第十六个元素以内的元素时，元素应该放在第几个索引处呢？这时候前面计算出来的散列值就有用了。第一个if判断我就不说了，相信大家看得懂(resize方法是用来初始化Node<K,V>[]数组大小，或者扩大数组容量用的)。我们直接看下图高亮的地方，n 其实就是Node<K,V>[]这个数组的大小。重点是tab[i=(n-1)&hash]，注意！注意！这里的n的大小是2的幂次方大小(至于为什么往后resize()面看)，而hash就是第1步得到的那个！！！，可以看出（n-1）& hash是一只小于等于 n 得，所以这里 不可能会 发生下表越界。所以就这样存入了元素。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190711142729202.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMyODA4Mzk5,size_16,color_FFFFFF,t_70)
##  说说resize()方法，如图
相信大家可以很容易看懂这段代码，至于为什么table大小是2的幂次方，是因为2进制的数中它的最高位总是1，后面全是0如（10000），而减掉1之后，恰好反过来了（01111），这样再进行与(&)运算的时候就会有跟大的散列度。举个例子，如果n-1为01011，那么第三位的那个0，不管与什么与运算都为0，就只存在了0这一种值，从排列组合角度看，无形之中就少了一半。这样算出来的作为索引的话，就有更大的可能落到之前的已有元素的索引上，就会造成重新寻址等等，降低性能！！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190711144344615.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190711144551603.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190711144614359.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMyODA4Mzk5,size_16,color_FFFFFF,t_70)

# 最后，由于博主知识有限，有什么不好的，或者不清楚的地方，可以顺着网线过来打我！！！哈哈哈