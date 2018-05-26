# 深入浅出Java多线程--慕课网

### 1、进程、线程概念



#### 进程

程序（任务）的执行过程

举个例子：当我们运行qq这个可执行文件的时候，就变成了一个进程

持有资源（例如：共享内存，共享文件）和线程。也就是说进程是线程的载体，脱离了进程去谈进程是没有任何意义的

#### 线程

当qq进程在跑的时候，我们可以利用qq来进行文字的聊天、收发文件等等，所有的这些任务我们都可以理解为线程

举个生活中的例子：

我们可以把进程比作一个班级，那么班级中的每一个学生就可以看作是一个线程。在这里，学生是班级中的最小单元，构成了班级中的最小单位。一个班级可以有多个学生，这些学生都是用班级中不同的座椅，板凳等等来进行学习生活。在这里我们说：

> 1、线程是系统中最小的执行单元
>
> 2、同一个进程中有多个线程
>
> 3、线程共享进程的资源

##### 线程的交互

互斥同步

### 2、Java中线程的常用方法

![](http://oklbfi1yj.bkt.clouddn.com/%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BAJava%E5%A4%9A%E7%BA%BF%E7%A8%8B--%E6%85%95%E8%AF%BE%E7%BD%91/1.PNG)

#### Thread常用方法

![](http://oklbfi1yj.bkt.clouddn.com/%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BAJava%E5%A4%9A%E7%BA%BF%E7%A8%8B--%E6%85%95%E8%AF%BE%E7%BD%91/2.PNG)

其中`yield()`方法在释放完处理器资源之后，会**重新竞争处理器资源**，而不是直接让给其他线程

### 3、使用线程来模拟隋唐演义故事

#### 主要的思路

使用Runnable对象，使得轻而易举的隔离双方的差异，使得他们为“两个部队”

![](http://oklbfi1yj.bkt.clouddn.com/%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BAJava%E5%A4%9A%E7%BA%BF%E7%A8%8B--%E6%85%95%E8%AF%BE%E7%BD%91/3.PNG)

另一方面，历史是人民群众创造的，但是英雄人物可以推动历史的发展。因此，有一个关键人物的线程（KeyPersonThread）来模拟这种英雄人物

![](http://oklbfi1yj.bkt.clouddn.com/%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BAJava%E5%A4%9A%E7%BA%BF%E7%A8%8B--%E6%85%95%E8%AF%BE%E7%BD%91/4.PNG)

最后，我们需要一个舞台，来将我们的所有元素融入进历史故事之中，展现浩瀚的历史场面

![](http://oklbfi1yj.bkt.clouddn.com/%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BAJava%E5%A4%9A%E7%BA%BF%E7%A8%8B--%E6%85%95%E8%AF%BE%E7%BD%91/5.PNG)

所以这个故事有三个对象：军队线程（ArmyRunnable）、英雄人物线程（KeyPersonThread）、舞台线程（Stage）

此时只有一个演员线程：

```java
class Actor extends Thread {
	public void run() {
		System.out.println(getName() + "是一个演员");
		int count = 0;
		boolean keepRunning = true;

		while (keepRunning) {
		    System.out.println(getName() + "登台演出" + (++count) + "次");

		    if (100 == count) {
		    	break;
		    }

		    if ((count % 10) == 0) {
		    	try {
		    		Thread.sleep(1000);

		    	} catch (InterruptedException e) {
		    		e.printStackTrace();
		    	}
		    }
		}

		System.out.println(getName() + "的演出结束了！");
	}

	public static void main(String[] args) {
		Thread actor = new Actor();
		actor.setName("Mr.Thread"); // 设置线程的名字

		actor.start();
	}
}
```

此时再加入一个演员线程：

```java
class Actor extends Thread {
	public void run() {
		System.out.println(getName() + "是一个演员");
		int count = 0;
		boolean keepRunning = true;

		while (keepRunning) {
		    System.out.println(getName() + "登台演出" + (++count) + "次");

		    if (100 == count) {
		    	break;
		    }

		    if ((count % 10) == 0) {
		    	try {
		    		Thread.sleep(1000);

		    	} catch (InterruptedException e) {
		    		e.printStackTrace();
		    	}
		    }
		}

		System.out.println(getName() + "的演出结束了！");
	}

	public static void main(String[] args) {
		Thread actor = new Actor();
		actor.setName("Mr.Thread"); // 设置线程的名字

		actor.start();

		Thread actressThread = new Thread(new Actress(), "Ms.Runnable");
		actressThread.start();
	}
}

class Actress implements Runnable {
    public void run() {
		System.out.println(Thread.currentThread().getName() + "是一个演员");
		int count = 0;
		boolean keepRunning = true;

		while (keepRunning) {
		    System.out.println(Thread.currentThread().getName() + "登台演出" + (++count) + "次");

		    if (100 == count) {
		    	break;
		    }

		    if ((count % 10) == 0) {
		    	try {
		    		Thread.sleep(1000);

		    	} catch (InterruptedException e) {
		    		e.printStackTrace();
		    	}
		    }
		}

		System.out.println(Thread.currentThread().getName() + "的演出结束了！");
	}
}
```

因为`Runnable`没有`setName()`方法，所以，需要先调用`Thread.currentThread()`方法来**获取当前的线程的引用**，然后再来调用`setName()`方法来获取演员线程的名字

我们会发现，两个演员他们会穿插着登场（但是没有规定一定有顺序）

因为，作为计算机处理器，在同一时间同一个处理器（或者说同一个核）只能运行一个线程。当一个线程释放了处理器之后，另一个线程才占有了该处理器。这也是两个线程穿插的原因

现在，创建军队：

```java
/*
军队线程
模拟双方的行为
*/
class ArmyRunnable implements Runnable {
	// volatile 保证了线程可以正确的读取其他线程写入的值
	// 如果没有volatile，由于可见性的问题，当前线程可能不能正确读到keepRunning这个值
    volatile boolean keepRunning = true;

	public void run() {
		while (keepRunning) {
		    // 发动5连击
            for (int i = 0; i < 5; i++) {
            	System.out.println(Thread.currentThread().getName() + "进攻对方" + i + "次");
            }
            // 让出了处理器时间，下次该谁进攻还不一定
            Thread.yield();
		}

		System.out.println(Thread.currentThread().getName() + "结束了战斗！");
	}
}
```

然后在创建一个舞台，让两军开始厮杀：

```java
/*
隋唐演义大戏舞台
*/
public class Stage extends Thread {
    public void run() {
    	ArmyRunnable armyTaskOfSuiDynasty = new ArmyRunnable();
    	ArmyRunnable armyTaskOfRevolt = new ArmyRunnable();

    	// 使用Runnable接口来创建军队线程
    	Thread armyOfSuiDynasty = new Thread(armyTaskOfSuiDynasty, "隋军");
    	Thread armyOfRevolt = new Thread(armyTaskOfRevolt, "农民起义军");

    	// 调用线程对象的start()方法，启动军队线程，让军队开始作战
    	armyOfSuiDynasty.start();
    	armyOfRevolt.start();

    	// 舞台线程的休眠，大家专心观看军队的厮杀
    	try {
    		/*
            休眠的越久，就战斗的越久，执行到keepRunning = false的时间就越长。反之越短。
            因为keepRunning的值是在Stage线程里面修改的
            在休眠的时候，Stage线程会让出处理器。此时军队线程会继续抢占处理器。因为此时的keepRunning还是true
    		*/
    		Thread.sleep(50);

    	} catch (InterruptedException e) {
    		e.printStackTrace();
    	}

    	armyTaskOfSuiDynasty.keepRunning = false;
    	armyTaskOfRevolt.keepRunning = false;
    }

	public static void main(String[] args) {
		Stage stage = new Stage();
		stage.start();
	}
}
```

此时，我们加入英雄人物：

```java
class KeyPersonThread extends Thread {
	public void fun() { // 重写Thread的run方法
		System.out.println(Thread.currentThread().getName() + "开始了战斗！");
		for (int i = 0; i < 10; i++) {
			System.out.println(Thread.currentThread().getName() + "左突右杀，攻击隋军...");
		}

		System.out.println(Thread.currentThread().getName() + "结束了战斗！");
	}
}
```

把英雄加入舞台中进行演出：

```java
/*
隋唐演义大戏舞台
*/
public class Stage extends Thread {
    public void run() {
    	ArmyRunnable armyTaskOfSuiDynasty = new ArmyRunnable();
    	ArmyRunnable armyTaskOfRevolt = new ArmyRunnable();

    	// 使用Runnable接口来创建军队线程
    	Thread armyOfSuiDynasty = new Thread(armyTaskOfSuiDynasty, "隋军");
    	Thread armyOfRevolt = new Thread(armyTaskOfRevolt, "农民起义军");

    	// 调用线程对象的start()方法，启动军队线程，让军队开始作战
    	armyOfSuiDynasty.start();
    	armyOfRevolt.start();

    	// 舞台线程的休眠，大家专心观看军队的厮杀
    	try {
    		/*
            休眠的越久，就战斗的越久，执行到keepRunning = false的时间就越长。反之越短。
            因为keepRunning的值是在Stage线程里面修改的
            在休眠的时候，Stage线程会让出处理器。此时军队线程会继续抢占处理器。因为此时的keepRunning还是true
    		*/
    		Thread.sleep(50);

    	} catch (InterruptedException e) {
    		e.printStackTrace();
    	}

    	System.out.println("正当双方激战正酣，半路杀出了个程咬金");

    	Thread mrCheng = new KeyPersonThread();
    	mrCheng.setName("程咬金");

    	System.out.println("程咬金的理想就是结束战争，是百姓安居乐业！");

    	armyTaskOfSuiDynasty.keepRunning = false;
    	armyTaskOfRevolt.keepRunning = false;

        try {
    		Thread.sleep(2000);
    	} catch (InterruptedException e) {
    		e.printStackTrace();
    	}

    	/*
        *历史大戏留给关键人物
    	*/
    	mrCheng.start();

    	/*
        *万众瞩目，所有线程等待程先生完成历史使命
    	*/
    	try {
    		mrCheng.join(); //如果没有使用这句，那么，当英雄线程执行的过程中，退出了对处理器的占有后，Stage线程会占有处理器，此时，可能会执行后面的那两条战争结束和谢谢观看的打印语句。之后，英雄线程才会继续执行，参加战斗
    	} catch (InterruptedException e) {
    		e.printStackTrace();
    	}

    	System.out.println("战争结束，人民安居乐业，程先生实现了积极的人生梦想，为人民作出了贡献");
    	System.out.println("谢谢观看隋唐演义，再见！");
    }

	public static void main(String[] args) {
		Stage stage = new Stage();
		stage.start();
	}
}
```















































































