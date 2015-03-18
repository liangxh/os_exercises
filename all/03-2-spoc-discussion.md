# lec6 SPOC思考题


NOTICE
- 有"w3l2"标记的题是助教要提交到学堂在线上的。
- 有"w3l2"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题
---

（1） (w3l2) 请简要分析64bit CPU体系结构下的分页机制是如何实现的
```
  + 采分点：说明64bit CPU架构的分页机制的大致特点和页表执行过程
  - 答案没有涉及如下3点；（0分）
  - 正确描述了64bit CPU支持的物理内存大小限制（1分）
  - 正确描述了64bit CPU下的多级页表的级数和多级页表的结构或反置页表的结构（2分）
  - 除上述两点外，进一步描述了在多级页表或反置页表下的虚拟地址-->物理地址的映射过程（3分）
 ```
- [x]  

>  

```
對64bit CPU, 可以處理的內存大小最大為2^64bytes (但事實上目前還没有成功實現的硬件), 以為了滿足更高效的內存分配以及更大的虛擬存儲空間, CPU體系結構下存在分頁機制.

而對於多級頁表, 其級數對應虛擬地址轉換成物理地址的過程中要經過多少的頁表的對應, 同時對應虛擬地址的高位有多少個頁表下標. 每個頁表何多個頁表項, 每個頁表項含兩個部分: 標誌位, 同於表示不同狀態, 如valid, read-only; 基址, 是下一級頁表的基址, 或幀的基址. 虛擬地址根據高位的頁表下標經過一級或多級轉換后便能得出實際存儲的物理地址.

考虑實現對2^70bytes虛擬空間的管理, 可設計2級頁表, 對70位的虛擬地址, 60~69位為第一級頁表項的下標, 50~59位為第二級頁表項的下標. 則
第一級頁表基址 + 為第一級頁表項的下標 = 第二級頁表基址
第二級頁表基址 + 為第二級頁表項的下標 = 物理幀基址
物理幀基址_偏移量(0~49位) = 物理地址
```

## 小组思考题
---

（1）(spoc) 某系统使用请求分页存储管理，若页在内存中，满足一个内存请求需要150ns。若缺页率是10%，为使有效访问时间达到0.5ms,求不在内存的页面的平均访问时间。请给出计算步骤。 

- [x]  

> 500=0.9\*150+0.1\*x

```
500 000 = 0.9 \* 150 + 0.1 \* x
x = 4998650.0 (ns) = 4.999 (ms)
```

（2）(spoc) 有一台假想的计算机，页大小（page size）为32 Bytes，支持32KB的虚拟地址空间（virtual address space）,有4KB的物理内存空间（physical memory），采用二级页表，一个页目录项（page directory entry ，PDE）大小为1 Byte,一个页表项（page-table entries
PTEs）大小为1 Byte，1个页目录表大小为32 Bytes，1个页表大小为32 Bytes。页目录基址寄存器（page directory base register，PDBR）保存了页目录表的物理地址（按页对齐）。

PTE格式（8 bit） :
```
  VALID | PFN6 ... PFN0
```
PDE格式（8 bit） :
```
  VALID | PT6 ... PT0
```
其
```
VALID==1表示，表示映射存在；VALID==0表示，表示映射不存在。
PFN6..0:页帧号
PT6..0:页表的物理基址>>5
```
在[物理内存模拟数据文件](./03-2-spoc-testdata.md)中，给出了4KB物理内存空间的值，请回答下列虚地址是否有合法对应的物理内存，请给出对应的pde index, pde contents, pte index, pte contents。
```
Virtual Address 6c74
Virtual Address 6b22
Virtual Address 03df
Virtual Address 69dc
Virtual Address 317a
Virtual Address 4546
Virtual Address 2c03
Virtual Address 7fd7
Virtual Address 390e
Virtual Address 748b
```

比如答案可以如下表示：
```
Virtual Address 7570:
  --> pde index:0x1d  pde contents:(valid 1, pfn 0x33)
    --> pte index:0xb  pte contents:(valid 0, pfn 0x7f)
      --> Fault (page table entry not valid)
      
Virtual Address 21e1:
  --> pde index:0x8  pde contents:(valid 0, pfn 0x7f)
      --> Fault (page directory entry not valid)

Virtual Address 7268:
  --> pde index:0x1c  pde contents:(valid 1, pfn 0x5e)
    --> pte index:0x13  pte contents:(valid 1, pfn 0x65)
      --> Translates to Physical Address 0xca8 --> Value: 16
```

> 

