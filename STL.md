## allocate

1. C++的new和C中的malloc

   C++中内存配置的基本操作是new，内存释放的基本操作是delete；相当于C中的malloc与free。其实，**C++中的new和delete底层实现就是malloc和free**。

   考虑到小区块可能造成内存破碎的问题，对于大小大于128字节的内存申请，调用第一级配置器，即直接使用malloc和free；对小于128字节的内存申请，使用第二级配置器，采用复杂的memory pool整理方式。其中，第二级配置器**维护16个自由链表，负责16种小型区块的次配置能力，内存池以malloc方式得到，如果内存不足，则自动转入第一级配置器申请内存。**

2. realloc

   头文件：\#include <stdlib.h> 有些编译器需要#include <alloc.h>

   函数原型：extern void *realloc(void *mem_address, unsigned int newsize)

   功能：将第一个参数mem_address指向的内存区域的大小resize为newsize

   返回值：如果重新分配成功则返回指向被分配内存的指针，否则返回空指针NULL。

   **注意：realloc首先从原size后查看是否有一块连续内存使得加上之后变成newsize，如果有则加上。如果没有则要在堆上面找到一块newsize这么大的内存，然后将原内存中的内容复制到这块内存中，然后返回这块新内存的开头位置的指针。之前那一块内存会放回堆里面以后再用。**

3. 在STL中，由于不是利用new来配置内存的，所以内存配置失败不能使用set_new_handler。但是其内部实现了一个类似功能的函数oom_malloc和oom_realloc。在内存配置失败的时候会检查者两个函数如果不为空，就会不断循环执行这两个函数直到成功。**如果没有定义，会返回bad_alloc异常，然后exit(1)退出。**

4. STL中第二级内存配置器，用于分配小于128字节的内存

   内存池（memory pool）又称为次层配置（sub-allocation）：**每次配置一大块内存，并维护与之对应的自由链表。**如果多次请求相同大小的内存块，就直接从自由链表内取得；当释放内存块时，会自动回到自由链表中。**自由链表中内存块有16中大小，最小为8，最大为128，都是8的倍数，当申请的内存块不是8的倍数的时候，会自动增大到8的倍数，大于128的就用第一级配置器。自由链表中内存块的地址不是连续的，是通过链表来连接的，节点是一个union。**从自由链表中取得的内存块是从链表头取得的，所以每次都会调整自由链表的链表头指向被取走内存块的->free_list_link，相当于头部删除。如果自由链表没有可用的一个区块，就会调用refill函数填充自由链表空间。

5. deallocate

   回收内存与分配内存类似，如果内存大小大于128，调用第一级配置器free了；如果小于128，找到对应的自由链表，然后头插法。

6. refill

   调用chunk_alloc（size_t size，int& nobjs），size表示内存块的大小，nobjs表示内存块的数量。次函数从内存池那里获得n个新区块，默认为20，内存不足是分不到20块。取得的第0块内存块返回给请求分配内存的进程，剩余的加入到自由链表，**每次refill只能分到n块相同大小的内存块**。

7. **内存池（memory pool）**

   内存池是一大块内存，用free_start和free_end来标记内存池占有内存的起点和终点。refill会调用chunk_alloc来从内存池中申请内存，此时才会根据请求内存的大小返回对应大小的内存块给客户端，分割还是交给调用的那个函数自己分割。**如果申请了n块x大小的内存，内存池没有那么多，会将至少一块内存返回给客户端。例如：chunk_alloc(32,20)申请20块32大小的内存块，内存池只剩下32*10这么多内存，就返回10块，1块给客户，剩余9块归自由链表。如果内存池连一块内存都拿不出来，就会利用malloc从heap中取得内存，申请的大小是请求的2倍+n，这个n随着向heap内存申请次数增加而增大。例如，客户端申请5块32的内存块，内存池真的1块也拿不出来了，就malloc申请10块32的内存，5块给客户，5块归到内存池以后用。**如果heap也用尽了，**那就调用第一级配置器，那里有oom_malloc，说不定有办法，如果还是无法解决，只能返回bad_alloc异常。**

