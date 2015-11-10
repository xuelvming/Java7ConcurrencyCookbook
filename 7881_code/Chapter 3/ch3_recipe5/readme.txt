Running concurrent phased tasks

One of the most complex and powerful functionalities offered by the Java concurrency API is the ability to execute concurrent-phased tasks using the Phaser class . This mechanism is useful when we have some concurrent tasks divided into steps. The Phaser class provides us with the mechanism to synchronize the threads at the end of each step, so no thread starts its second step until all the threads have finished the first one.

As with other synchronization utilities, we have to initialize the Phaser class with the number of tasks that participate in the synchronization operation, but we can dynamically modify this number by increasing or decreasing it.

In this recipe, you will learn how to use the Phaser class to synchronize three concurrent tasks. The three tasks look for files with the extension .log modified in the last 24 hours in three different folders and their subfolders. This task is divided into three steps:

Get a list of the files with the extension .log in the assigned folder and its subfolders.
Filter the list created in the first step by deleting the files modified more than 24 hours ago.
Print the results in the console.
At the end of the steps 1 and 2 we check if the list has any elements or not. If it hasn't any element, the thread ends its execution and is eliminated from the the phaser class.

Getting ready

The example of this recipe has been implemented using the Eclipse IDE. If you use Eclipse or other IDE like NetBeans, open it and create a new Java project.

How to do it...

Follow these steps to implement the example:

