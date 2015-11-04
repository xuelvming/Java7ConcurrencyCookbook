Synchronizing tasks in a common point

The Java concurrency API provides a synchronizing utility that allows the synchronization of two or more threads in a determined point. It's the CyclicBarrier class. This class is similar to the CountDownLatch class explained in the Waiting for multiple concurrent events recipe in this chapter, but presents some differences that make them a more powerful class.

The CyclicBarrier class is initialized with an integer number, which is the number of threads that will be synchronized in a determined point. When one of those threads arrives to the determined point, it calls the await() method to wait for the other threads. When the thread calls that method, the CyclicBarrier class blocks the thread that is sleeping until the other threads arrive. When the last thread calls the await() method of the CyclicBarrier class, it wakes up all the threads that were waiting and continues with its job.

One interesting advantage of the CyclicBarrier class is that you can pass an additional Runnable object as an initialization parameter, and the CyclicBarrier class executes this object as a thread when all the threads have arrived to the common point. This characteristic makes this class adequate for the parallelization of tasks using the divide and conquer programming technique.

In this recipe, you will learn how to use the CyclicBarrier class to synchronize a set of threads in a determined point. You will also use a Runnable object that will execute after all the threads have arrived to that point. In the example, you will look for a number in a matrix of numbers. The matrix will be divided in subsets (using the divide and conquer technique), so each thread will look for the number in one subset. Once all the threads have finished their job, a final task will unify the results of them.

Getting ready

The example of this recipe has been implemented using the Eclipse IDE. If you use Eclipse or other IDE such as NetBeans, open it and create a new Java project.

How to do it...

Follow these steps to implement the example:

