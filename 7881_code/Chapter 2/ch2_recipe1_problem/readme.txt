Processing uncontrolled exceptions in a group of threads

A very important aspect in every programming language is the mechanism that provides management of error situations in your application. Java language, as almost all modern programming languages, implements an exception-based mechanism to manage error situations. It provides a lot of classes to represent different errors. Those exceptions are thrown by the Java classes when an error situation is detected. You can also use those exceptions, or implement your own exceptions, to manage the errors produced in your classes.

Java also provides a mechanism to capture and process those exceptions. There are exceptions that must be captured or re-thrown using the throws clause of a method. These exceptions are called checked exceptions. There are exceptions that don't have to be specified or caught. These are the unchecked exceptions.

In the recipe, Controlling the interruption of a Thread, you learned how to use a generic method to process all the uncaught exceptions that are thrown in a Thread object.

Another possibility is to establish a method that captures all the uncaught exceptions thrown by any Thread of the ThreadGroup class.

In this recipe, we will learn to set this handler using an example.

Getting ready

The example of this recipe has been implemented using the Eclipse IDE. If you use Eclipse or other IDE such as NetBeans, open it and create a new Java project.

How to do it...

Follow these steps to implement the example:

First, we have to extend the ThreadGroup class by creating a class called MyThreadGroup that extends from ThreadGroup. We have to declare a constructor with one parameter, because the ThreadGroup class doesn't have a constructor without it.
public class MyThreadGroup extends ThreadGroup {
  public MyThreadGroup(String name) {
    super(name);
  }
Override the uncaughtException() method. This method is called when an exception is thrown in one of the threads of the ThreadGroup class. In this case, this method will write in the console information about the exception and Thread that throws it and interrupts the rest of the threads in the ThreadGroup class.
  @Override
  public void uncaughtException(Thread t, Throwable e) {
    System.out.printf("The thread %s has thrown an Exception\n",t.getId());
    e.printStackTrace(System.out);
    System.out.printf("Terminating the rest of the Threads\n");
    interrupt();
  }
Create a class called Task and specify that it implements the Runnable interface.
public class Task implements Runnable {
Implement the run() method. In this case, we will provoke an AritmethicException exception. For this, we will divide 1000 between random numbers until the random generator generates a zero and the exception is thrown.
  @Override
  public void run() {
    int result;
    Random random=new Random(Thread.currentThread().getId());
    while (true) {
      result=1000/((int)(random.nextDouble()*1000));
      System.out.printf("%s : %f\n",Thread.currentThread().getId(),result);
      if (Thread.currentThread().isInterrupted()) {
        System.out.printf("%d : Interrupted\n",Thread.currentThread().getId());
        return;
      }
    }
  }
Now, we are going to implement the main class of the example by creating a class called Main and implement the main() method.
public class Main {
  public static void main(String[] args) {
Create an object of the MyThreadGroup class.
    MyThreadGroup threadGroup=new MyThreadGroup("MyThreadGroup");
Create an object of the Task class.
    Task task=new Task();
Create two Thread objects with this Task and start them.
    for (int i=0; i<2; i++){
      Thread t=new Thread(threadGroup,task);
      t.start();
    }
Run the example and see the results.
How it works...

When you run the example, you will see how one of the Thread objects threw the exception and the other one was interrupted.

When an uncaught exception is thrown in Thread, the JVM looks for three possible handlers for this exception.

First, it looks for the uncaught exception handler of the thread, as was explained in the Processing uncontrolled exceptions in a Thread recipe. If this handler doesn't exist, then the JVM looks for the uncaught exception handler for the ThreadGroup class of the thread, as we learned in this recipe. If this method doesn't exist, the JVM looks for the default uncaught exception handler, as was explained in the Processing uncontrolled exceptions in a Thread recipe.

If none of the handlers exit, the JVM writes the stack trace of the exception in the console and exits the program.

See also

The Processing uncontrolled exceptions in a thread recipe in Chapter 1, Thread Management
