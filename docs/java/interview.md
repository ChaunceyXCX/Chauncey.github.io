# java面试题整理

## 基础

1. 简单说下跨平台?

>由于不同操作系统之间所支持的指令集存在差异,所以在操作系统上加个虚拟机提供统一的接口来屏蔽系统之间的差异

2. Java8中基本数据类型?
  
| 数据类型 | 字节 | 默认值 | 包装类型 |
| -----   | --- | ----- | ----    |
| byte    | 1   |   0   | Byte    |
| short   | 2   |   0   | Short   |
| int     | 4   |   0   | Integer |
| long    | 8   |   0   | Long    |
| float   | 4   | 0.0f  | Float   |
| double  | 8   | 0.0d   | Double |
| char    | 2   |'\u0000'| Character|
| boolean | 4   |false   | Boolean |

- 自动装箱与自动拆箱:
  > 自动装箱: new Integer(1),底层调用:Integer.valueOf(1),得到的是一个对象
  > 自动拆箱: int i = new Integer(1) ; 底层调用的是i.intValue(); 得到的是int
- 基本数据类型与包装类型的区别:
  1. 声明方式不同;
  2. 存储方式以及存储位置不同:基本数据类型存储在栈中,而包装类型存储在
  3. 初始值不同:包装类型的初始值为null

1. ==和equals的区别?

> == 比较的是两个引用在内存中是不是同一个对象
> equals用来比较某些特征,String的equals是重写后的,实现比较两个对象内容是否相等

4. String/StringBuffer/StringBuilder的区别
   1. 数据可变和不可变:
       - String底层使用一个不可变字符数组:private final char value[]; 所以它的内容不可变;
       - StringBuffer和StringBuilder都继承了AbstractStringBuilder底层使用的是可变数组:char[] value;
   2. 线程安全:
        - StringBuilder是线程不安全的,效率极高;而StringBuffer是线程不安全的,效率极低,StringBuffer的append()方法有同步锁,而StringBuilder没有
    3. 相同点:
        - StringBuffer与StringBuilder有共同的父类:AbstractStringBuilder
    4. 操作可变字符串的速度: StringBuilder>StringBuffer>String

5. 说一下Java中的集合:
    - Collection下:List系(有序,元素可重复),Set系(无序,元素不重复)
    >Set根据equals和hashcode判断重复,一个对象要存入set中必须重写equals和hashcode
    - Map下:HashMap线程不同步; TreeMap线程同步
    - Map系列是对Collection的补充两者没有任何关系

6. ArrayList和LinkedList区别?
   
    - ArrayList基于动态数组的数据结构,LinkedList基于链表的数据结构
    - 由于两者数据结构的区别:对于随机访问(get和set)ArrayList由于LinkedList(因为LinkedList要移动指针),对于新增和删除,LinkedList占优势,因为链表只需要连接相应指针

    ### ArrayList与LinkedList源码分析:
    [ArrayList源码解析](./ArrayListSource )

    [LinkedList源码解析](./LinkedListSource )

7. 集合中ConCurrentModificationException异常出现的原因
   ```java
   public void main (String[] args) {
       ArrayList<Integer> list = new ArayList<>();
       list.add(2);
       Iterator<Integer> iterator = list.iterator();
       while(iterator.hasNext()) {
           Integer i = iterator.next();
           if(i==2) {
               list.remove(i); //会导致modCount和expectedModCount的值不一致,抛出异常
           }
       }
   }

   //正确写法,调用Iterator的remove方法
   public static void main(String[] args)  {
        ArrayList<Integer> list = new ArrayList<Integer>();
        list.add(2);
        Iterator<Integer> iterator = list.iterator();
        while(iterator.hasNext()){
            Integer integer = iterator.next();
            if(integer==2)
                iterator.remove();   //注意这个地方
        }
    }
    ```

8. HashMap,HashTable,ConcurrentHashMap的区别

    - 相同点:
        1. HashMap和HashTable都实现了Map接口
        2. 都是key-value存储
    - 不同点:
        1. HashMap可以把null作为key或者value,HashTable不可以
        2. HashMap线程不安全,效率高,HashTable线程不安全,效率低
        3. HashMap的迭代器(Iterator)是fail-false迭代器,HashTable的(Enumerator)不是的
            >fail-false : 最快时间把错误抛出而不是让程序执行

    - 能否既保证线程安全又提高效率        
        - 使用ConCurrentHashMap,它是HashTable的替代,比HashTable的扩展性好
        - TODO
          -  [ ] 三者源码

    - 能否让HashMap同步?
    > HashMap可以通过以下语句同步: Map m = Collections.synchronizedMap(hashMap);