8. uninitialized_copy、uninitialized_fill、uninitialized_fill_n

   这三个函数**有一个共同点，除非需要操作的数据是POD类型，也就是说传统C自带的那些struct等类型，否则都要一个一个位置调用对应的构造函数，不能批量复制。**

   1. uninitialized_copy，接受三个参数，first，last，result

      first，last表示需要复制的内存范围，result表示复制输出开始的地方。如果是POD类型，那就调用copy函数（STL中的算法）；如果不是POD类型，就要一个一个位置调用对应的构造函数；**如果是char*字符串类型，可以调用memmove函数，这个函数跟memcopy作用一样，区别在于memcopy的两块内存不能有重叠部分，但是memmove无所谓。**

   2. uninitialized_fill，接受三个参数，first，last，x

      first，last表示需要复制的内存范围，x表示填充这些地方的value。如果是POD类型，那就调用fill来实现最快；如果不是POD类型，那就一个一个第调用构造函数。

   3. uninitialized_fill_n，接受三个参数，first，n，x

      跟uninitialized_fill一样，只不过第二个参数变成了n，可以指定数量了。



## 迭代器iterator



## 序列容器（vector、list等）

1. heap内包含一个vector；priority_queue内包含一个heap；stack和queue内都包含一个deque；set/map/multiset/multimap内包含一个红黑树；hash_x内包含一个hashtable；

