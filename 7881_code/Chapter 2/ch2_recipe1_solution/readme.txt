
Synchronizing a method

In this recipe, we will learn how to use one of the most basic methods for synchronization in Java, that is, the use of the synchronized keyword to control the concurrent access to a method. Only one execution thread will access one of the methods of an object declared with the synchronized keyword. If another thread tries to access any method declared with the synchronized keyword of the same object, it will be suspended until the first thread finishes the execution of the method.

In other words, every method declared with the synchronized keyword is a critical section and Java only allows the execution of one of the critical sections of an object.

Static methods have a different behavior. Only one execution thread will access one of the static methods declared with the synchronized keyword, but another thread can access other non-static methods of an object of that class. You have to be very careful with this point, because two threads can access two different synchronized methods if one is static and the other one is not. If both methods change the same data, you can have data inconsistency errors.

To learn this concept, we will implement an example with two threads accessing a common object. We will have a bank account and two threads; one that transfers money to the account and another one that withdraws money from the account. Without synchronization methods, we could have incorrect results. Synchronization mechanisms ensures that the final balance of the account will be correct.

Getting ready

The example of this recipe has been implemented using the Eclipse IDE. If you use Eclipse or other IDE such as NetBeans, open it and create a new Java project.

How to do it...

Follow these steps to implement the example:

Create a class called Account that will model our bank account. It has only one double attribute, named balance.
public class Account {
      private double balance;
Implement the setBalance() and getBalance() methods to write and read the value of the attribute.
  public double getBalance() {
    return balance;
  }

  public void setBalance(double balance) {
    this.balance = balance;
  }
Implement a method called addAmount() that increments the value of the balance in a certain amount that is passed to the method. Only one thread should change the value of the balance, so use the synchronized keyword to convert this method into a critical section.
  public synchronized void addAmount(double amount) {
    double tmp=balance;
    try {
      Thread.sleep(10);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    tmp+=amount;
    balance=tmp;
  }
Implement a method called subtractAmount()that decrements the value of the balance in a certain amount that is passed to the method. Only one thread should change the value of the balance, so use the synchronized keyword to convert this method into a critical section.
  public synchronized void subtractAmount(double amount) {
    double tmp=balance;
    try {
      Thread.sleep(10);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    tmp-=amount;
    balance=tmp;
  }
Implement a class that simulates an ATM. It will use the subtractAmount() method to decrement the balance of an account. This class must implement the Runnable interface to be executed as a thread.
public class Bank implements Runnable {
Add an Account object to this class. Implement the constructor of the class that initializes that Account object.
  private Account account;

  public Bank(Account account) {
    this.account=account;
  }
Implement the run() method. It makes 100 calls to the subtractAmount() method of an account to reduce the balance.
  @Override
   public void run() {
    for (int i=0; i<100; i++){
      account.sustractAmount(1000);
    }
  }
Implement a class that simulates a company and uses the addAmount() method of the Account class to increment the balance of the account. This class must implement the Runnable interface to be executed as a thread.
public class Company implements Runnable {
Add an Account object to this class. Implement the constructor of the class that initializes that account object.
  private Account account;

  public Company(Account account) {
    this.account=account;
  }
Implement the run() method . It makes 100 calls to the addAmount() method of an account to increment the balance.
  @Override
   public void run() {
    for (int i=0; i<100; i++){
      account.addAmount(1000);
    }
  }
Implement the main class of the application by creating a class named Main that contains the main() method.
public class Main {

  public static void main(String[] args) {
Create an Account object and initialize its balance to 1000.
    Account  account=new Account();
    account.setBalance(1000);
Create a Company object and Thread to run it.
    Company  company=new Company(account);
    Thread companyThread=new Thread(company);
Create a Bank object and Thread to run it.
    Bank bank=new Bank(account);
    Thread bankThread=new Thread(bank);
Write the initial balance to the console.
    System.out.printf("Account : Initial Balance: %f\n",account.getBalance());
Start the threads.
    companyThread.start();
    bankThread.start();
Wait for the finalization of the two threads using the join() method and print in the console the final balance of the account.
    try {
      companyThread.join();
      bankThread.join();
      System.out.printf("Account : Final Balance: %f\n",account.getBalance());
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
How it works...

In this recipe, you have developed an application that increments and decrements the balance of a class that simulates a bank account. The program makes 100 calls to the addAmount() method that increments the balance by 1000 in each call and 100 calls to the subtractAmount() method that decrements the balance by 1000 in each call. You should expect the final and initial balances to be equal.

You have tried to force an error situation using a variable named tmp to store the value of the account's balance, so you read the account's balance, you increment the value of the temporal variable, and then you establish the value of the account's balance again. Additionally, you have introduced a little delay using the sleep() method of the Thread class to put the thread that is executing the method to sleep for 10 milliseconds, so if another thread executes that method, it can modify the account's balance provoking an error. It's the synchronized keyword mechanism that avoids those errors.

If you want to see the problems of concurrent access to shared data, delete the synchronized keyword of the addAmount() and subtractAmount() methods and run the program. Without the synchronized keyword, while a thread is sleeping after reading the value of the account's balance, another method will read the account's balance, so both the methods will modify the same balance and one of the operations won't be reflected in the final result.

As you can see in the following screenshot, you can obtain inconsistent results:

How it works...
If you run the program often, you will obtain different results. The order of execution of the threads is not guaranteed by the JVM. So every time you execute them, the threads will read and modify the account's balance in a different order, so the final result will be different.

Now, add the synchronize keyword as you learned before and run the program again. As you can see in the following screenshot, now you obtain the expected result. If you run the program often, you will obtain the same result. Refer to the following screenshot:

How it works...
Using the synchronized keyword, we guarantee correct access to shared data in concurrent applications.

As we mentioned in the introduction of this recipe, only a thread can access the methods of an object that use the synchronized keyword in their declaration. If a thread (A) is executing a synchronized method and another thread (B) wants to execute other synchronized methods of the same object, it will be blocked until the thread (A) ends. But if threadB has access to different objects of the same class, none of them will be blocked.

There's more...

The synchronized keyword penalizes the performance of the application, so you must only use it on methods that modify shared data in a concurrent environment. If you have multiple threads calling a synchronized method, only one will execute them at a time while the others will be waiting. If the operation doesn't use the synchronized keyword, all the threads can execute the operation at the same time, reducing the total execution time. If you know that a method will not be called by more than one thread, don't use the synchronized keyword.

You can use recursive calls with synchronized methods. As the thread has access to the synchronized methods of an object, you can call other synchronized methods of that object, including the method that is executing. It won't have to get access to the synchronized methods again.

We can use the synchronized keyword to protect the access to a block of code instead of an entire method. We should use the synchronized keyword in this way to protect the access to the shared data, leaving the rest of operations out of this block, obtaining a better performance of the application. The objective is to have the critical section (the block of code that can be accessed only by one thread at a time) be as short as possible. We have used the synchronized keyword to protect the access to the instruction that updates the number of persons in the building, leaving out the long operations of this block that don't use the shared data. When you use the synchronized keyword in this way, you must pass an object reference as a parameter. Only one thread can access the synchronized code (blocks or methods) of that object. Normally, we will use the this keyword to reference the object that is executing the method.

    synchronized (this) {
      // Java code
    }