9. 拷贝文件的工具类使用的是字节流还是字符流?

    >使用的是:因为所有文件都是以二进制形式存在,考虑到通用性使用字节流

    - 字节流:传递字节(二进制)
    - 字符流:传递字符

10. 两个对象的hashCode相同,则equals也一定为true,对吗?
    
    >不对:因为对象可以重写hashCode()方法让其返回值相同
    - TODO
      - [ ] 非特意情况是否一定相同??

    >两个对象equals为true,则hashCode()也一定相同吗?
    > 答: 不相同,同理,不重写hashCode()的话相同,重写的话,就会出现不相等的情况

11. java线程池创建?
    >使用Executors,其提供了4种创建方式

    1. newFixedThreadPool():创建固定大小的线程池
    2. newCachedThreadPool():创建无限大小的线程池,线程池中数量不固定,可根据需要手动修改
    3. newSingleThreadPool():创建只有一个线程的线程池
    4. newScheduledThreadPool():创建固定大小的线程池,可以延时或定时执行任务

### 线程池作用
    
- 限制线程个数,避免线程过多导致系统运行缓慢或崩溃
- 避免频繁的创建和销毁,节约资源,响应更快

12. Math.round(-2.5)等于多少?
    >它不是四舍五入,口诀:原数加0.5后向下取整,所以等于-2
    >-2.6等于-2;2.6等于3
    



## 线程相关

1. 创建线程的方法:
    - 继承Thread类,重写run()方法;
    ```java
    public class MyThread extends Thread{
        @Override
        public void run() {
            while(! interrupted()) { // @@1
                //do sth
            }
        }

        public static void main(String[] args) {
            MyThread t1 = new MyThread("t1");
            t1.start();


            t1.interrupt(); //@@2 
            // 想要中断线程时最好结合 @@1 和 @@2 调用interrupt()方法实现
            // 禁止调用stop();方法,该方法不会释放占用的资源
        }
    }
    ```
    - 实现Runnable接口,重写run()方法
    ```java
    //Runnable只是用来修饰所执行的任务的,它不是一个线程对象,想要启动一个Runnable任务必须将它放到线程对象里面
    public class MyThread implements Runnable {
        @Override
        public void run() {
            //do sth
        }

        public static void main(String[] args) {
            Thread t1 = new Thread(new MyThread());
            t1.start();
        }
    }
    ```

    - 匿名内部类创建线程对象
    ```java
    public static void main(String[] args) {
        //创建无参数线程对象
        new Thread() {
            @Override
            public void run() {
                //do sth
            }
        }.start();

        //创建带线程任务的线程对象
        new Thread( new Runnable() {
            @Override
            public void run() {
                //do sth
            }
        }).start();
    }
    ```

    - 创建带返回值的线程
    ```java
    public class MyThread implements Callable {
        @Override
        public Object call() throws Exception {
            int result = 1;
            //do sth
            return result;
        }

        public static void main(String[] args) {
            MyThread t1 = new MyThread();
            FutureTask<Integer> task = new FutureTask<Integer>(t1);
            Thread thread = new Thread(task);
            thread.start();

            Integer result = task.get();
        }
    }
    ```
    - 定时器Timer创建
    ```java
    public void main(String[] args) {
        Timer timer = new Timer();
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                //do sth
            }
        },0,1000); //延迟0,周期1s
    }
    ```
    - 线程池创建线程
    ```java
    public static void main(String[] args) {
        //创建一个有十个线程的线程池
        ExecutorService threadPool = Executors.newFixedThreadPool(10);
        long threadPoolUseTime = System.currentTimeMillis();
        for (int i=0; i<10; i++) {
            threadPool.execute(new Runnable() {
                @Override
                public void run() {
                    //do sth
                }
            });
        }
        long threadPoolUseTime1 = System.currentTimeMills();
        //多线程用时
        threadPoolUserTime1-threadPoolUseTime;
        //销毁线程池
        threadPool.shutdown();
    }
    ```
    - 利用Java8新特性stream实现并发
    ```java
    public static void main(String[] args) {
        List<Integer> values = Arrays.asList(10,20,30,40);
        //parallel: 平行的,并行的
        //所以此行代表并发执行sum();
        int result = values.parallelStream().mapToInt(p->p*2).sum();

        //乱序输出说明是并发执行的
        values.parallelStream().forEach(p-> System.out.println(p));
    }
2. sleep() 与 wait() 的区别?

    >sleep()会让线程进入阻塞状态,时间结束继续运行(不会释放锁)
    >wait()会让线程释放锁进入等待队列,notify/notifyall后会进入就绪队列