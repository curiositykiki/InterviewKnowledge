### 小知识

1. **=和：=和？=和+=**

   ①=是最基本的赋值；

   ②：=是覆盖之前的赋值；

   ③？=是如果没有赋值就将这个值赋予；

   ④+=是添加等号后面的值；

   例子：

   x=foo

   y=$(x) bar

   x=xyz

   由于makefile会展开以后再决定变量的值，**变量的值会等于最后一次的赋值，所以这里的y是xyz bar**；

   x：=foo

   y：=$(x) bar

   x：=xyz

   使用：=那么变量的值会**等于最近一次的赋值，所以这里的y是foo bar；**

2. **$^和$<和$@**

   ①$^是所有依赖；

   ②$<是第一个依赖；

   ③$@表示目标集合；

   例子：

   OBJS = ifconfig.o tftp_util.o ping.o
   SRCS = $(OBJS:.o=.c)

   此时$^表示$(objs)；$<表示ifconfig.o；$@表示$(SRCS);



### GCC编程四个过程:预处理-编译-汇编-链接

1. 预处理

   编译器将**C源代码中包含的头文件如stdio.h编译进来**；

2. 编译

   编译器**检查代码的规范性，是否存在语法错误等，检查无误后，将其翻译成汇编语言；**

3. 汇编

   编译器将**编译阶段生成的.s文件转成二进制目标代码；**

4. 链接

   **将编译输出文件链接成可执行文件；**

   这里有一个点，即使函数中没有定义printf的实现，依然可以调用，是因为这里使用了**动态库**。

   函数库一般分为**静态库和动态库**。静态库是指编译链接时，将库文件中代码全部写入文件中，因此生成的文件较大，**好处是运行的时候不再需要库文件，后缀名一般为.a；**动态库与之相反，在编译链接的时候**没有**将具体代码写入文件，**而是在程序执行时由链接文件加载库函数，好处是节省系统的开销。后缀名一般为.so。**GCC在编译时默认使用动态库。

5. gcc使用的时候-c和-o的区别：

   ①**-c表示编译和汇编，但不要链接**；一般用于生成.o文件，全部生成完了之后再链接；

   ②**-o表示将输出写入后面的文件中**；一般就是生成可执行文件a的时候-o a；

   ③如果**什么都不加**会自动生成可执行文件；例如：gcc main.c会生成a.out可执行文件；

6. 具体请看https://www.cnblogs.com/qytan36/archive/2010/05/25/1743955.html

   

### makefile的规则

1. 形如：

   ​	target ... ：prerequisites ...

   ​				command

   ​				...

   ​				...

   target可以是**可执行文件、object文件、伪文件**；

   prerequisites就是要生成target**所需的文件**；

   command就是make要执行的**命令**；

2. 一个示例：

   ​	edit ： main.o  \

   ​				 kbd.o  

   ​				cc -o edit main.o kbd.o

   ​	

   ​	main.o ： main.c defs.h

   ​				cc -c main.c

   ​	kbd.o ：kbd.c defs.h command.h

   ​				cc -c kbd.c

   

   ​	clean ：

   ​				rm edit main.o kbd.o

   反斜杠表示换行；

   edit是我们最终生成的可执行文件，需要后面的.o文件；

   main和kbd分别都是.c和.h进行编译；

   clean是一个**伪文件**，相当于定义了一系列操作，**使用make clean来调用**；



### make是如何工作的

1. make会在**当前目录**下查找名为**Makefile或者makefile**的文件；
2. 如果找到，会将edit这个文件当成最终目标，因为在第一行第一个冒号前；
3. 如果edit**不存在、或者后面的.o文件比edit要新**，就会执行后面的指令来生成edit；
4. .o的文件规则一样，会递归生成；



### makefile中的变量