Create a class named FileSearch and specify that it implements the Runnable interface. This class implements the operation of searching for files with a determined extension modified in the last 24 hours in a folder and its subfolders.
public class FileSearch implements Runnable {
Declare a private String attribute to store the folder in which the search operation will begin.
  private String initPath;
Declare another private String attribute to store the extension of the files we are going to look for.
  private String end;
Declare a private List attribute to store the full path of the files we will find with the desired characteristics.
  private List<String> results;
Finally, declare a private Phaser attribute to control the synchronization of the different phases of the task.
  private Phaser phaser;
Implement the constructor of the class that will initialize the attributes of the class. It receives as parameters the full path of the initial folder, the extension of the files, and the phaser.
  public FileSearch(String initPath, String end, Phaser phaser) {
    this.initPath = initPath;
    this.end = end;
    this.phaser=phaser;
    results=new ArrayList<>();
  }
Now, you have to implement some auxiliary methods that will be used by the run() method. The first one is the directoryProcess() method. It receives a File object as a parameter and it processes all its files and subfolders. For each folder, the method will make a recursive call passing the folder as a parameter. For each file, the method will call the fileProcess() method:
  private void directoryProcess(File file) {

    File list[] = file.listFiles();
    if (list != null) {
      for (int i = 0; i < list.length; i++) {
        if (list[i].isDirectory()) {
          directoryProcess(list[i]);
        } else {
          fileProcess(list[i]);
        }
      }
    }
  }
Now, implement the fileProcess() method. It receives a File object as parameter and checks if its extension is equal to the one we are looking for. If they are equal, this method adds the absolute path of the file to the list of results.
  private void fileProcess(File file) {
    if (file.getName().endsWith(end)) {
      results.add(file.getAbsolutePath());
    }
  }
Now, implement the filterResults() method. It doesn't receive any parameter, and filters the list of files obtained in the first phase, deleting the files that were modified more than 24 hours ago. First, create a new empty list and get the actual date.
  private void filterResults() {
    List<String> newResults=new ArrayList<>();
    long actualDate=new Date().getTime();
Then, go through all the elements of the results list. For each path in the list of results, create a File object for that file and get the last modified date for it.
    for (int i=0; i<results.size(); i++){
      File file=new File(results.get(i));
      long fileDate=file.lastModified();

Then, compare that date with the actual date and, if the difference is less than one day, add the full path of the file to the new list of results.
      if (actualDate-fileDate< TimeUnit.MILLISECONDS.convert(1,TimeUnit.DAYS)){
        newResults.add(results.get(i));
      }
    }
Finally, change the old results list for the new one.
    results=newResults;
  }
Now, implement the checkResults() method. This method will be called at the end of the first and the second phase and it will check if the results list is empty or not. This method doesn't have any parameters.
  private boolean checkResults() {
First, check the size of the results list. If it's 0, the object writes a message to the console indicating this circumstance and then, calls the arriveAndDeregister() method of the Phaser object to notify it that this thread has finished the actual phase, and it leaves the phased operation.
  if (results.isEmpty()) {
      System.out.printf("%s: Phase %d: 0 results.\n",Thread.currentThread().getName(),phaser.getPhase());
      System.out.printf("%s: Phase %d: End.\n",Thread.currentThread().getName(),phaser.getPhase());
      phaser.arriveAndDeregister();
      return false;
Otherwise, if the results list has elements, the object writes a message to the console indicating this circumstance and then, calls the arriveAndAwaitAdvance() method of the Phaser object to notify it that this thread has finished the actual phase and it wants to be blocked until all the participant threads in the phased operation finish the actual phase.
    } else {
    System.out.printf("%s: Phase %d: %d results.\n",Thread.currentThread().getName(),phaser.getPhase(),results.size());
      phaser.arriveAndAwaitAdvance();
      return true;
    }
  }
The last auxiliary method is the showInfo() method that prints to the console the elements of the results list.
  private void showInfo() {
    for (int i=0; i<results.size(); i++){
      File file=new File(results.get(i));
      System.out.printf("%s: %s\n",Thread.currentThread().getName(),file.getAbsolutePath());
    }
    phaser.arriveAndAwaitAdvance();
  }
Now, it's time to implement the run() method that executes the operation using the auxiliary methods described earlier and the Phaser object to control the change between phases. First, call the arriveAndAwaitAdvance() method of the phaser object. The search won't begin until all the threads have been created.
   @Override
  public void run() {

    phaser.arriveAndAwaitAdvance();
Then, write a message to the console indicating the start of the search task.
    System.out.printf("%s: Starting.\n",Thread.currentThread().getName());
Check that the initPath attribute stores the name of a folder and use the directoryProcess() method to find the files with the specified extension in that folder and all its subfolders.
    File file = new File(initPath);
    if (file.isDirectory()) {
      directoryProcess(file);
    }
Check if there are any results using the checkResults() method. If there are no results, finish the execution of the thread with the return keyword.
    if (!checkResults()){
      return;
    }
Filter the list of results using the filterResults() method.
    filterResults();
Check again if there are any results using the checkResults() method. If there are no results, finish the execution of the thread with the return keyword.
    if (!checkResults()){
      return;
    }
Print the final list of results to the console with the showInfo() method, deregister the thread, and print a message indicating the finalization of the thread.
    showInfo();
    phaser.arriveAndDeregister();
    System.out.printf("%s: Work completed.\n",Thread.currentThread().getName());
Now, implement the main class of the example by creating a class named Main and add the main() method to it.
public class Main {

  public static void main(String[] args) {
Create a Phaser object with three participants.
    Phaser phaser=new Phaser(3);
Create three FileSearch objects with a different initial folder for each one. Look for the files with the .log extension.
    FileSearch system=new FileSearch("C:\\Windows", "log", phaser);
    FileSearch apps=
new FileSearch("C:\\Program Files","log",phaser);
    FileSearch documents=
new FileSearch("C:\\Documents And Settings","log",phaser);
Create and start a thread to execute the first FileSearch object.
    Thread systemThread=new Thread(system,"System");
    systemThread.start();
Create and start a thread to execute the second FileSearch object.
    Thread appsThread=new Thread(apps,"Apps");
    appsThread.start();
Create and start a thread to execute the third FileSearch object.
    Thread documentsThread=new Thread(documents, "Documents");
    documentsThread.start();
Wait for the finalization of the three threads.
    try {
      systemThread.join();
      appsThread.join();
      documentsThread.join();
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
Write the value of the finalized flag of the Phaser object using the isFinalized() method.
    System.out.println("Terminated: "+ phaser.isTerminated());
How it works...

The program starts creating a Phaser object that will control the synchronization of the threads at the end of each phase. The constructor of Phaser receives the number of participants as a parameter. In our case, Phaser has three participants. This number indicates to Phaser the number of threads that have to execute an arriveAndAwaitAdvance() method before Phaser changes the phase and wakes up the threads that were sleeping.

Once Phaser has been created, we launch three threads that execute three different FileSearch objects.

Note
In this example, we use paths of the Windows operating system. If you work with another operating system, modify the paths to adapt them to existing paths in your environment.

The first instruction in the run() method of this FileSearch object is a call to the arriveAndAwaitAdvance() method of the Phaser object. As we mentioned earlier, the Phaser knows the number of threads that we want to synchronize. When a thread calls this method, Phaser decreases the number of threads that have to finalize the actual phase and puts this thread to sleep until all the remaining threads finish this phase. Calling this method at the beginning of the run() method makes none of the FileSearch threads begin their job until all the threads have been created.

At the end of phase one and phase two, we check if the phase has generated results and the list with the results has elements, or otherwise the phase hasn't generated results and the list is empty. In the first case, the checkResults() method calls arriveAndAwaitAdvance() as explained earlier. In the second case, if the list is empty, there's no point in the thread continuing with its execution, so it returns. But you have to notify the phaser that there will be one less participant. For this, we used arriveAndDeregister(). This notifies the phaser that this thread has finished the actual phase, but it won't participate in the future phases, so the phaser won't have to wait for it to continue.

At the end of the phase three implemented in the showInfo() method, there is a call to the arriveAndAwaitAdvance() method of the phaser. With this call, we guarantee that all the threads finish at the same time. When this method ends its execution, there is a call to the arriveAndDeregister() method of the phaser. With this call, we deregister the threads of the phaser as we explained before, so when all the threads finish, the phaser will have zero participants.

Finally, the main() method waits for the completion of the three threads and calls the isTerminated() method of the phaser. When a phaser has zero participants, it enters the so called termination state and this method returns true. As we deregister all the threads of the phaser, it will be in the termination state and this call will print true to the console.

A Phaser object can be in two states:

Active: Phaserenters this state when it accepts the registration of new participants and its synchronization at the end of each phase. In this state, Phaser works as it has been explained in this recipe. This state is not mentioned in the Java concurrency API.
Termination: By default,Phaser enters in this state when all the participants in Phaser have been deregistered, so Phaser has zero participants. More in detail, Phaser is in the termination state when the method onAdvance() returns the true value. If you override that method, you can change the default behavior. When Phaser is on this state, the synchronization method arriveAndAwaitAdvance() returns immediately without doing any synchronization operation.
A notable feature of the Phaser class is that you haven't had to control any exception from the methods related with the phaser. Unlike other synchronization utilities, threads that are sleeping in a phaser don't respond to interruption events and don't throw an InterruptedException exception. There is only one exception that is explained in the There's more section below.

The following screenshot shows the results of one execution of the example:

How it works...
It shows the first two phases of the execution. You can see how the Apps thread finishes its execution in phase two because its results list is empty. When you execute the example, you will see how some threads finish a phase before the rest, but they wait until all have finished one phase before continuing with the rest.

There's more...

The Phaser class provides other methods related to the change of phase. These methods are as follows:

arrive(): This method notifies the phaser that one participant has finished the actual phase, but it should not wait for the rest of the participants to continue with its execution. Be careful with the utilization of this method, because it doesn't synchronize with other threads.
awaitAdvance(intphase): This method puts the current thread to sleep until all the participants of the phaser have finished the current phase of the phaser, if the number we pass as the parameter is equal to the actual phase of the phaser. If the parameter and the actual phase of the phaser aren't equal, the method returns immediately.
awaitAdvanceInterruptibly(intphaser): This method is equal to the method explained earlier, but it throws an InterruptedException exception if the thread that is sleeping in this method is interrupted.
Registering participants in the Phaser
When you create a Phaser object, you indicate how many participants will have that phaser. But the Phaser class has two methods to increment the number of participants of a phaser. These methods are as follows:

register(): This method adds a new participant to Phaser. This new participant will be considered as unarrived to the actual phase.
bulkRegister(intParties): This method adds the specified number of participants to the phaser. These new participants will be considered as unarrived to the actual phase.
The only method provided by the Phaser class to decrement the number of participants is the arriveAndDeregister() method that notifies the phaser that the thread has finished the actual phase, and it doesn't want to continue with the phased operation.

Forcing the termination of a Phaser
When a phaser has zero participants, it enters a state denoted by Termination. The Phaser class provides forceTermination() to change the status of the phaser and makes it enter in the Termination state independently of the number of participants registered in the phaser. This mechanism may be useful when one of the participants has an error situation, to force the termination of the phaser.

When a phaser is in the Termination state, the awaitAdvance() and arriveAndAwaitAdvance() methods immediately return a negative number, instead of a positive one that returns normally. If you know that your phaser could be terminated, you should verify the return value of those methods to know if the phaser has been terminated.

See also

The Monitoring a Phaser recipe in Chapter 8, Testing Concurrent Applications