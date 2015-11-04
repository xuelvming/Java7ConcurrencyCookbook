Using multiple conditions in a Lock

A lock may be associated with one or more conditions. These conditions are declared in the Condition interface. The purpose of these conditions is to allow threads to have control of a lock and check whether a condition is true or not and, if it's false, be suspended until another thread wakes them up. The Condition interface provides the mechanisms to suspend a thread and to wake up a suspended thread.

A classic problem in concurrent programming is the producer-consumer problem. We have a data buffer, one or more producers of data that save it in the buffer, and one or more consumers of data that take it from the buffer as explained earlier in this chapter

In this recipe, you will learn how to implement the producer-consumer problem using locks and conditions.

Getting Ready...

You should read the Synchronizing a block of code with a Lock recipe for a better understanding of this recipe.

How to do it...

Follow these steps to implement the example:

First, let's implement a class that will simulate a text file. Create a class named FileMock with two attributes: a String array named content and int named index. They will store the content of the file and the line of the simulated file that will be retrieved.
public class FileMock {

  private String content[];
  private int index;
Implement the constructor of the class that initializes the content of the file with random characters.
  public FileMock(int size, int length){
    content=new String[size];
    for (int i=0; i<size; i++){
      StringBuilder buffer=new StringBuilder(length);
      for (int j=0; j<length; j++){
        int indice=(int)Math.random()*255;
        buffer.append((char)indice);
      }
      content[i]=buffer.toString();
    }
    index=0;
  }
Implement the method hasMoreLines() that returns true if the file has more lines to process or false if we have achieved the end of the simulated file.
  public boolean hasMoreLines(){
    return index<content.length;
  }
Implement the method getLine()that returns the line determined by the index attribute and increases its value.
  public String getLine(){
    if (this.hasMoreLines()) {
      System.out.println("Mock: "+(content.length-index));
      return content[index++];
    }
    return null;
  }
Now, implement a class named Buffer that will implement the buffer shared by producers and consumers.
public class Buffer {
This class has six attributes:
A LinkedList<String> attribute named buffer that will store the shared data
An int type named maxSize that stores the length of the buffer
A ReentrantLock object called lock that controls the access to the blocks of code that modify the buffer
Two Condition attributes named lines and space
A boolean type called pendingLines that will indicate if there are lines in the buffer
  private LinkedList<String> buffer;

  private int maxSize;

  private ReentrantLock lock;

  private Condition lines;
  private Condition space;

  private boolean pendingLines;
Implement the constructor of the class. It initializes all the attributes described previously.
  public Buffer(int maxSize) {
    this.maxSize=maxSize;
    buffer=new LinkedList<>();
    lock=new ReentrantLock();
    lines=lock.newCondition();
    space=lock.newCondition();
    pendingLines=true;
  }
Implement the insert() method. It receives String as a parameter and tries to store it in the buffer. First, it gets the control of the lock. When it has it, it then checks if there is empty space in the buffer. If the buffer is full, it calls the await() method in the space condition to wait for free space. The thread will be woken up when another thread calls thesignal() or signalAll() method in the space Condition. When that happens, the thread stores the line in the buffer and calls the signallAll() method over the lines condition. As we'll see in a moment, this condition will wake up all the threads that were waiting for lines in the buffer.
  public void insert(String line) {
    lock.lock();
    try {
      while (buffer.size() == maxSize) {
        space.await();
      }
      buffer.offer(line);
      System.out.printf("%s: Inserted Line: %d\n", Thread.currentThread().getName(),buffer.size());
      lines.signalAll();
    } catch (InterruptedException e) {
      e.printStackTrace();
    } finally {
      lock.unlock();
    }
  }
Implement the get() method. It returns the first string stored in the buffer. First, it gets the control of the lock. When it has it, it checks if there are lines in the buffer. If the buffer is empty, it calls the await() method in the lines condition to wait for lines in the buffer. This thread will be woken up when another thread calls the signal() or signalAll() method in the lines condition. When it happens, the method gets the first line in the buffer, calls the signalAll() method over the space condition and returns String.
  public String get() {
    String line=null;
    lock.lock();
    try {
      while ((buffer.size() == 0) &&(hasPendingLines())) {
        lines.await();
      }

      if (hasPendingLines()) {
        line = buffer.poll();
        System.out.printf("%s: Line Readed: %d\n",Thread.currentThread().getName(),buffer.size());
        space.signalAll();
      }
    } catch (InterruptedException e) {
      e.printStackTrace();
    } finally {
      lock.unlock();
    }
    return line;
  }
Implement the setPendingLines() method that establishes the value of the attribute pendingLines. It will be called by the producer when it has no more lines to produce.
  public void setPendingLines(boolean pendingLines) {
    this.pendingLines=pendingLines;
  }
Implement the hasPendingLines() method. It returns true if there are more lines to be processed, or false otherwise.
  public boolean hasPendingLines() {
    return pendingLines || buffer.size()>0;
  }
It's now the turn of the producer. Implement a class named Producer and specify that it implements the Runnable interface.
public class Producer implements Runnable {
Declare two attributes: one object of the FileMock class and another object of the Buffer class.
  private FileMock mock;

