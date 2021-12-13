## synchronized锁的内容

```java
import java.util.concurrent.TimeUnit;

class Test1 {

    public static void main(String[] args) {
        Phone phone = new Phone();

        new Thread(() -> {
            phone.sendMsg();
        }, "A").start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(() -> {
            phone.call();
        }, "B").start();

    }
}


class Phone {

    // 锁的是方法的调用者
    // 用的是同一个锁，谁先拿到谁执行
    public synchronized void sendMsg() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    public synchronized void call() {
        System.out.println("打电话");
    }
}
```

```java
import java.util.concurrent.TimeUnit;

class Test2 {
    public static void main(String[] args) {
        // 一个对象只有一把锁
        Phone2 phone = new Phone2();

        new Thread(() -> {
            phone.sendMsg();
        }, "A").start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(() -> {
            phone.hello();
        }, "B").start();

        // 先hello后发短信
    }
}


class Phone2 {
    // 锁的是方法的调用者
    // 用的是同一个锁，谁先拿到谁执行
    public synchronized void sendMsg() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    public synchronized void call() {
        System.out.println("打电话");
    }

    // 没有锁，不是同步方法，不受锁的影响
    public void hello() {
        System.out.println("hello");
    }
}
```

```java
import java.util.concurrent.TimeUnit;

class Test3 {
    public static void main(String[] args) {
        // 一个对象只有一把锁
        // 两个对象的Class类模板只有一个，static锁的是class
        Phone3 phone1 = new Phone3();
        Phone3 phone2 = new Phone3();

        new Thread(() -> {
            phone1.sendMsg();
        }, "A").start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(() -> {
            phone2.call();
        }, "B").start();

        // 先短信后打电话
    }
}

// 两个静态的锁
// 类一加载就有了，锁的是Class
class Phone3 {
    public static synchronized void sendMsg() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    public static synchronized void call() {
        System.out.println("打电话");
    }

}
```

```java
import java.util.concurrent.TimeUnit;

class Test4 {
    public static void main(String[] args) {
        // 一个对象只有一把锁
        // 两个对象的Class类模板只有一个，static锁的是class
        Phone4 phone1 = new Phone4();
        Phone4 phone2 = new Phone4();

        new Thread(() -> {
            phone1.sendMsg();
        }, "A").start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(() -> {
            phone2.call();
        }, "B").start();

        // 先打电话后发短信
    }
}

class Phone4 {

    // 锁的是Class
    public static synchronized void sendMsg() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    // 普通同步方法 锁的是调用者
    public synchronized void call() {
        System.out.println("打电话");
    }

}
```