```
Virtual Address 0x6c74:
    --> pde index: 0x1b pde contens:(valid 1, pfn 0x20)
      --> pde index: 0x3 pde contens:(valid 1, pfn 0x61)
        Translates to Physical Address 0xc34 --> Value: 0x6

Virtual Address 0x6b22:
    --> pde index: 0x1a pde contens:(valid 1, pfn 0x52)
      --> pde index: 0x19 pde contens:(valid 1, pfn 0x47)
        Translates to Physical Address 0x8e2 --> Value: 0x1a

Virtual Address 0x3df:
    --> pde index: 0x0 pde contens:(valid 1, pfn 0x5a)
      --> pde index: 0x1e pde contens:(valid 1, pfn 0x5)
        Translates to Physical Address 0xbf --> Value: 0xf

Virtual Address 0x69dc:
    --> pde index: 0x1a pde contens:(valid 1, pfn 0x52)
      --> pde index: 0xe pde contens:(valid 0, pfn 0x7f)
        --> Fault (page directory entry not valid)

Virtual Address 0x317a:
    --> pde index: 0xc pde contens:(valid 1, pfn 0x18)
      --> pde index: 0xb pde contens:(valid 1, pfn 0x35)
        Translates to Physical Address 0x6ba --> Value: 0x1e

Virtual Address 0x4546:
    --> pde index: 0x11 pde contens:(valid 1, pfn 0x21)
      --> pde index: 0xa pde contens:(valid 0, pfn 0x7f)
        --> Fault (page directory entry not valid)

Virtual Address 0x2c03:
    --> pde index: 0xb pde contens:(valid 1, pfn 0x44)
      --> pde index: 0x0 pde contens:(valid 1, pfn 0x57)
        Translates to Physical Address 0xae3 --> Value: 0x16

Virtual Address 0x7fd7:
    --> pde index: 0x1f pde contens:(valid 1, pfn 0x12)
      --> pde index: 0x1e pde contens:(valid 0, pfn 0x7f)
        --> Fault (page directory entry not valid)

Virtual Address 0x390e:
    --> pde index: 0xe pde contens:(valid 0, pfn 0x7f)
      --> Fault (page directory entry not valid)

Virtual Address 0x748b:
    --> pde index: 0x1d pde contens:(valid 1, pfn 0x0)
      --> pde index: 0x4 pde contens:(valid 0, pfn 0x7f)
        --> Fault (page directory entry not valid)
```

Code

```
def hex_to_dec():
	return

def read_mem(fname):
	fobj = open(fname, 'r')
	mem = []
	for line in fobj:
		if line.startswith('page'):
			tmp_line = line.split(':')
			tmp_line = tmp_line[1].split(' ')
			del tmp_line[0], tmp_line[len(tmp_line) - 1]
			mem.append(tmp_line)
	
	zeros = []
	zeros.append('')
	for i in range(1, 9):
		zeros.append('0' + zeros[-1])
	
	for i in range(len(mem)):
		for j in range(len(mem[i])):
			mem[i][j] = bin(int(mem[i][j], 16))
			mem[i][j] = mem[i][j][2:len(mem[i][j])]
			mem[i][j] = zeros[8-len(mem[i][j])] + mem[i][j]
	
	return mem

if __name__ == '__main__':
	mem = read_mem('mem.txt')
	vas = ['6c74','6b22','03df','69dc','317a','4546','2c03','7fd7', '390e','748b'
]

	pdbr = int('220', 16) / 32

	zeros = []
	zeros.append('')
	for i in range(1, 9):
		zeros.append('0' + zeros[-1])

	for va in vas:
		va = bin(int(va, 16))
		va = va[2: len(va)]
		va = zeros[8 - len(va)] + va

		va = va[(len(va) - 16):len(va)]
		pde = int(va[1:6], 2)
		pte = int(va[6:11], 2)
		bias = int(va[11:16], 2)

		indent = '  '
		print 'Virtual Address ' + hex(int(va, 2)) + ':'
		indent = '  '+indent
		
		content = mem[pdbr][pde]
		index = int(content[1:8], 2)
	
		print indent + '--> pde index: '+ hex(pde) + ' pde contens:(valid ' + content[0] + ', pfn ' + hex(index) + ')'
		indent = '  '+indent

		if content[0] == '0':
			print indent + '--> Fault (page directory entry not valid)\n'
			continue
	
		content = mem[index][pte]
		index = int(content[1:8], 2)
		print indent + '--> pde index: '+ hex(pte) + ' pde contens:(valid ' + content[0] + ', pfn ' + hex(index) + ')'
		indent = '  '+indent

		if content[0] == '0':
			print indent + '--> Fault (page directory entry not valid)\n'
			continue

		value = mem[index][bias]
		index = bin(index)
		index = hex(int(index[2:len(index)] + va[11:16], 2))
		print indent + 'Translates to Physical Address '+index+' --> Value: ' +hex(int(value, 2))+ '\n'
		
```


（3）请基于你对原理课二级页表的理解，并参考Lab2建页表的过程，设计一个应用程序（可基于python, ruby, C, C++，LISP等）可模拟实现(2)题中描述的抽象OS，可正确完成二级页表转换。


（4）假设你有一台支持[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)的机器，请问你如何设计操作系统支持这种类型计算机？请给出设计方案。

 (5)[X86的页面结构](http://os.cs.tsinghua.edu.cn/oscourse/OS2015/lecture06#head-1f58ea81c046bd27b196ea2c366d0a2063b304ab)
--- 

## 扩展思考题

阅读64bit IBM Powerpc CPU架构是如何实现[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)，给出分析报告。

--- 