2. **序列式容器**：其中的元素都可以有序，但未必有序。

   C++本身提供array，STL提供了vector、list、deque、stack、queue、priority_queue。

   1. vector

      ①vector与array类似，唯一的区别在于**空间的运用灵活性。**array是静态空间，一旦配置无法改变，但是vector会自行扩增空间；

      ②vector维护一个**连续**线性空间，使用的迭代器为**普通指针**。

      ③vector使用start和finish分别指向**已被使用的空间的范围**，end_of_storage指向**连续空间的尾部。**

      ④vector每次遇到空间不足，则将可用空间**扩增至2倍**，如果依然不够继续扩增。但是不是在**源地址后面扩增**，而是**重新配置内存、元素移动、释放空间。**因此，指向原vector的迭代器全部都会失效。

      ⑤erase（f，r）是将r到末尾的元素复制到f开头的内存，然后将r后面的元素全部destory；

      ⑥insert（begin，n，a）首先检查备用空间是否足够；如果足够，检查插入点后的数据是否比要插入的n个要多，根据这个结果重新标注尾部然后开始复制；如果备用空间不足，就扩增空间为**2\*size或者size+n**，然后将begin之前的元素复制过来，然后插入n个a，最后复制原vector剩余的元素。

   2. list

      list即**环状双向链表**，它的好处是**每次插入或者删除都只配置一个空间**，绝对不造成空间的浪费，且插入和删除都是常数时间。

      ①list的迭代器不是普通指针，但其**内部有一个普通的指针。**其中的++或者--操作其实就是通过prev和next和指针的合作完成的。

      ②其内部指针node初始时prev和next都**指向自身**，所以判空就是node==node->next；插入的第一个元素是node->next；尾部（不能装载元素）就是node；

      ③list的insert与其他类型一样，都是在**迭代器前面**。例如，迭代器指向的node值为3，在这里插入，那么新的节点就在3的前面。

      ④由于STL的sort需要随机访问迭代器的支持，所以list的sort只能用**其内部的一个成员函数sort，排序思想是归并排序。**首先在内部声明一个carry和counter[64]的list，counter[i]表示原list的前2^i个元素的排序结果；**每次都从原list中取一个元素，取完删除**；然后先放在counter[0]，然后再取一个元素，通过merge和swap后放在counter[1]；以此类推，等再拿到两个元素的时候与counter[1]的merge和swap然后放在counter[2]，此时有4个元素了；一直循环到最后。
      
   3. deque
   
      deque是一种**双向开口的连续线性空间**，虽然vector也可以实现类似的操作，但是**vector在头部插入元素的效率非常低**。此外，deque的扩张**不需要经历重新配置一块更大的空间、复制元素、释放旧空间，**这个过程。但是，也因此**deque的排序很慢**，可以使用vector排序完成后复制回deque。
   
      deque是逻辑上的连续空间，其实质是一段一段的连续空间。**deque采取一块map（不是STL的map，是一块内存映射）作为主控，**这个map是一小块连续的空间，其中**每个元素都是一个指针，指向一块连续的内存**，deque的元素就放在这些连续的内存中，默认的连续内存大小为512字节。
   
      deque的迭代器不是普通指针，迭代器是一个带有四个指针的结构体：**cur指向当前将要写入的那个内存、first指向这一块512字节连续内存的开始地址、last指向这一块512字节连续内存的结束地址、node指向中控器中这块内存的指针地址，就是用来返回中控的。**，所以begin或者end都是这种结构的。
   
      当**push_back**加入元素的时候，如果满了就需要从中控获取下一块连续内存的地址然后放入；如果是使用**insert**，需要检查**插入的位置前后节点数的数量**，移动数量较少的那一部分；
   
      当**pop_back**的时候，类似的，需要检查是否已经空了，如果空了需要从中控获取前一块连续内存再pop；如果是使用**erase**，需要检查**删除的位置前后节点的数量**，移动数量较少的那一部分，注意这里如果是int等类型不需要删除，直接覆盖然后调整指针即可；
   
      虽然扩张的时候不需要三部曲，但是**map耗尽的时候依然需要三部曲**，但是只是调整中控而已，那些连续内存快不需要调整。
   
   4. stack
   
      栈是一种先进后出的数据结构，就是单边可以插入删除的vector，在默认情况下**使用deque作为stack底层实现。**像这种**修改已经存在的数据结构的接口等形成的另一种容器称为container adapter（配接器）。**stack不支持随机访问，只有一个 top指针可以对其操作。此外，还可以用**list作为底层实现**。
   
   5. queue
   
      与stack对比，queue是一种先进先出的数据结构，**queu无迭代器、只可以push和pop增减元素，**其他操作不支持，默认状态下**使用deque作为底层实现。**queue也是一种container adapter。类似地，可以使用list作为queue的底层实现。
   
   6. heap
   
      堆，是一棵完全二叉树，其实**并不属于STL的容器**，但是priority_queue就是基于堆实现的，默认是大顶堆。由于堆不是STL的容器，因此必须由一种STL容易来实现它，**默认使用vector**。注意，**堆的下标从1开始，**对于节点i，左子节点为2i，右子节点为2i+1，；对于任意节点i的父节点为i/2；
   
      **插入节点**：每次插入都在vector的末端插入，调整方法为**percolate up（上溯）**，即自底向上；
   
      **删除节点**：每次删除都在vector的首部，调整方法为**percolate down（下溯）**，即自顶向下；注意，首先是头结点跟尾结点交换，然后将size-1，不用真的删除元素，只需要调整size；
   
      **堆的排序**：排序之后，这个堆就**不再是一个合法的堆**了。具体方法与删除节点类似，就是不断将头结点放在最后，然后调整其余的节点，以此类推；
   
   7. priority_queue
   
      优先队列默认是大顶堆，**底层实现为vector**，是一种container adapter（配接器）。具体的内容跟heap相似；
   
   8. slist
   
      slist是一个**单链表**，跟forward_list一样都是单链表，那么为什么还需要提供forwar_list呢？好像是没什么区别，使用方法也都类似，但是**forward_list没有size()函数**，这是因为forward就是为了实现极致性能的单链表而努力，维护size会造成性能上的损失。如果不需要查看size最好使用forward_list。
   
      slist自带一个头结点，头结点不存储信息，只代表该单链表存在，头结点的next就是迭代器begin的起点。由于slist**只提供push_front头插法**，如果要插入尾部可以使用insert_after。
   
      

## 关联式容器（set、map红黑树等）

1. 如果不带unordered的，例如set、map、multiset、multimap其底层实现都是**红黑树**；带unordered的都是哈希表；
2. 如果是set则不允许key重复，如果带multi的，则允许key重复；
3. **红黑树**相关：
   1. 由于红黑树的平衡经常需要父节点的相关信息，所以红黑树的节点**比普通的二叉树多了一个parent指针，指向父节点；**
   2. 红黑树的遍历，即operator++（），operator++（int）只是调用前者；因为当前节点的左子树肯定早就全部访问完了，所以如果有右子树，则下一个目标就是右子树的最左下角的节点；如果没有右子树，则判断是否是其父节点的右子节点，如果是就需要一直往上，只有不是父节点的右子节点；

