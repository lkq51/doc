##浅谈JVM GC
其实第一门使用GC技术的是Lisp语言（Lisp也是第一门函数式编程语言，现在AI的主力编程语言，因其（）之多名扬天下。 《黑客与画家》的作者Paul Graham在书中大力推广这门语言，感兴趣的同学可以去了解下）

### 判断对象是否存活的方式

#### 1.引用计数算法
这是一种非常简洁明了，为人熟知的方法。简单地说

> 给对象添加一个引用计数器，当有一个地方引用了它。计数器值就加一；引用失效时就减一。任何时候只要计数器的值为0，就是对象不可使用了。

但这种方式有一个缺点就是无法解决对象间相互引用的问题。
	
	public class ReferenceCountingGC {

		public Object instance = null;
		private static final int _1MB = 1024 * 1024;

		private byte[] bigSize = new Byte[2 * _1MB];

		public static void testGC(){
			ReferenceCountingGC objA = new ReferenceCountingGC();
			ReferenceCountingGC objB = new ReferenceCountingGC();
			objA.instance = objB;
			objB.instance = objA;

			objA = null;
			objB = null;

			System.gc();
		}
	
	}


虽然现在的JVM都不采用这种方式进行垃圾回收，但Redis还是通过这种方式实现内存回收。
#### 2.可达性分析算法
在主流的商用语言中java，C#，Lisp都是通过可到达性分析来判断对象是否存活的。
通过"GC Roots"对象作为起始点，从这些节点开始向下搜索，所走过的路称为引用链，当一个对象到GC Roots没有任何引用链相连时。（图论中称为GC Roots到这个对象不可达）则证明这个对象时不可用的。

在java中 GC Roots可以是以下几种：

1. 虚拟机栈（栈帧中的本地变量表）中引用的对象

2. 方法区中类静态属性引用的对象

3. 方法区中常量引用的对象

4. 本地方法栈中JNI（即Native方法）引用的对象



### 引用
jdk1.2之后对引用进行了分类

1. 强引用

2. 软引用

3. 弱引用

4. 虚引用

引用强度依次减弱


强引用：
>类似Object Object = new Object()
>只要引用存在 垃圾收集器就永远不会回收该对象

软引用：
>有用但不是必需的对象。在系统发生内存溢出异常之前，会把这些对象列进回收范围之内进行二次回收，如果这些对象回收了还是没有足够的内存，才会抛出内存溢出异常

弱引用：
>当垃圾收集器工作时，无论内存是否足够都会回收弱引用的对象

虚引用：
>无法通过虚引用来获取一个对象，也不会对对象存活时间造成任何影响。只会在被回收的时候得到一个系统的通知








    

    
