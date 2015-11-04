Creating threads through a factory

The factory pattern is one of the most used design patterns in the object-oriented programming world. It is a creational pattern and its objective is to develop an object whose mission will be creating other objects of one or several classes. Then, when we want to create an object of one of those classes, we use the factory instead of using the new operator.

With this factory, we centralize the creation of objects with some advantages:

It's easy to change the class of the objects created or the way we create these objects.
It's easy to limit the creation of objects for limited resources. For example, we can only have n objects of a type.
It's easy to generate statistical data about the creation of the objects.
Java provides an interface, the ThreadFactory interface to implement a Thread object factory. Some advanced utilities of the Java concurrency API use thread factories to create threads.

In this recipe, we will learn how to implement a ThreadFactory interface to create Thread objects with a personalized name while we save statistics of the Thread objects created.

Getting ready

The example of this recipe has been implemented using the Eclipse IDE. If you use Eclipse or other IDE such as NetBeans, open it and create a new Java project.

How to do it...

Follow these steps to implement the example:

Create a class called MyThreadFactory and specify that it implements the ThreadFactory interface.
public class MyThreadFactory implements ThreadFactory {
Declare three attributes: an integer number called counter, which we will use to store the number of the Thread object created, a String called name with the base name of every Thread created, and a List of String objects called stats to save statistical data about the Thread objects created. We also implement the constructor of the class that initializes these attributes.
  private int counter;
  private String name;
  private List<String> stats;

  public MyThreadFactory(String name){
    counter=0;
    this.name=name;
    stats=new ArrayList<String>();
  }
Implement the newThread() method. This method will receive a Runnable interface and returns a Thread object for this Runnable interface. In our case, we generate the name of the Thread object, create the new Thread object, and save the statistics.
  @Override
  public Thread newThread(Runnable r) {
    Thread t=new Thread(r,name+"-Thread_"+counter);
    counter++;
    stats.add(String.format("Created thread %d with name %s on %s\n",t.getId(),t.getName(),new Date()));
    return t;
  }
Implement the method getStatistics() that returns a String object with the statistical data of all the Thread objects created.
  public String getStats(){
    StringBuffer buffer=new StringBuffer();
    Iterator<String> it=stats.iterator();

    while (it.hasNext()) {
      buffer.append(it.next());
      buffer.append("\n");
    }

    return buffer.toString();
  }
Create a class called Task and specify that it implements the Runnable interface. For this example, these tasks are going to do nothing apart from sleeping for one second.
public class Task implements Runnable {
  @Override
  public void run() {
    try {
      TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}
Create the main class of the example. Create a class called Main and implement the main() method.
public class Main {
  public static void main(String[] args) {
Create a MyThreadFactory object and a Task object.
    MyThreadFactory factory=new MyThreadFactory("MyThreadFactory");
    Task task=new Task();
Create 10 Thread objects using the MyThreadFactory object and start them.
    Thread thread;
    System.out.printf("Starting the Threads\n");
    for (int i=0; i<10; i++){
      thread=factory.newThread(task);
      thread.start();
    }
Write in the console the statistics of the thread factory.
    System.out.printf("Factory stats:\n");
    System.out.printf("%s\n",factory.getStats());
Run the example and see the results.
How it works...

The ThreadFactory interface has only one method called newThread. It receives a Runnable object as a parameter and returns a Thread object. When you implement a ThreadFactory interface, you have to implement that interface and override this method. Most basic ThreadFactory, has only one line.

return new Thread(r);
You can improve this implementation by adding some variants by:

Creating personalized threads, as in the example, using a special format for the name or even creating our own thread class that inherits the Java Thread class
Saving thread creation statistics, as shown in the previous example
Limiting the number of threads created
Validating the creation of the threads
And anything more you can imagine
The use of the factory design pattern is a good programming practice but, if you implement a ThreadFactory interface to centralize the creation of threads, you have to review the code to guarantee that all threads are created using that factory.

See also

The Implementing the ThreadFactory interface to generate custom threads recipe in Chapter 7, Customizing Concurrency Classes
The Using our ThreadFactory in an Executor object recipe in Chapter 7, Customizing Concurrency Classes