### set

1. set的key==value，且不允许有相同的key；且不允许修改其key和value；
2. set和list一样，插入删除之后，**之前的迭代器依然有效**；
3. **为什么使用set的find函数比algorithm的find更有效？**因为我们的set底层实现是红黑树，所以自带的find是基于红黑树的搜索，而algorithm的find**只是普通的循环搜索；**

### map

1. first和second分别是类型与key和value相同的public的成员变量；
2. map的插入和删除也不影响已有的迭代器，同时只允许修改value，不允许修改key；
3. key和value是用piar存储的；

### multiset

1. 与set一样，只是其插入函数是insert_equal，set的插入函数为insert_unique；

### multimap

1. 与map一样，只是其插入函数是insert_equal，map的插入函数为insert_unique；



### 哈希表

1. 哈希表的模，一般都是质数，即没有约数的整数；例如53、97等；**好处是避免数据分布过于集中；**
2. 一般的哈希表都是拉链来解决冲突的，**哈希表的底层是一个指针数组vector，这有利于动态扩容，每一个vector都有一个指向链表的指针；**遍历的过程就是从vector[i]的链表头结点开始遍历到NULL，然后到达vector[i+1]的链表头结点；
3. **什么时候需要扩容哈希表？**加上新插入的元素之后，此时的元素个数大于哈希表vector的大小，这个时候就扩容；首先，找到下一个质数作为取模的数值；
4. **如何保证扩容之后之前的key，value依然有效呢？**从头开始，将所有的元素依次重新映射到新的哈希表中，在新的vector对应的链表中利用头插法；





## 算法

1. STL中的算法很多都需要头文件algorithm或numeric；

2. 算法总览：

   ![](img/stl1.png)

   ![](img/stl2.png)

   ![](img/stl3.png)

   ![](img/stl4.png)

