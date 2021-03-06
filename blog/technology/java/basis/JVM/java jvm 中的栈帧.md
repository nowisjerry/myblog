# java jvm 中的栈帧

### **栈帧数据结构**

栈帧(Stack Frame)是用来支持虚拟机进行方法调用和方法执行的数据结构，它是虚拟机运行时数据区中的虚拟机栈的栈元素。

栈帧(Stack Frame)存储了**方法的局部变量表、操作数栈、动态连接、和方法返回地址、额外的附加信息**。

每个方法在执行的同时，都会创建一个栈帧(Stack Frame)。每一个方法从调用开始至执行完成的过程，都对应着一个栈帧在虚拟机栈里面从入栈到出栈的过程。

### **栈帧的内存分配**

在编译程序代码的时候，栈帧中需要多大的局部变量表，多深的操作数栈都已经完全确定了，并且写入到方法表的Code属性中了，因此一个栈帧需要分配多少内存，不会受到程序运行期变量数据的影响，而仅仅取决于具体虚拟机的实现。

### **当前栈帧(Current Stack Frame)**

一个线程中的方法调用链可能会很长，很多方法都同时处于执行状态。对于执行引擎来说，在活动线程中，只有位于栈顶的栈帧才是最有效的,称为当前栈帧(Current Stack Frame),与这个栈帧相关联的方法称为当前方法。执行引擎运行的所有的字节码指令都只针对当前栈帧进行操作。在概念模型上，典型的栈帧结构图如下：

### **局部变量表**

局部变量表(Local Variable Table)是一组变量值存贮空间，用于存放方法参数和方法内定义的局部变量。在Java程序编译为Class文件时候，就在方法的Code属性的max_locals数据项中确定了该方法所需要分配的局部变量表的最大容量。单位为Slot

局部变量表的容量以变量槽(Variable Slot)为最小单位。每个变量槽都可以存储32位长度的内存空间，例如boolean、byte、char、short、int、float、reference。

对于64位长度的数据类型（long，double），虚拟机会以高位对齐方式为其分配两个连续的Slot空间，也就是相当于把一次long和double数据类型读写分割成为两次32位读写。

### **整型**

**整型字节(b)bite(位)**byte11*8short22*8int44*8long88*8

### **浮点型**

**浮点型字节(b)bite(位)**float44*8double88*8

### **Char类型**

**char类型字节(b)bite(位)**char22*8

### **布尔型**

**布尔型字节(b)bite(位)**boolean11*8

其中:8bit=1b [一个Byte等于8个bit（位）]，1024b=1kb

为了节省栈帧空间，局部变量表中的Slot是可以重用的，方法体中定义的变量，其作用域并不一定会覆盖整个方法体，如果当前字节码PC计数器的值已经超过了某个变量的作用域，那么这个变量对应的Slot就可以交给其他变量使用。
优点 ： 节省栈帧空间。
缺点 ： 影响到系统的垃圾收集行为。（如大方法占用较多的Slot，执行完该方法的作用域后没有对Slot赋值或者清空设置null值，垃圾回收器便不能及时的回收该内存。）

### **操作数栈**

操作数栈(operand Stack)也常称为操作栈，它是一个后入先出栈。和局部变量表一样，操作数栈的最大深度也在编译的时候写入到Code属性的max_stacks中。

操作数栈的每一个元素可用是任意的Java数据类型，包括long和double。32位数据类型所占的栈容量为1，64位数据类型占用的栈容量为2。

当一个方法刚刚开始执行的时候，这个方法的操作数栈是空的，在方法执行的过程中，会有各种字节码指令往操作数栈中写入和提取内容，也就是出栈 / 入栈操作。例如，在做算术运算的时候是通过操作数栈来进行的，又或者在调用其它方法的时候是通过操作数栈来进行参数传递的。

### **动态连接**

每个栈帧都包含一个指向运行时常量池中该栈帧所属方法的引用，持有这个引用是为了支持方法调用过程中的动态连接（Dynamic Linking）。

Class文件的常量池中存在大量符号引用，字节码中的方法调用指令就以常量池中指向方法的符号引用作为参数，这些符号引用一部分在类加载阶段中的解析阶段会转为直接引用，这种转化也称为静态解析。另外的一部分将在每一次运行时期转化为直接引用。这部分称为动态连接。

### **方法返回地址**

当一个方法开始执行后，只有2种方式可以退出这个方法 ：

方法返回指令 ： 执行引擎遇到一个方法返回的字节码指令，这时候有可能会有返回值传递给上层的方法调用者，这种退出方式称为正常完成出口。

异常退出 ： 在方法执行过程中遇到了异常，并且没有处理这个异常，就会导致方法退出。

无论采用任何退出方式，在方法退出之后，都需要返回到方法被调用的位置，程序才能继续执行，方法返回时可能需要在栈帧中保存一些信。用来帮助恢复它的上层方法的执行状态。

方法退出的过程实际上就等于把当前栈帧出栈，因此退出可能执行的操作有：
1.恢复上层方法的局部变量表和操作数栈
2.把返回值(如果存在返回值)压入调用者栈帧的操作数栈中
3.调整PC计数器的值以指向方法调用指令后面的一条指令

### **附加信息**

虚拟机规范允许具体的虚拟机实现增强一些规范里没有描述的信息到栈帧之中，例如与调试相关的信息，这部分信息取决于具体的虚拟机实现。

 



https://zhuanlan.zhihu.com/p/76862224