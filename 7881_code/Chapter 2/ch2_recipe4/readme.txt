Synchronizing a block of code with a Lock

Java provides another mechanism for the synchronization of blocks of code. It's a more powerful and flexible mechanism than the synchronized keyword. It's based on the Lock interface and classes that implement it (as ReentrantLock). This mechanism presents some advantages, which are as follows:

It allows the structuring of synchronized blocks in a more flexible way. With the synchronized keyword, you have to get and free the control over a synchronized block of code in a structured way. The Lock interfaces allow you to get more complex structures to implement your critical section.
The Lock interfaces provide additional functionalities over the synchronized keyword. One of the new functionalities is implemented by the tryLock() method. This method tries to get the control of the lock and if it can't, because it's used by other thread, it returns the lock. With the synchronized keyword, when a thread (A) tries to execute a synchronized block of code, if there is another thread (B) executing it, the thread (A) is suspended until the thread (B) finishes the execution of the synchronized block. With locks, you can execute the tryLock() method. This method returns a Boolean value indicating if there is another thread running the code protected by this lock.
The Lock interfaces allow a separation of read and write operations having multiple readers and only one modifier.
The Lock interfaces offer better performance than the synchronized keyword.
In this recipe, you will learn how to use locks to synchronize a block of code and create a critical section using the Lock interface and the ReentrantLock class that implements it, implementing a program that simulates a print queue.

Getting Ready...

The example of this recipe has been implemented using the Eclipse IDE. If you use Eclipse or other IDE such as NetBeans, open it and create a new Java project.

How to do it...

Follow these steps to implement the example:

Create a class named PrintQueue that will implement the print queue.
public class PrintQueue {
Declare a Lock object and initialize it with a new object of the ReentrantLock class.
  private final Lock queueLock=new ReentrantLock();
Implement the printJob() method. It will receive Object as a parameter and it will not return any value.
  public void printJob(Object document){
Inside the printJob() method, get the control of the Lock object calling the lock() method.
    queueLock.lock();
Then, include the following code to simulate the printing of a document:
    try {
      Long duration=(long)(Math.random()*10000);
      System.out.println(Thread.currentThread().getName()+ ": PrintQueue: Printing a Job during "+(duration/1000)+
" seconds");
      Thread.sleep(duration);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
Finally, free the control of the Lock object with the unlock() method.
finally {
      queueLock.unlock();
    }
Create a class named Job and specify that it implements the Runnable interface.
public class Job implements Runnable {
Declare an object of the PrintQueue class and implement the constructor of the class that initializes that object.
  private PrintQueue printQueue;

  public Job(PrintQueue printQueue){
    this.printQueue=printQueue;
  }
Implement the run() method. It uses the PrintQueue object to send a job to print.
  @Override
  public void run() {
    System.out.printf("%s: Going to print a document\n", Thread.currentThread().getName());
    printQueue.printJob(new Object());
    System.out.printf("%s: The document has been printed\n", Thread.currentThread().getName());
  }
Create the main class of the application by implementing a class named Main and add the main() method to it.
public class Main {

  public static void main (String args[]){
Create a shared PrintQueue object.
    PrintQueue printQueue=new PrintQueue();
Create 10 Job objects and 10 threads to run them.
    Thread thread[]=new Thread[10];
    for (int i=0; i<10; i++){
      thread[i]=new Thread(new Job(printQueue),"Thread "+ i);
    }
Start the 10 threads.
    for (int i=0; i<10; i++){
      thread[i].start();
    }
How it works...

In the following screenshot, you can see a part of the output of one execution, of this example:

How it works...
The key to the example is in the printJob() method of the PrintQueue class. When we want to implement a critical section using locks and guarantee that only one execution thread runs a block of code, we have to create a ReentrantLock object. At the beginning of the critical section, we have to get the control of the lock using the lock() method. When a thread (A) calls this method, if no other thread has the control of the lock, the method gives the thread (A) the control of the lock and returns immediately to permit the execution of the critical section to this thread. Otherwise, if there is another thread (B) executing the critical section controlled by this lock, the lock() method puts the thread (A) to sleep until the thread (B) finishes the execution of the critical section.

At the end of the critical section, we have to use the unlock() method to free the control of the lock and allow the other threads to run this critical section. If you don't call the unlock() method at the end of the critical section, the other threads that are waiting for that block will be waiting forever, causing a deadlock situation. If you use try-catch blocks in your critical section, don't forget to put the sentence containing the unlock() method inside the finally section.

There's more...

The Lock interface (and the ReentrantLock class) includes another method to get the control of the lock. It's the tryLock() method. The biggest difference with the lock() method is that this method, if the thread that uses it can't get the control of the Lock interface, returns immediately and doesn't put the thread to sleep. This method returns a boolean value, true if the thread gets the control of the lock, and false if not.

Note
Take into consideration that it is the responsibility of the programmer to take into account the result of this method and act accordingly. If the method returns the false value, it's expected that your program doesn't execute the critical section. If it does, you probably will have wrong results in your application.

The ReentrantLock class also allows the use of recursive calls. When a thread has the control of a lock and makes a recursive call, it continues with the control of the lock, so the calling to the lock() method will return immediately and the thread will continue with the execution of the recursive call. Moreover, we can also call other methods.

More Info
You have to be very careful with the use of Locks to avoid deadlocks . This situation occurs when two or more threads are blocked waiting for locks that never will be unlocked. For example, a thread (A) locks a Lock (X) and a thread (B) locks a Lock (Y). If now, the thread (A) tries to lock the Lock (Y) and the thread (B) simultaneously tries to lock the Lock (X), both threads will be blocked indefinitely, because they are waiting for locks that will never be liberated. Note that the problem occurs, because both threads try to get the locks in the opposite order. The Appendix , Concurrent programming design, explains some good tips to design concurrent applications adequately and avoid these deadlocks problems.

See also

The Synchronizing a method recipe in Chapter 2, Basic Thread Synchronization
The Using multiple conditions in a Lock recipe in Chapter 2, Basic Thread Synchronization