3. 重点算法讲解：

   1. 一般使用STL的算法需要配合**两个迭代器**，且其区间一般为左闭右开；

   2. **迭代器是一种行为类似于指针的对象，为了实现STL函数的泛化；**

   3. accumulate：**用于计算first到end之间所有值的和**

      ①template<class inputiterator,class T>

      ​    T accumulate(inputiterator first,inputiterator end,T init)；

      其查找范围为[first，end)，init是运算的初值（**防止first与end之间一个元素都没有**）；

      ```c++
      #include<iostream>
      #include<vector>
      #include<algorithm>
      #include<numeric>
      using namespace std;
      int main() {
      	//vector<int> wh{ 5,1,2,0,3,9,7,4,8,6 };
      	vector<int> wh{ 1,2,3,4,5,6,7,8,9 };
      	int sum = accumulate(wh.begin(), wh.end(), 0);
      	cout << sum << endl;//输出45
      	return 0;
      }
      ```

      

      ②template<class inputiterator,class T>

      ​    T accumulate(inputiterator first,inputiterator end,T init,BinaryOperation binary_op)；

      前三个参数与前面一致，第四个参数是自定义的仿函数；

      ```c++
      #include<iostream>
      #include<vector>
      #include<algorithm>
      #include<numeric>
      using namespace std;
      int main() {
      	//vector<int> wh{ 5,1,2,0,3,9,7,4,8,6 };
      	vector<int> wh{ 1,2,3,4,5,6,7,8,9 };
      	int sum = accumulate(wh.begin(), wh.end(), 0, [](int x, int y) {return x + 3*y; });
      	cout << sum << endl;//输出135
      	return 0;
      }
      ```

      这里的底层实现为init=binary_op（init，wh[i]），相当于将1~9的每个数乘以3再求和；

   4. inner_product：求两个数组的内积，也就是乘积和；

      ①template<class inputiterator,class T>

      ​    T inner_product(inputiterator first1,inputiterator end1,inputiterator first2,T init)；

      ```c++
      #include<iostream>
      #include<vector>
      #include<algorithm>
      #include<numeric>
      #include<functional>
      using namespace std;
      int main() {
      	vector<int> res{ 9,8,7,6,5,4,3,2,1 }, wh{ 1,2,3,4,5,6,7,8,9 };
      	int sum = inner_product(wh.begin(), wh.end(), res.begin(),0);
      	cout <<sum<< endl;//输出165，也就是0+9*1+8*2+7*3+...
      	return 0;
      }
      ```

      其底层运算实现为init=init+\*first1 \* \*first2；

      ②template<class inputiterator,class T>

      ​    T inner_product(inputiterator first1,inputiterator end1,inputiterator first2,T init,binaryoperator binary_op1,binaryoperator binary_op2)；

      前四个参数与上面一直，第五和第六个都是仿函数，其底层实现逻辑为：

      **init = binary_op1（init，binary_op2（\*first1,\*first2））**

      ```c++
      #include<iostream>
      #include<vector>
      #include<algorithm>
      #include<numeric>
      #include<functional>
      using namespace std;
      int main() {
      	vector<int> res{ 9,8,7,6,5,4,3,2,1 }, wh{ 1,2,3,4,5,6,7,8,9 };
      	int sum = inner_product(wh.begin(), wh.end(), res.begin(),0,plus<int>(),minus<int>());
      	cout <<sum<< endl;//输出0,也就是0+(9-1)+(8-2)+...
      	return 0;
      }
      ```

   5. partial_sum：计算数组的局部和

      ①template<class inputiterator,class T>

      ​    outputiterator partial_sum(inputiterator first,inputiterator end,outputiterator result)；

      前两个参数表示运算的范围，第三个参数用于接收结果。

      ```c++
      #include<iostream>
      #include<vector>
      #include<algorithm>
      #include<numeric>
      #include<functional>
      using namespace std;
      int main() {
      	vector<int> wh2{ 9,8,7,6,5,4,3,2,1 }, wh1{ 1,2,3,4,5,6,7,8,9 },res(9,0);
      	partial_sum(wh1.begin(), wh1.end(), res.begin());
      	for (int i = 0; i < 9; ++i)
      		cout << res[i] << " ";//输出1 3 6 10 15 21 28 36 45
      	return 0;
      }
      ```

      其中元素res[i]表示wh1中first到i之间所有数的和；其中res第一个元素固定为wh1第一个元素；

      ②template<class inputiterator,class T>

      ​    outputiterator partial_sum(inputiterator first,inputiterator end,outputiterator result,binaryoperator binary_op)；

      ```c++
      #include<iostream>
      #include<vector>
      #include<algorithm>
      #include<numeric>
      #include<functional>
      using namespace std;
      int main() {
      	vector<int> wh2{ 9,8,7,6,5,4,3,2,1 }, wh1{ 1,2,3,4,5,6,7,8,9 },res(9,0);
      	partial_sum(wh1.begin(), wh1.end(), res.begin(),minus<int>());
      	for (int i = 0; i < 9; ++i)
      		cout << res[i] << " ";//输出1 -1 -4 -8 -13 -19 -26 -34 -43
      	return 0;
      }
      ```

      其运算的逻辑为res[i]=binary_op（wh1的前i-1项结果，wh1[i]）；

   6. adjacent_difference：计算每两个相邻的元素之间的差值

      ①template<class inputiterator,class T>

      ​    outputiterator adjacent_difference(inputiterator first,inputiterator end,outputiterator result)；

      

      前两个参数表示运算的区间，第三个参数result表示接收答案的一个迭代器；

      ```c++
      #include<iostream>
      #include<vector>
      #include<algorithm>
      #include<numeric>
      #include<functional>
      using namespace std;
      int main() {
      	vector<int> res(9,0),wh{ 1,2,3,4,5,6,7,8,9 };
      	adjacent_difference(wh.begin(), wh.end(), res.begin());
      	for (int i = 0; i < 9; ++i)
      		cout << res[i] << " ";//输出8个1
      	cout << endl;
      	return 0;
      }
      ```

      注意其返回值是一个迭代器，指向res的下一个地址；

      ②template<class inputiterator,class T>

      ​    outputiterator adjacent_difference(inputiterator first,inputiterator end,outputiterator result,binaryoperator binary_op)；

      前三个函数同上，第四个参数是自定义的一个运算函数；

      ```c++
      #include<iostream>
      #include<vector>
      #include<algorithm>
      #include<numeric>
      #include<functional>
      using namespace std;
      int main() {
      	vector<int> res(9,0),wh{ 1,2,3,4,5,6,7,8,9 };
      	adjacent_difference(wh.begin(), wh.end(), res.begin(),plus<int>());
      	for (int i = 0; i < 9; ++i)
      		cout << res[i] << " ";//输出1 3 5 7 9 11 13 15 17
      	cout << endl;
      	return 0;
      }
      ```

      注意，第一个元素必然会在输出里面，传入的函数的运算逻辑是binary_op(前一个数，后一个数)；

