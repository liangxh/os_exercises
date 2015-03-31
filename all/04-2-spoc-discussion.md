#lec9 虚存置换算法spoc练习

## 个人思考题
1. 置换算法的功能？

2. 全局和局部置换算法的不同？

3. 最优算法、先进先出算法和LRU算法的思路？

4. 时钟置换算法的思路？

5. LFU算法的思路？

6. 什么是Belady现象？

7. 几种局部置换算法的相关性：什么地方是相似的？什么地方是不同的？为什么有这种相似或不同？

8. 什么是工作集？

9. 什么是常驻集？

10. 工作集算法的思路？

11. 缺页率算法的思路？

12. 什么是虚拟内存管理的抖动现象？

13. 操作系统负载控制的最佳状态是什么状态？

## 小组思考题目

----
(1)（spoc）请证明为何LRU算法不会出现belady现象

```
考虑物理页大小n的LRU (后称为A)和物理页大小n+1的LRU(后称为B), 对相同的仿存请求序列, 相同的初始状态(全空).当发生n次缺失后, A的物理页列表已满, 而B的未满. 当A发生第n+1次页缺失, B没有缺失, 并且比A多对应一个他没有的页. 假设在t时刻A对应的页都在B中, 若在t+1时刻有访存请求, 有如下情况
1) 在A中没有缺失, 则由归纳假设, B也没有缺失.
2) 在A中缺失而需要换出最近最少使用页, 此时B有如下情况
	i) B没有缺失, 归纳假设于t+1时刻依然成立
	i) B缺失而需要换出最近最少使用页, 针对LRU, 若为B中的最近最少使用, 则应在A上一次缺页时被置换出, 因此换入新页后, A对应的页也都在B中, 归纳假设于t+1时刻依然成立
综合以上由数学归纳法可得在任意时刻, A对应的页都在B中, 即A出现页失的时间序列是B的子集, 即LRU算法不会出现belady现象.
```

(2)（spoc）根据你的`学号 mod 4`的结果值，确定选择四种替换算法（0：LRU置换算法，1:改进的clock 页置换算法，2：工作集页置换算法，3：缺页率置换算法）中的一种来设计一个应用程序（可基于python, ruby, C, C++，LISP等）模拟实现，并给出测试。请参考如python代码或独自实现。
 - [页置换算法实现的参考实例](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab3/page-replacement-policy.py)

```
if __name__ == '__main__':
	print 2012011290 % 4

	#example from video 05
	addrList = [3, 3, 4, 2, 3, 5, 3, 5, 1, 4]
	memory = [1, 4, 5]
	memory_t = [0, 1, 2]

	miss = 0
	hits = 0
	threshold = 4

	for n in addrList:
            idx = -1
	    for i in range(len(memory)):
		if memory[i] == n:
		    idx = i
		    break
	    if idx != -1:
		hits = hits + 1
		memory_t[idx] = -1
		
	    else:
		miss = miss + 1
		memory.append(n)
		memory_t.append(-1)
		

	    del_list = []

	    for i in range(len(memory)):
		memory_t[i] = memory_t[i] + 1
		if memory_t[i] == threshold:
		    del_list.append(i)

	    for i in range(len(del_list)):
		del memory[del_list[i] - i]
		del memory_t[del_list[i] - i]

	    if idx != -1:
		print 'Access: %d HIT -> '%(n), memory, ' <- Replaced:- [Hits:%d Missess:%d]'%(hits, miss)
	    else:
		print 'Access: %d MISS -> '%(n), memory, ' <- Replaced:- [Hits:%d Missess:%d]'%(hits, miss) 
		

	print ''
	print 'FINALSTATS hits %d   misses %d   hitrate %.2f' % (hits, miss, (100.0*float(hits))/(float(hits)+float(miss)))
	print ''

```

## 扩展思考题
（1）了解LIRS页置换算法的设计思路，尝试用高级语言实现其基本思路。此算法是江松博士（导师：张晓东博士）设计完成的，非常不错！

参考信息：

 - [LIRS conf paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang02_LIRS.pdf)
 - [LIRS journal paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang05_LIRS.pdf)
 - [LIRS-replacement ppt1](http://dragonstar.ict.ac.cn/course_09/XD_Zhang/(6)-LIRS-replacement.pdf)
 - [LIRS-replacement ppt2](http://www.ece.eng.wayne.edu/~sjiang/Projects/LIRS/sig02.ppt)
