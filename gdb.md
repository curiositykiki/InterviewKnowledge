### linux程序员必须会用gdb进行程序的调试（一般命令后面要加空格）

1. 想要对某个程序进行gdb调试，编译的时候要加上-g，例如g++ -g -o test；
2. 如果是已经编译好的程序我怎么知道是否可以gdb？使用readelf -S 程序名|grep debug如果有东西就说明加入了-g，否则就不能gdb；readelf -h 程序名可以看看elf文件的头部信息，例如在什么机器上运行等信息；
3. 使用"gdb 程序名"启动gdb，例如gdb test，即启动gdb对test程序进行debug；
4. 在gdb中，**直接回车，表示再次执行上一次的命令**；
5. 使用list（缩写为l）查看当前程序的源代码；使用list+num查看第num行前10行和后10行；使用list+func查看函数func的源代码；
6. 怎么设置断点呢？使用break（缩写为b） + num表示在第num设置断点；使用b+func表示在func（）的入口处设置断点；使用b fn1 if a>b表示在fn1处如果a大于b就断开，否则不断开；
7. 那么怎么查看我们设置了多少断点分别在哪？使用info b查看设置的所有断点的位置和状态；使用delete 断点号n表示删除断点n；使用disable 断点号n表示关闭第n个断点，但是不删除；使用enable 断点号n表示启动断点n；delete breakpoints表示删除所有断点；
8. 进入gdb之后就是交互模式了，使用start从第一行运行，使用run直接运行到断点break；
9. 使用continue（缩写为c），可以继续执行到下一个断点；
10. 使用step（s）单步调试，如果是函数调用，则**进入函数**；
11. 使用next（n）单步调试，如果是函数调用，**不进入函数**；
12. 使用until跳出循环体，使用until+num直接运行到第num行；
13. call func（a）表示直接调用func函数，**这里的调用是会影响到程序的后续执行的**；
14. quit（q）退出调试；
15. 在gdb里面，可以随时查看变量的值及其类型，但是注意，如果编译的时候加上了-O3表示3级优化（最高级优化），有些变量会显示<optimaized out>即优化之后看不到了，那就只能降低优化级别或者别的方法（我也暂时不知道）；使用print + a表示显示a的值，可以是a+b这样的表达式；
16. 使用display a表示每一步运行之后都把a打印出来看看；
17. watch a表示监视a，如果a变化了，就终止程序；
18. whatis a表示查询a的类型，相当于typeof；
19. info function表示查询函数function；
20. 运行时的信息怎么查询呢？使用where或者bt（backtrace）查看当前的调用栈，目前在#0层，还会表示在哪一行进行调用的；info program表示看看program是否在运行以及进程号，暂停原因；
21. layout表示分割窗口，一遍看代码一遍测试，感觉好鸡肋啊；
22. ctrl+L刷新窗口；

23. 使用info threads可以查看当前进程中有多少个线程；然后使用thread num可以切换到num线程进行进一步调试；