4. power（T n，int x，binaryoperator op）函数，其中的op默认为乘积函数。幂运算函数，其底层实现为**二进制运算**，参考leetcode50。

   大概思路为n^10，10的二进制表示为1010，所以n^10=n^8\*n^2，因此可以先计算n^2然后再乘上n^8即可；

5. iota（inputoperator first，inputiterator end，T value）函数，**在first到end的区间内填充value，value+1，value+2，...**

   ```c++
   #include<iostream>
   #include<vector>
   #include<algorithm>
   #include<numeric>
   #include<functional>
   using namespace std;
   int main() {
   	vector<int> wh2{ 9,8,7,6,5,4,3,2,1 }, wh1{ 1,2,3,4,5,6,7,8,9 },res(9,0);
   	iota(wh1.begin(), wh1.end(), 5);
   	for (int i = 0; i < 9; ++i)
   		cout << wh1[i] << " ";//输出5 6 7 8 9 10 11 12 13
   	return 0;
   }
   ```

4. max和min函数都有两个版本：

   ①template<class T>

   ​    const T& max（const T &a,const T &b）{

   ​		return a>b?a:b;

   }

   ②template<class T>

   ​    const T& max（const T &a,const T &b,compare op）{

   ​		return op(a,b)?a:b;

   }

5. **copy算法：**可以将输入区间[f,l)的元素copy到输出区间[r,r+(l-f))中。

   **特别注意区间重叠的情况：**

   1. result的尾部与（first，end）重叠
   2. result的头部与（first，end）重叠

   ![](img/stl5.png)

   3. 如果使用的数组是char[]，则不用在意这个问题。因为此时调用copy会调用memmove（**对输入区间进行复制，然后再返回输出空间**），重叠问题不会出现；如果使用的是deque则有可能会出现重叠区间覆盖的问题；如果使用vector又不会出现问题；**区别在于，vector和char[]使用的迭代器或者指针，都是C++自带的指针，会调用memmove保证不会出现问题；**而deque使用的是类似指针的迭代器，所以没有调用memmove，有可能会出现区间覆盖的问题；

   4. 当然，如果result头部与输入区间重叠，**我们可以考虑使用copy_backward，逆向复制**；

   5. copy函数不能生成新的元素，即使使用vector遇到空间不足也不会扩容，所以调用前要保证区间足够大；

   6. 对于一个类的**默认构造函数、拷贝构造函数、赋值运算符、析构函数，**如果有下列三种情况之一的：

      1. 显示定义；
      2. 类中包含非静态非POD类型的数据成员；
      3. 有基类；

      则这个类是一个non-trivial的类，就是不平凡的类；POD类型肯定都是平凡的；

      **对于平凡的类型，copy函数会直接调用memmove来提升效率；**

      **对于非平凡的类型，copy函数会调用其赋值运算符进行复制；**



