Arranging independent attributes in synchronized classes

When you use the synchronized keyword to protect a block of code, you must pass an object reference as a parameter. Normally, you will use the this keyword to reference the object that executes the method, but you can use other object references. Normally, these objects will be created exclusively with this purpose. For example, if you have two independent attributes in a class shared by multiple threads, you must synchronize the access to each variable, but there is no problem if there is one thread accessing one of the attributes and another thread accessing the other at the same time.

In this recipe, you will learn how to resolve this situation's programming with an example that simulates a cinema with two screens and two ticket offices. When a ticket office sells tickets, they are for one of the two cinemas, but not for both, so the numbers of free seats in each cinema are independent attributes.

Getting ready

The example of this recipe has been implemented using the Eclipse IDE. If you use Eclipse or other IDE such as NetBeans, open it and create a new Java project.

How to do it...

Follow these steps to implement the example:

Create a class called Cinema and add to it two long attributes named vacanciesCinema1 and vacanciesCinema2.
public class Cinema {

  private long vacanciesCinema1;
  private long vacanciesCinema2;
Add to the Cinema class two additional Object attributes named controlCinema1 and controlCinema2.
  private final Object controlCinema1, controlCinema2;
Implement the constructor of the Cinema class that initializes all the attributes of the class.
  public Cinema(){
    controlCinema1=new Object();
    controlCinema2=new Object();
    vacanciesCinema1=20;
    vacanciesCinema2=20;
  }
Implement the sellTickets1() method that is called when some tickets for the first cinema are sold. It uses the controlCinema1 object to control the access to the synchronized block of code.
  public boolean sellTickets1 (int number) {
    synchronized (controlCinema1) {
      if (number<vacanciesCinema1) {
        vacanciesCinema1-=number;
        return true;
      } else {
        return false;
      }
    }
  }
Implement the sellTickets2() method that is called when some tickets for the second cinema are sold. It uses the controlCinema2 object to control the access to the synchronized block of code.
  public boolean sellTickets2 (int number){
    synchronized (controlCinema2) {
      if (number<vacanciesCinema2) {
        vacanciesCinema2-=number;
        return true;
      } else {
        return false;
      }
    }
  }
Implement the returnTickets1() method that is called when some tickets for the first cinema are returned. It uses the controlCinema1 object to control the access to the synchronized block of code.
  public boolean returnTickets1 (int number) {
    synchronized (controlCinema1) {
      vacanciesCinema1+=number;
      return true;
    }
  }
Implement the returnTickets2() method that is called when some tickets for the second cinema are returned. It uses the controlCinema2 object to control the access to the synchronized block of code.
  public boolean returnTickets2 (int number) {
    synchronized (controlCinema2) {
      vacanciesCinema2+=number;
      return true;
    }
  }
Implement another two methods that return the number of vacancies in each cinema.
  public long getVacanciesCinema1() {
    return vacanciesCinema1;
  }

  public long getVacanciesCinema2() {
    return vacanciesCinema2;
  }
Implement the class TicketOffice1 and specify that it implements the Runnable interface.
public class TicketOffice1 implements Runnable {
Declare a Cinema object and implement the constructor of the class that initializes that object.
  private Cinema cinema;

  public TicketOffice1 (Cinema cinema) {
    this.cinema=cinema;
  }
Implement the run() method that simulates some operations over the two cinemas.
  @Override
   public void run() {
    cinema.sellTickets1(3);
    cinema.sellTickets1(2);
    cinema.sellTickets2(2);
    cinema.returnTickets1(3);
    cinema.sellTickets1(5);
    cinema.sellTickets2(2);
    cinema.sellTickets2(2);
    cinema.sellTickets2(2);
  }
Implement the class TicketOffice2 and specify that it implements the Runnable interface.
public class TicketOffice2 implements Runnable {
Declare a Cinema object and implement the constructor of the class that initializes that object.
  private Cinema cinema;

  public TicketOffice2(Cinema cinema){
    this.cinema=cinema;
  }
Implement the run() method that simulates some operations over the two cinemas.
  @Override
  public void run() {
    cinema.sellTickets2(2);
    cinema.sellTickets2(4);
    cinema.sellTickets1(2);
    cinema.sellTickets1(1);
    cinema.returnTickets2(2);
    cinema.sellTickets1(3);
    cinema.sellTickets2(2);
    cinema.sellTickets1(2);
  }
Implement the main class of the example by creating a class called Main and add to it the main() method.
public class Main {

  public static void main(String[] args) {
Declare and create a Cinema object.
    Cinema cinema=new Cinema();
Create a TicketOffice1 object and Thread to execute it.
    TicketOffice1 ticketOffice1=new TicketOffice1(cinema);
    Thread thread1=new Thread(ticketOffice1,"TicketOffice1");
Create a TicketOffice2 object and Thread to execute it.
    TicketOffice2 ticketOffice2=new TicketOffice2(cinema);
    Thread thread2=new Thread(ticketOffice2,"TicketOffice2");
Start both threads.
    thread1.start();
    thread2.start();
Wait for the completion of the threads.
    try {
      thread1.join();
      thread2.join();
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
Write to the console the vacancies of the two cinemas.
    System.out.printf("Room 1 Vacancies: %d\n",cinema.getVacanciesCinema1());
    System.out.printf("Room 2 Vacancies: %d\n",cinema.getVacanciesCinema2());
How it works...

When you use the synchronized keyword to protect a block of code, you use an object as a parameter. JVM guarantees that only one thread can have access to all the blocks of code protected with that object (note that we always talk about objects, not about classes).

Note
In this example, we have an object that controls access to the vacanciesCinema1 attribute, so only one thread can modify this attribute each time, and another object controls access to the vacanciesCinema2 attribute, so only one thread can modify this attribute each time. But there may be two threads running simultaneously, one modifying the vacancesCinema1 attribute and the other one modifying the vacanciesCinema2 attribute.

When you run this example, you can see how the final result is always the expected number of vacancies for each cinema. In the following screenshot, you can see the results of an execution of the application:

How it works...
There's more...

There are other important uses of the synchronize keyword. See the See also section for other recipes that explain the use of this keyword.
