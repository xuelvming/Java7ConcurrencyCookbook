Controlling concurrent access to multiple copies of a resource

In the Controlling concurrent access to a resource recipe, you learned the basis of semaphores.

In that recipe, you implemented an example using binary semaphores. These kinds of semaphores are used to protect the access to one shared resource, or to a critical section that can only be executed by one thread at a time. But semaphores can also be used when you need to protect various copies of a resource, or when you have a critical section that can be executed by more than one thread at the same time.

In this recipe, you will learn how to use a semaphore to protect more than one copy of a resource. You are going to implement an example, which has one print queue that can print documents in three different printers.

Getting ready

The example of this recipe has been implemented using the Eclipse IDE. If you use Eclipse or other IDE such as NetBeans, open it and create a new Java project.

Implement the example described in the Controlling concurrent access to a resource recipe in this chapter.

How to do it...

Follow these steps to implement the example:

As we mentioned earlier, you are going to modify the print queue example implemented with semaphores. Open the PrintQueue class and declare a boolean array called freePrinters. This array stores printers that are free to print a job and printers that are printing a document.
  private boolean freePrinters[];
Also, declare a Lock object named lockPrinters. You will use this object to protect the access to the freePrinters array.
  private Lock lockPrinters;
Modify the constructor of the class to initialize the new declared objects. The freePrinters array has three elements, all initialized to the true value. The semaphore has 3 as its initial value.
  public PrintQueue(){
    semaphore=new Semaphore(3);
    freePrinters=new boolean[3];
    for (int i=0; i<3; i++){
      freePrinters[i]=true;
    }
    lockPrinters=new ReentrantLock();
  }
Modify also the printJob() method. It receives an Object called document as the unique parameter.
  public void printJob (Object document){
First of all, the method calls the acquire() method to acquire the access to the semaphore. As this method can throw an InterruptedException exception, you must include the code to process it.
    try {
      semaphore.acquire();
Then you get the number of the printer assigned to print this job using the private method getPrinter().
      int assignedPrinter=getPrinter();
Then, implement the lines that simulate the printing of a document waiting for a random period of time.
      long duration=(long)(Math.random()*10);
      System.out.printf("%s: PrintQueue: Printing a Job in Printer%d during %d seconds\n",Thread.currentThread().getName(),assignedPrinter,duration);
      TimeUnit.SECONDS.sleep(duration);
Finally, release the semaphore calling the release() method and mark the printer used as free, assigning true to the corresponding index in the freePrinters array.
      freePrinters[assignedPrinter]=true;
    } catch (InterruptedException e) {
      e.printStackTrace();
    } finally {
      semaphore.release();      
    }
Implement the getPrinter() method. It's a private method that returns an int value and it has no parameters.
  private int getPrinter() {
First of all, declare an int variable to store the index of the printer.
    int ret=-1;
Then, get the access to the lockPrinters object.
    try {
      lockPrinters.lock();
Then, find the first true value in the freePrinters array and save its index in a variable. Modify this value to false, because this printer will be busy.
    for (int i=0; i<freePrinters.length; i++) {
      if (freePrinters[i]){
        ret=i;
        freePrinters[i]=false;
        break;
      }
    }
Finally, free the lockPrinters object and return the index of the true value.
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      lockPrinters.unlock();
    }
    return ret;
The Job and Core classes have no modifications.
How it works...

The key of this example is in the PrintQueue class. The Semaphore object is created using 3 as the parameter of the constructor. The first three threads that call the acquire() method will get the access to the critical section of this example, while the rest will be blocked. When a thread finishes the critical section and releases the semaphore, another thread will acquire it.

In this critical section, the thread gets the index of the printer assigned to print this job. This part of the example is used to give more realism to the example, but it doesn't use any code related with semaphores.

The following screenshot shows the output of an execution of this example:

How it works...
Each document is printed in one of the printers. The first one is free.

There's more...

The acquire(), acquireUninterruptibly(), tryAcquire(), and release() methods have an additional version which has an int parameter. This parameter represents the number of permits that the thread that uses them wants to acquire or release, so as to say, the number of units that this thread wants to delete or to add to the internal counter of the semaphore. In the case of the acquire(), acquireUninterruptibly(), and tryAcquire() methods, if the value of this counter is less than this value, the thread will be blocked until the counter gets this value or a greater one.

See also

The Controlling concurrent access to a resource recipe in Chapter 3, Thread Synchronization Utilities
The Monitoring a Lock interface recipe in Chapter 8, Testing Concurrent Applications
The Modifying lock fairness recipe in Chapter 2, Basic Thread Synchronization