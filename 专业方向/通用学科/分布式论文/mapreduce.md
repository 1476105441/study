# 2 编程模型

## 2.1 例子

考虑这样一个问题：在一个庞大的文档集合中计算每一个单词出现的次数。用户可能会写出类似下面的伪代码：

~~~
map(String key,String value):
	//key: document name
	//value: document contents
	for each word w in value:
		EmitIntermediate(w,"1");
		
reduce(String key,Iterator values):
	//key: a word
	//value: a list of counts
	int result = 0;
	for each v in values:
		result += ParseInt(v);
	Emit(AsString(result));
~~~

map函数输送每个单词和其出现的次数（这个简单的例子中为“1”）。reduce函数将每一个单词出现的次数加到一起，并输出为结果。

此外，用户编写代码去填补MapReduce缺失的部分（specification object），指定输入输出的文件名称、可选的调节参数。然后调用MapReduce函数，将刚刚填写的specification object传入进去。接着，用户编写的代码就可以和MapReduce库链接在一起了。附录A包含了这个例子中的所有代码。

## 2.3 类型

尽管先前的伪代码是用字符串输入和输出函数写的，但用户提供的map和reduce函数在概念上有相似的类型：

~~~
map		(k1,v1)			-> list(k2,v2)
reduce	(k2,list(v2))	 -> list(v2)
~~~

换句话说，输入的key和value来自于与输出的key和value不同的域。此外，中间的key和value与输出的key和value来自同一个域。

我们的C++实现传递字符串给用户定义的函数，并留给用户代码，以便在字符串和合适类型之间转换。



# 3、实现

MapReduce接口可能拥有许多不同的实现。需要根据环境来确定正确的选择。例如，一个实现可能适合于共享内存较小的机器，另一种则适合于大的NUMA多进程的机器，也有适合于一个巨大的通过网络连接的机器集群的实现。