6. **set相关算法**

   集合set，一共提供了四种算法：**并集（union）、交集（intersection）、差集（differebce）、对称差集（symmetric difference）**；

   1. 这四种算法均要求**区间内元素有序**，所以unordered_set不能用这四种算法啦，哈希表作为底层实现的都不能使用这四种算法；

      ```c++
      #include<iostream>
      #include<vector>
      #include<algorithm>
      #include<numeric>
      #include<functional>
      #include<set>
      using namespace std;
      template<class T>
      struct display {
      	void operator()(const T &a) {
      		cout << a << " ";
      	}
      };
      int main() {
      	int arr1[5] = { 1,3,5,7,9 }, arr2[5] = { 2,4,5,8,9 };
      	set<int> s1(arr1, arr1 + 5), s2(arr2, arr2 + 5);
      	for_each(s1.begin(), s1.end(), display<int>());
      	cout << endl;
      	for_each(s2.begin(), s2.end(), display<int>());
      	cout << endl;
      	vector<int> res(10, 0);
      	set_union(s1.begin(), s1.end(), s2.begin(), s2.end(), res.begin());
      	for_each(res.begin(), res.end(), display<int>());
      	cout << endl;//交集，输出1 2 3 4 5 7 8 9
      	fill(res.begin(), res.end(), 0);
      	set_intersection(s1.begin(), s1.end(), s2.begin(), s2.end(), res.begin());
      	for_each(res.begin(), res.end(), display<int>());
      	cout << endl;//并集，输出5 9
      	fill(res.begin(), res.end(), 0);
      	set_difference(s1.begin(), s1.end(), s2.begin(), s2.end(), res.begin());
      	for_each(res.begin(), res.end(), display<int>());
      	cout << endl;//差集，注意这里是s1-s2，输出1 3 7
      	fill(res.begin(), res.end(), 0);
      	set_difference(s2.begin(), s2.end(), s1.begin(), s1.end(), res.begin());
      	for_each(res.begin(), res.end(), display<int>());
      	cout << endl;//差集，注意这里是s2-s1，输出2 4 8
      	fill(res.begin(), res.end(), 0);
      	set_symmetric_difference(s1.begin(), s1.end(), s2.begin(), s2.end(), res.begin());
      	for_each(res.begin(), res.end(), display<int>());
      	cout << endl;//对称差集，就是两个差集的并集，输出1 2 3 4 7 8
      	return 0;
      }
      ```

   2. set_union

      求出两个集合的并集。**注意，其实这个算法可以用于multiset，也就是集合中元素可能有重复；**例如s1中有m个1，s2中有n个1，那么并集中有max（m，n）个1；是一个稳定的算法；

      其底层实现值得学习：首先s1和s2都是有序的，声明两个迭代器指向开头，只要两个迭代器都不为空则执行循环，循环有三种情况：

      ①s1的元素比较小，则此元素加入res，然后迭代器++；

      ②s2的元素比较小，则此元素加入res，然后迭代器++；

      ③s1和s2的元素相等，取s1中的元素加入res，然后两个迭代器++；

      当跳出循环的时候，有一方不为空，则将剩余的元素全部copy到res中；

   3. set_intersection

      两个集合的交集。**注意，其实这个算法可以用于multiset，也就是集合中元素可能有重复；**例如s1中有m个1，s2中有n个1，那么交集中有min（m，n）个1；是一个稳定的算法；

      这个的底层实现就很简单了，首先s1和s2都是有序的，声明两个迭代器指向开头，只要两个迭代器都不为空则执行循环，循环有三种情况：

      ①s1的元素比较小，直接迭代器++；

      ②s2的元素比较小，直接迭代器++；

      ③s1和s2的元素相等，取s1中的元素加入res，然后两个迭代器++；

      当跳出循环的时候，将剩余的元素全部丢弃；

   4. set_difference

      两个集合的差集，即s1中出现的，但是s2中没有的元素集合。**注意，其实这个算法可以用于multiset，也就是集合中元素可能有重复；**例如s1中有m个1，s2中有n个1，那么并集中有max（m-n，0）个1；是一个稳定的算法；

      首先s1和s2都是有序的，声明两个迭代器指向开头，只要两个迭代器都不为空则执行循环，循环有三种情况：

      ①s1的元素比较小，则此元素加入res，然后迭代器++；

      ②s2的元素比较小，直接迭代器++；

      ③s1和s2的元素相等，直接两个迭代器++；

      当跳出循环的时候，有s1不为空，则将剩余的元素全部copy到res中；

   5. set_symmetric_difference

      两个集合的对称差集，即s1中出现的，但是s2中没有的元素，同时s2中出现，但是s1中没有的元素的集合。**注意，其实这个算法可以用于multiset，也就是集合中元素可能有重复；**例如s1中有m个1，s2中有n个1，那么并集中有abs（m-n）个1；是一个稳定的算法；

      首先s1和s2都是有序的，声明两个迭代器指向开头，只要两个迭代器都不为空则执行循环，循环有三种情况：

      ①s1的元素比较小，则此元素加入res，然后迭代器++；

      ②s2的元素比较小，则次元素加入res，然后迭代器++；

      ③s1和s2的元素相等，直接两个迭代器++；

      当跳出循环的时候，有一个不为空，则将剩余的元素全部copy到res中；