  private Buffer buffer;
Implement the constructor of the class that initializes both attributes.
  public Producer (FileMock mock, Buffer buffer){
    this.mock=mock;
    this.buffer=buffer;
  }
Implement the run() method that reads all the lines created in the FileMock object and uses the insert() method to store them in the buffer. Once it finishes, use the setPendingLines() method to alert the buffer that it's not going to generate more lines.
   @Override
  public void run() {
    buffer.setPendingLines(true);
    while (mock.hasMoreLines()){
      String line=mock.getLine();
      buffer.insert(line);
    }
    buffer.setPendingLines(false);
  }
Next is the consumer's turn. Implement a class named Consumer and specify that it implements the Runnable interface.
public class Consumer implements Runnable {
Declare a Buffer object and implement the constructor of the class that initializes it.
  private Buffer buffer;

  public Consumer (Buffer buffer) {
    this.buffer=buffer;
  }
Implement the run() method. While the buffer has pending lines, it tries to get one and process it.
   @Override
  public void run() {
    while (buffer.hasPendingLines()) {
      String line=buffer.get();
      processLine(line);
    }
  }
Implement the auxiliary method processLine(). It only sleeps for 10 milliseconds to simulate some kind of processing with the line.
  private void processLine(String line) {
    try {
      Random random=new Random();
      Thread.sleep(random.nextInt(100));
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
Implement the main class of the example by creating a class named Main and add the main() method to it.
public class Main {

  public static void main(String[] args) {
Create a FileMock object.
    FileMock mock=new FileMock(100, 10);
Create a Buffer object.
    Buffer buffer=new Buffer(20);
Create a Producer object and Thread to run it.
    Producer producer=new Producer(mock, buffer);
    Thread threadProducer=new Thread(producer,"Producer");
Create three Consumer objects and three threads to run it.
    Consumer consumers[]=new Consumer[3];
    Thread threadConsumers[]=new Thread[3];

    for (int i=0; i<3; i++){
      consumers[i]=new Consumer(buffer);
      threadConsumers[i]=new Thread(consumers[i],"Consumer "+i);
    }
Start the producer and the three consumers.
    threadProducer.start();
    for (int i=0; i<3; i++){
      threadConsumers[i].start();
    }
How it works...

All the Condition objects are associated with a lock and are created using the newCondition() method declared in the Lock interface. Before we can do any operation with a condition, you have to have the control of the lock associated with the condition, so the operations with conditions must be in a block of code that begins with a call to a lock() method of a Lock object and ends with an unlock() method of the same Lock object.

When a thread calls the await() method of a condition, it automatically frees the control of the lock, so that another thread can get it and begin the execution of the same, or another critical section protected by that lock.

Note
When a thread calls the signal() or signallAll() methods of a condition, one or all of the threads that were waiting for that condition are woken up, but this doesn't guarantee that the condition that made them sleep is now true, so you must put the await() calls inside a while loop. You can't leave that loop until the condition is true. While the condition is false, you must call await() again.

You must be careful with the use of await() and signal(). If you call the await() method in a condition and never call the signal() method in this condition, the thread will be sleeping forever.

A thread can be interrupted while it is sleeping, after a call to the await() method, so you have to process the InterruptedException exception.

There's more...

The Condition interface has other versions of the await() method, which are as follows:

await(long time, TimeUnit unit): The thread will be sleeping until:
It's interrupted
Another thread calls the singal() or signalAll() methods in the condition
The specified time passes
The TimeUnit class is an enumeration with the following constants: DAYS, HOURS, MICROSECONDS, MILLISECONDS, MINUTES, NANOSECONDS, and SECONDS
awaitUninterruptibly(): The thread will be sleeping until another thread calls the signal() or signalAll() methods, which can't be interrupted
awaitUntil(Date date): The thread will be sleeping until:
It's interrupted
Another thread calls the singal() or signalAll() methods in the condition
The specified date arrives
You can use conditions with the ReadLock and WriteLock locks of a read/write lock.

See also

The Synchronizing a block of code with a Lock recipe in Chapter 2, Basic Thread Synchronization
The Synchronizing data access with read/write locks recipe in Chapter 2, Basic Thread Synchronization