1. 在makefile中，**符号“${x}”表示x是一个变量**，可以在文件中定义它。例如：

   ​	objects = main.o kbd.o

   ​	edit ： ${objects}

   ​			cc -o edit ${objects}

   ​	跟上面的效果一样。

2. makefile的自动推导：我们编译一个x.o文件，makefile会自动寻找x.c来执行cc -c x.c操作，因此可以省略。例如：

   main.o ： main.c defs.h

   ​		cc -c main.c

   等价于

   main.o ： defs.h

3. 可以**显式声明clean为伪目标文件**，例如：

   .PHONY ： clean

   clean ： 

   ​		rm edit ${objects}

4. **rm和-rm**的区别，如果带有小横杠，表示的意思是，**如果发生错误，直接忽略，继续执行操作。**

5. makefile中的注释使用井号 \#；

6. makefile中的command前必须使用**制表符Tab**；



### 引用其他makefile

1. 类似于C语言，可以使用include来包含别的makefile，且可以包含**文件、变量或者通配符**例如：

   include  foo.make \*.mk ${bar}

   其中\*.mk表示当前目录中**所有后缀为.mk**的文件；

2. 如果需要到别的目录寻找包含文件，可以使用-I或者--include-dir；

3. 如果本目录没有找到，也没有定义某文件在哪个目录，会到**/usr/local/bin或/usr/include**中寻找；



### make的工作方式

1. 读入所有的makefile；
2. 读入include的makefile；
3. 初始化文件中的变量；
4. 推导隐晦规则，并分析所有规则；
5. 为所有目标文件创建依赖关系；
6. 根据依赖目标，决定哪些目标需要重新生成；
7. 执行生成命令；

其中1-5为**编译**，6-7为**链接**。



### 书写的规则

1. 一般来说，**make会以UNIX的标准shell，也就是说/bin/sh来执行命令**；

2. **通配符**：

   ①\*表示与任意内容适配；

   ②~表示当前用户目录，~aa表示用户aa的根目录；

   ③还有？和[...]；

3. 文件搜寻：指定路径让makefile去寻找依赖，例如

   VPATH = src ： ../headers

   **冒号“：”**表示分割，寻找会按照**前到后的顺序去寻找**。

4. 此外，vpath还可以**指定某种类型的文件在哪里找。**例如：

   ①vpath <pattern> <dir>：pattern这种类型的文件到dir去找，例如：

   ​		vpath  %.h  ../headers

   ​		这里的%匹配若干字符，表示后缀为.h的文件去../headers去找

   ②vpaht <pattern>：清除pattern这种类型文件的搜索目录；

   ③vpath ： 清除所有已设置的搜索目录

   可以使用多个vpath命令，会按照**先后顺序**进行查找。



### 伪目标.PHONY的使用

1. **一次生成多个可执行文件**：

   all ： p1 p2

   .PHONY  ： all

   p1 ： p1.o

   ​		cc -o p1 p1.o

   p2 ： p2.o

   ​		cc -o p2 p2.o

2. **生成多种不同操作**：

   .PHONY ： clean1 clean2

   clean1 ： 

   ​		rm p1

   clean2 ： 

   ​		rm \*.o

   使用make clean1和make clean2分别执行不同方法。



### 多目标

1. **多个目标依赖于同一个文件**，如何写：

   bigoutput littleoutput ： text.g

   ​		generate  text.g -$（substr output,,$@） > $@

   等价于

   bigoutput ： text.g

   ​		generate text.g -big > bigoutput

   littleoutput ： text.g

   ​		generate text.g -little > littleoutput

   这里的**$()**是一个**函数**，substr是函数名，output是参数，$@是目标的集合（这里就是指bigoutput和littleoutput）。

   **$(substr a,b,$@)的作用是，将$@目标集里面的所有元素的子串a替换为子串b，这里b为空，那就是删掉$@里面所有元素的a这个子串。**



### 静态模式

这个功能用于有多个文件需要编译，如何简洁表示的方式。例如：