We're going to start the example by implementing two auxiliary classes. First, create a class named MatrixMock. This class will generate a random matrix of numbers between one and 10 where the threads are going to look for a number.
public class MatrixMock {
Declare a privateint matrix named data.
  private int data[][];
Implement the constructor of the class. This constructor will receive the number of rows of the matrix, the length of each row, and the number we are going to look for as parameters. All the three parameters are of type int.
  public MatrixMock(int size, int length, int number){
Initialize the variables and objects used in the constructor.
    int counter=0;
    data=new int[size][length];
    Random random=new Random();
Fill the matrix with random numbers. Each time you generate a number, compare it with the number you are going to look for. If they are equal, increment the counter.
    for (int i=0; i<size; i++) {
      for (int j=0; j<length; j++){
        data[i][j]=random.nextInt(10);
        if (data[i][j]==number){
          counter++;
        }
      }
    }
Finally, print a message in the console, which shows the number of occurrences of the number you are going to look for in the generated matrix. This message will be used to check that the threads get the correct result.
    System.out.printf("Mock: There are %d ocurrences of number in generated data.\n",counter,number);
Implement the getRow() method. This method receives an int parameter with the number of a row in the matrix and returns the row if it exists, and returns null if it doesn't exist.
  public int[] getRow(int row){
    if ((row>=0)&&(row<data.length)){
      return data[row];
    }
    return null;
  }
Now, implement a class named Results. This class will store, in an array, the number of occurrences of the searched number in each row of the matrix.
public class Results {
Declare a private int array named data.
  private int data[];
Implement the constructor of the class. This constructor receives an integer parameter with the number of elements of the array.
  public Results(int size){
    data=new int[size];
  }
Implement the setData() method. This method receives a position in the array and a value as parameters, and establishes the value of that position in the array.
  public void  setData(int position, int value){
    data[position]=value;
  }
Implement the getData() method. This method returns the array with the array of the results.
  public int[] getData(){
    return data;
  }
Now that you have the auxiliary classes, it's time to implement the threads. First, implement the Searcher class. This class will look for a number in determined rows of the matrix of random numbers. Create a class named Searcher and specify that it implements the Runnable interface.
public class Searcher implements Runnable {
Declare two private int attributes named firstRow and lastRow. These two attributes will determine the subset of rows where this object will look for.
  private int firstRow;

  private int lastRow;
Declare a private MatrixMock attribute named mock.
  private MatrixMock mock;
Declare a private Results attribute named results.
  private Results results;
Declare a private int attribute named number that will store the number we are going to look for.
  private int number;
Declare a CyclicBarrier object named barrier.
  private final CyclicBarrier barrier;
Implement the constructor of the class that initializes all the attributes declared before.
  public Searcher(int firstRow, int lastRow, NumberMock mock, Results results, int number, CyclicBarrier barrier){
    this.firstRow=firstRow;
    this.lastRow=lastRow;
    this.mock=mock;
    this.results=results;
    this.number=number;
    this.barrier=barrier;
  }
Implement the run() method that will search for the number. It uses an internal variable called counter that will store the number of occurrences of the number in each row.
   @Override
  public void run() {
    int counter;
Print a message in the console with the rows assigned to this object.
    System.out.printf("%s: Processing lines from %d to %d.\n",Thread.currentThread().getName(),firstRow,lastRow);
Process all the rows assigned to this thread. For each row, count the number of occurrences of the number you are searching for and store this number in the corresponding position of the Results object.
    for (int i=firstRow; i<lastRow; i++){
      int row[]=mock.getRow(i);
      counter=0;
      for (int j=0; j<row.length; j++){
        if (row[j]==number){
          counter++;
        }
      }
      results.setData(i, counter);
    }
Print a message in the console to indicate that this object has finished searching.
    System.out.printf("%s: Lines processed.\n",Thread.currentThread().getName());
Call the await() method of the CyclicBarrier object and add the necessary code to process the InterruptedException and BrokenBarrierException exceptions that this method can throw.
    try {
      barrier.await();
    } catch (InterruptedException e) {
      e.printStackTrace();
    } catch (BrokenBarrierException e) {
      e.printStackTrace();
    }
Now, implement the class that calculates the total number of occurrences of the number in the matrix. It uses the Results object that stores the number of appearances of the number in each row of the matrix to make the calculation. Create a class named Grouper and specify that it implements the Runnable interface.
public class Grouper implements Runnable {
Declare a private Results attribute named results.
  private Results results;
Implement the constructor of the class that initializes the Results attribute.
  public Grouper(Results results){
    this.results=results;
  }
Implement the run() method that will calculate the total number of occurrences of the number in the array of results.
   @Override
  public void run() {
Declare an int variable and write a message to the console to indicate the start of the process.
    int finalResult=0;
    System.out.printf("Grouper: Processing results...\n");
Get the number of occurrences of the number in each row using the getData() method of the results object. Then, process all the elements of the array and add their value to the finalResult variable.
    int data[]=results.getData();
    for (int number:data){
      finalResult+=number;
    }
Print the result in the console.
    System.out.printf("Grouper: Total result: %d.\n",finalResult);
Finally, implement the main class of the example by creating a class named Main and add the main() method to it.
public class Main {

  public static void main(String[] args) {
Declare and initialize five constants to store the parameters of the application.
    final int ROWS=10000;
    final int NUMBERS=1000;
    final int SEARCH=5;
    final int PARTICIPANTS=5;
    final int LINES_PARTICIPANT=2000;
Create a MatrixMock object named mock. It will have 10,000 rows of 1000 elements. Now, you are going to search for the number five.
    MatrixMock mock=new MatrixMock(ROWS, NUMBERS,SEARCH);
Create a Results object named results. It will have 10,000 elements.
    Results results=new Results(ROWS);
Create a Grouper object named grouper.
    Grouper grouper=new Grouper(results);
Create a CyclicBarrier object called barrier. This object will wait for five threads. When this thread finishes, it will execute the Grouper object created previously.
    CyclicBarrier barrier=new CyclicBarrier(PARTICIPANTS,grouper);
Create five Searcher objects, five threads to execute them, and start the five threads.
    Searcher searchers[]=new Searcher[PARTICIPANTS];
    for (int i=0; i<PARTICIPANTS; i++){
      searchers[i]=new Searcher(i*LINES_PARTICIPANT, (i*LINES_PARTICIPANT)+LINES_PARTICIPANT, mock, results, 5,barrier);
      Thread thread=new Thread(searchers[i]);
      thread.start();
    }
    System.out.printf("Main: The main thread has finished.\n");
How it works...

The following screenshot shows the results of an execution of this example:

How it works...
The problem resolved in the example is simple. We have a big matrix of random integer numbers and you want to know the total number of occurrences of a number in this matrix. To get a better performance, we use the divide and conquer technique. We divide the matrix in five subsets and use a thread to look for the number in each subset. These threads are objects of the Searcher class.

We use a CyclicBarrier object to synchronize the completion of the five threads and to execute the Grouper task to process the partial results, and calculate the final one.

As we mentioned earlier, the CyclicBarrier class has an internal counter to control how many threads have to arrive to the synchronization point. Each time a thread arrives to the synchronization point, it calls the await() method to notify the CyclicBarrier object that has arrived to its synchronization point. CyclicBarrier puts the thread to sleep until all the threads arrive to their synchronization point.

When all the threads have arrived to their synchronization point, the CyclicBarrier object wakes up all the threads that were waiting in the await() method and, optionally, creates a new thread that executes a Runnable object passed as the parameter in the construction of CyclicBarrier (in our case, a Grouper object) to do additional tasks.

There's more...

The CyclicBarrier class has another version of the await() method:

await(longtime,TimeUnitunit): The thread will be sleeping until it's interrupted; the internal counter of CyclicBarrier arrives to 0 or specified time passes. The TimeUnit class is an enumeration with the following constants: DAYS, HOURS, MICROSECONDS, MILLISECONDS, MINUTES, NANOSECONDS, and SECONDS.
This class also provides the getNumberWaiting() method that returns the number of threads that are blocked in the await() method, and the getParties() method that returns the number of tasks that are going to be synchronized with CyclicBarrier.

Resetting a CyclicBarrier object
The CyclicBarrier class has some points in common with the CountDownLatch class, but they also have some differences. One of the most important differences is that a CyclicBarrier object can be reset to its initial state, assigning to its internal counter the value with which it was initialized.

This reset operation can be done using the reset() method of the CyclicBarrier class. When this occurs, all the threads that were waiting in the await() method receive a BrokenBarrierException exception. This exception was processed in the example presented in this recipe by printing the stack trace, but in a more complex application, it could perform some other operation, such as restarting their execution or recovering their operation at the point it was interrupted.

Broken CyclicBarrier objects
A CyclicBarrier object can be in a special state denoted by broken. When there are various threads waiting in the await() method and one of them is interrupted, this thread receives an InterruptedException exception, but the other threads that were waiting receive a BrokenBarrierException exception and CyclicBarrier is placed in the broken state.

The CyclicBarrier class provides the isBroken() method, then returns true if the object is in the broken state; otherwise it returns false.

See also

The Waiting for multiple concurrent events recipe in Chapter 3, Thread Synchronization Utilities
