Synchronizing data access with read/write locks

One of the most significant improvements offered by locks is the ReadWriteLock interface and the ReentrantReadWriteLock class, the unique one that implements it. This class has two locks, one for read operations and one for write operations. There can be more than one thread using read operations simultaneously, but only one thread can be using write operations. When a thread is doing a write operation, there can't be any thread doing read operations.

In this recipe, you will learn how to use a ReadWriteLock interface implementing a program that uses it to control the access to an object that stores the prices of two products.

Getting Ready...

You should read the Synchronizing a block of code with a Lock recipe for a better understanding of this recipe.

How to do it...

Follow these steps to implement the example:

Create a class named PricesInfo that stores information about the prices of two products.
public class PricesInfo {
Declare two double attributes named price1 and price2.
  private double price1;
  private double price2;
Declare a ReadWriteLock object called lock.
  private ReadWriteLock lock;
Implement the constructor of the class that initializes the three attributes. For the lock attribute, we create a new ReentrantReadWriteLock object.
  public PricesInfo(){
    price1=1.0;
    price2=2.0;
    lock=new ReentrantReadWriteLock();
  }
Implement the getPrice1() method that returns the value of the price1 attribute. It uses the read lock to control the access to the value of this attribute.
  public double getPrice1() {
    lock.readLock().lock();
    double value=price1;
    lock.readLock().unlock();
    return value;
  }
Implement the getPrice2() method that returns the value of the price2 attribute. It uses the read lock to control the access to the value of this attribute.
  public double getPrice2() {
    lock.readLock().lock();
    double value=price2;
    lock.readLock().unlock();
    return value;
  }
Implement the setPrices() method that establishes the values of the two attributes. It uses the write lock to control access to them.
  public void setPrices(double price1, double price2) {
    lock.writeLock().lock();
    this.price1=price1;
    this.price2=price2;
    lock.writeLock().unlock();
  }
Create a class named Reader and specify that it implements the Runnable interface. This class implements a reader of the values of the PricesInfo class attributes.
public class Reader implements Runnable {
Declare a PricesInfo object and implement the constructor of the class that initializes that object.
  private PricesInfo pricesInfo;

  public Reader (PricesInfo pricesInfo){
    this.pricesInfo=pricesInfo;
  }
Implement the run() method for this class. It reads 10 times the value of the two prices.
  @Override
  public void run() {
    for (int i=0; i<10; i++){
      System.out.printf("%s: Price 1: %f\n", Thread.currentThread().getName(),pricesInfo.getPrice1());
      System.out.printf("%s: Price 2: %f\n", Thread.currentThread().getName(),pricesInfo.getPrice2());
    }
  }
Create a class named Writer and specify that it implements the Runnable interface. This class implements a modifier of the values of the PricesInfo class attributes.
public class Writer implements Runnable {
Declare a PricesInfo object and implement the constructor of the class that initializes that object.
  private PricesInfo pricesInfo;

  public Writer(PricesInfo pricesInfo){
    this.pricesInfo=pricesInfo;
  }
Implement the run() method. It modifies three times the value of the two prices that are sleeping for two seconds between modifications.
  @Override
  public void run() {
    for (int i=0; i<3; i++) {
      System.out.printf("Writer: Attempt to modify the prices.\n");
      pricesInfo.setPrices(Math.random()*10, Math.random()*8);
      System.out.printf("Writer: Prices have been modified.\n");
      try {
        Thread.sleep(2);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
  }
Implement the main class of the example by creating a class named Main and add the main() method to it.
public class Main {

  public static void main(String[] args) {
Create a PricesInfo object.
    PricesInfo pricesInfo=new PricesInfo();
Create five Reader objects and five Threads to execute them.
    Reader readers[]=new Reader[5];
    Thread threadsReader[]=new Thread[5];

    for (int i=0; i<5; i++){
      readers[i]=new Reader(pricesInfo);
      threadsReader[i]=new Thread(readers[i]);
    }
Create a Writer object and Thread to execute it.
    Writer writer=new Writer(pricesInfo);
      Thread  threadWriter=new Thread(writer);
Start the threads.
    for (int i=0; i<5; i++){
      threadsReader[i].start();
    }
    threadWriter.start();
How it works...

In the following screenshot, you can see a part of the output of one execution of this example:

How it works...
As we mentioned previously, the ReentrantReadWriteLock class has two locks, one for read operations and one for write operations. The lock used in read operations is obtained with the readLock() method declared in the ReadWriteLock interface. This lock is an object that implements the Lock interface, so we can use the lock(), unlock(), and tryLock() methods. The lock used in write operations is obtained with the writeLock() method declared in the ReadWriteLock interface. This lock is an object that implements the Lock interface, so we can use the lock(), unlock(), and tryLock() methods. It is the responsibility of the programmer to ensure the correct use of these locks, using them with the same purposes for which they were designed.When you get the read lock of a Lock interface, you can't modify the value of the variable. Otherwise, you probably will have inconsistency data errors.

See also

The Synchronizing a block of code with a Lock recipe in Chapter 2, Basic Thread Synchronization
The Monitoring a Lock interface recipe in Chapter 8, Testing concurrent Applications