files ： foo.elc bar.o lise.o

$(filter %.o,$(files))： %.o ： %.c

​			$(CC)  -c $(CFLAGS)  $< -o $@

$(filter %.elc,$(files))： %.elc ： %.el

​			emacs -f batch-byte-compile $<

相当于

files ： foo.elc bar.o lise.o

bar.o： bar.c

​			$(CC)  -c $(CFLAGS)  bar.c -o bar.o

lose.o： lose.c

​			$(CC)  -c $(CFLAGS)  lose.c -o lose.o

foo.elc ：foo.el

​			emacs -f batch-byte-compile foo.el

分析：

①第一行 targets ： pattern ： prereq中，

**targets就是我们需要的一些列最终目标；**

**pattern就是targets的模式，比如pattern是%.o就是指targets里面所有的.o文件；**

**prereq就是指对应的pattern里面的依赖，比如%.c就是指将pattern中的文件对应的依赖就是后缀不同的对应文件；**

filter是一个函数，后面的都是参数；

**$<指的是依赖集合的第一个依赖；**

**$@指的是目标集合里的目标；**



### 自动生成依赖性（很难理解）

在大型工程中，哪些c文件包含了哪些**头文件**，有时候很繁琐，所以可以通过makefile来自动生成**依赖关系。**例如：

cc -M main.c会输出main.o ： main.c  defs.h（**这里如果是gcc/g++要使用-MM**）

我们可以为**每一个c文件**都生成一个**存放依赖关系的d文件，这个d文件是一种makefile，当写最后的makefile的时候，可以include。**但是如果我们修改了文件以后，d文件也应该会自动更新，因此需要使用**自动生成依赖性**。

我们执行main.o ： main.c  defs.h的时候会生成main.o，如果我们执行main.o main.d： main.c  defs.h那就可以自动生成main.d了，请看下面：

%.d ： %.c

​		set -e；rm -f $@；\

​		$(CC) -M $(CPPFLAGS) $< > $@.$$$$; \

​		sed 's,\\($*\\)\\.o[ :]\*,\\1.o $@ :,g' < $@.$$$$ > $@;\

​		rm -f $@.$$$$;

①**因为这4行命令要多次凋用，定义成命令包以简化书写，所以后面都有反斜杠**；

②set -e是指**如果某个命令返回非0参数，立刻退出；**

③rm -f $@是指删除所有的目标文件，**-f表示遇到不存在的文件不报错**；

④sed ’s/a/b/g‘是指**将a的字段用b来替换，s表示match可以使正则表达式，g表示所有的a都用b替换，没有g的话只会替换第一个a；**此外，/可以用，来替代，可读性++；

⑤>表示**重定向输出**，比如$(CC) -M $(CPPFLAGS) $< **>** $@.$$$$就是将$(CC) -M $(CPPFLAGS) $<生成的信息重定向输出到$@.$$$$文件中，这里的$$$$表示随机数字；

⑥<也是**重定向读入**，比如sed 's,\\($*\\)\\.o[ :]\*,\\1.o $@ :,g' < $@.$$$$ > $@表示从文件$@.$$$$重定向读入的信息经过sed替换后重定向输出到$@中；

⑦在sed 's,\\($*\\)\\.o[ :]\*,\\1.o $@ :,g'中，\\($\*\\)的好处在于，可以用\\1来引用括号内的内容；[ :]\*表示若干个空格和冒号；这个替换的意思就是**将xxx.o：这样的内容替换为xxx.o xxx.d：**，符合我们的要求；

⑧最后rm删掉所有的临时文件；

⑨@echo src=$(src)表示在调试的时候显示一些值

⑩为了能够在C++程序里调用C函数，**必须把每一个要调用的C函数，其声明都包括在extern "C"{}块里面**，这样C++链接时才能成功链接它们。 

更多相关https://blog.csdn.net/huyansoft/article/details/8924624