7. heap相关算法：make_heap、push_heap、pop_heap、sort_heap；

8. 其他算法：

   一般都是进行单纯的线性移动进行数据的统计和判断等；例如find、count、for_each、remove、reverse、rotate、max_element、min_element等；

   1. 如果vec是一个vector，max_element（vec.begin(),vec.end()）会返回**一个迭代器**，所以要使用cout<<\*max_element（vec.begin(),vec.end()）<<endl;才可以正常输出最大元素的下标；

   2. remove和unique都是**只移除但是不删除的函数**，remove（first，end，value）移除与value相等的元素；unique（first，end）移除重复的元素；

   3. replace（first，end，oldvalue，newvalue）将oldvalue全部替换为newvalue；

   4. replace_if（first，end，op，newvalue），如果op（i）为true，则替换为newvalue；

   5. reverse（first，end）将所有元素翻转；

   6. rotate（first，middle，end）**将[first，middle)的元素全部移动到end的后面**，此时middle变成了这个容器的第一个元素；

      这个的底层实现还是很酷炫的：

      ```c++
      #include<iostream>
      #include<algorithm>
      #include<string>
      using namespace std;
      void strratate(int f, int m, int l, string &s) {
      	if (f == l || f == l - 1)
      		return;
      	for (int i = m;;) {
      		swap(s[i], s[f]);
      		f++, i++;
      		if (f == m) {
      			if (i == l)
      				return;
      			m = i;
      		}
      		else if (i == l)
      			i = m;
      	}
      }
      int main() {
      	string str("ABCDEFGH");
      	strratate(0, 4, str.size(),str);
      	cout << str << endl;
      	return 0;
      }
      ```

      还有一个更酷炫的：

      reverse(s[first], s[mid]);

      reverse(s[mid], s[last]);

      reverse(s[first], s[end]);

   7. search（first1，end1，first2，end2）在first1到end1中寻找完全匹配first2到end2的子序列的开头的迭代器，如果没有则返回end1；

      这里居然不是KMP算法，而是最普通的不断回溯的暴力算法；

   8. random_shuufle（first，end）将first到end的元素全部打乱，**要求支持随机访问**；

      其实现还是挺有趣的：

      ![](img/stl6.png)

   9. sort（first，end，op）将first到end的元素按照op的规则排序，**只支持能够随机访问的那些容器**，很多STL容器都有自己的sort成员函数，红黑树那些肯定用自带的；链表不支持随机访问，stack、queue都是由限制的不允许排序，所以就用在vector中多点；

      1. **底层实现是快速排序，**快速排序会不断缩小排序范围，当排序范围小于一个阈值，**就会使用直接插入排序；**此外，如果递归层次太深，还会使用**堆排序**；

      2. 在STL中的sort函数使用**三点取中来选择枢轴**；

      3. 使用直接插入排序的**阈值**因设备而异，一般就取数组长度为n，找到一个2^k<=n的最大的k，这个k就是进行直接插入排序的阈值；

      4. 总结一下：

         ①开始sort之前，会先确定一个直接插入排序**阈值**；

         ②使用**三数取中**找到一个合适的枢轴，进行一次快排，分割成两半继续排序；

         ③如果长度小于阈值，则进入直接插入排序；

         ④如果递归深度大于**2\*阈值**，就调用堆排序；（也就是introsort了）

   10. nth_element（first，nth，end），一次快排，使得first到nth前面的元素全部小于nth到end之间的元素，当然可以加一个op作为第四个参数来指定比较方法；

       其底层实现是不断调用快排的partition，根据返回的分割点与nth进行比较，不断缩小排序范围，直到排序到达分割点；



## 仿函数

