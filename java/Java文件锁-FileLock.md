# Java文件锁-FileLock

### Java FileLock（文件锁）

`FileLock`通过`FileChannel`对象获取，简单示例如下：
```java
try (RandomAccessFile stream = new RandomAccessFile("d:/test-lock", "rw");
    FileChannel channel = stream.getChannel()) {

    // 获取锁。如果有其他进程已经获得锁，则此 channel.lock() 被挂起，直至成功获取锁。
    FileLock lock = channel.lock();

    try {
        // 逻辑代码
        System.out.println("locked...");
        TimeUnit.SECONDS.sleep(2);

    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        lock.release();
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

### `FileChannel`获得锁有两类方法：
* `channel.lock()` - 如果锁被其他进程占有，则会一直等待其他进程释放锁
* `channel.tryLock()` - 如果锁被其他进程占有，会立即返回`null`，并不会一直等待锁的释放


### `FileLock`有两种类型：

* 排他锁 - `channel.lock()`/`channel.lock(0, Integer.MAX_VALUE, false)`
    * 排他锁需要`WritableChannel`，`new RandomAccessFile(file, "rw")`/`new FileOutputStream(file)` 可行，
    `new RandomAccessFile(file, "r")`/`new FileInputStream(file)` 会报错。
    * 当有进程获得`排他锁`后，另一个进程尝试读此文件会报错。在Windows系统下，使用Notepad打开时，不会报错，
    但你会发现打开的是一个空文件。
    * 当有进程获得`排他锁`后，另一个进程尝试获取`排他锁`或`共享锁`都将会被阻塞，直至`排他锁`释放。
    
* 共享锁 - `channel.lock(0, Integer.MAX_VALUE, true)`
    * 共享锁需要`ReadableChannel`，`new FileInputStream(file)`/`new RandomAccessFile(file, "rw")`/`new RandomAccessFile(file, "r")` 可行，
    `new FileOutputStream(file)`会报错。
    * 当有进程获得`共享锁`后，另一个进程可以读文件内容，但尝试写文件时会报错（就算`当前`获得锁的线程执行写操作也会`报错`）。
    在Windows系统中，使用Notepad打开文件后，可以看到文件内容，尝试写数据时不会报错，但保存文件后，你再次打开文件发现内容为空。
    * 当有进程获得`共享锁`后，另一个进程尝试获取`排他锁`将会被阻塞，直至`共享锁`释放。当另一个进程尝试获取`共享锁`时将会成功。
    * 第一个参数`position`：锁的起始位置
    * 第二个参数`size`：锁的区域大小，即同一个文件根据锁的不同区域可同时出现多个锁
    * 第三个参数`shared`：是否为共享锁


### 注意事项

在同一个JVM进程中，一个文件的同区域锁只能获取一次，第二次获取锁时会直接报错（无论是当前线程还是其他线程，只要在同一个JVM进程中就会报错），
而不会等待第一次获取的锁释放。

> File locks are held on behalf of the entire Java virtual machine. They are not suitable for controlling access to a file by multiple 
threads within the same virtual machine.

> On some systems, closing a channel releases all locks held by the Java virtual machine on the underlying file regardless of whether the 
locks were acquired via that channel or via another channel open on the same file. It is strongly recommended that, within a program, 
a unique channel be used to acquire all locks on any given file.
