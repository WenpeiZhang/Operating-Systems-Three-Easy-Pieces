6.1 The first known correct software solution to the critical-section problem for two processes was developed by **Dekker**. The two processes, P0 and P1, share the following variables:
```c
boolean flag[2]; /* initially false */
int turn;
```
The structure of process Pi (i == 0 or 1) is shown in Figure 6.25; the other process is Pj (j == 1 or 0). Prove that the algorithm satisfies all three requirements for the critical-section problem.

Answer: 

*This algorithm satisfies the three conditions of mutual exclusion.*

*(1) Mutual exclusion is ensured through the use of the flag and turn variables. If both processes set their flag to true, only one will succeed. Namely, the process whose turn it is. The waiting process can only enter its critical section when the other process updates the value of turn.*

*(2) Progress is provided, again through the flag and turn variables. This algorithm does not provide strict alternation. Rather, if aprocess wishes to access their critical section, it can set their flag vari- able to true and enter their critical section. It only sets turn to the value of the other process upon exiting its critical section. If this process wishes to enter its critical section again - before the other process - it repeats the process of entering its critical section and setting turn to the other process upon exiting.*

*(3) Bounded waiting is preserved through the use of the turn variable. Assume two processes wish to enter their respec- tive critical sections. They both set their value of flag to true, however only the thread whose turn it is can proceed, the other thread waits. If bounded waiting were not preserved, it would therefore be possible that the waiting process would have to wait indefinitely while the first process repeatedly entered - and exited - its critical section. However, Dekker’s algorithm has a process set the value of turn to the other pro- cess, thereby ensuring that the other process will enter its critical section next.*

6.7 Describe how the **Swap()** instruction can be used to provide **mutual exclusion** that satisfies the **bounded-waiting requirement**.
```c
 do {
        waiting[i] = TRUE;
        key = TRUE;
        while (waiting[i] && key)
            key = Swap(&lock, &key);
            waiting[i] = FALSE;
              /* critical section */
            j = (i+1) % n;
            while ((j != i) && !waiting[j])
                j = (j+1) % n;
            if (j == i)
                lock = FALSE;
            else
                waiting[j] = FALSE;
              /* remainder section */
          } while (TRUE);

```
6.8 Servers can be designed to limit the number of open connections. For example, a server may wish to have only N socket connections at any point in time. As soon as N connections are made, the server will not accept another incoming connection until an existing connection is re- leased. Explain how semaphores can be used by a server to limit the number of concurrent connections.

*Answer: **A semaphore is initialized to the number of allowable open socket connections.** When a connection is accepted, the *acquire()* method is called, when a connection is released, the *release()* method is called. If the system reaches the number of allowable socket connections, subsequent calls to *acquire()* will block until an existing connection is terminated and the release method is invoked.*

6.9 Show that, if the *wait()* and *signal()* semaphore operations are not executed atomically, then mutual exclusion may be violated.

*Answer: A wait operation atomically decrements the value associated with a semaphore. If two wait operations are executed on a semaphore when its value is 1, if the two operations are not performed atomically, then it is possible that both operations might proceed to decrement the semaphore value thereby violating mutual exclusion.*

6.10 Show how to implement the *wait()* and *signal()* semaphore operations in multiprocessor environments using the *TestAndSet()* instruction. The solution should exhibit minimal busy waiting.

*Answer: Here is the pseudocode for implementing the operations:*
```c
int guard = 0;
int semaphore value = 0;

wait()
{
    while(TestAndSet(&guard) == 1);
    if(semaphore_value == 0){
        atomically add process to a queue of processes
        waiting for the semaphore and set guard to 0;
        }else{
        semaphore_value--;
        guard = 0;
        }
}

signal()
{
    while(TestAndSet(&guard) == 1);
    if (semaphore value == 0 &&
                     there is a process on the wait queue)
    wake up the first process in the queue of waiting processes
    else
        semaphore_value++;
    guard = 0;
}
```
6.11 ***The Sleeping-Barber Problem***. A barbershop consists of a waiting room with *n* chairs and a barber room with one barber chair. If there are no customers to be served, the barber goes to sleep. If a customer enters the barbershop and all chairs are occupied, then the customer leaves the shop. If the barber is busy but chairs are available, then the customer sits in one of the free chairs. If the barber is asleep, the customer wakes up the barber. Write a program to coordinate the barber and the customers.

Answer:
(1) BarberShop.java
```c


//This class defines the Barber Shop Scenario.
//This scenario allows N customers to enter it.
//It also contains a number of chairs that allow the customers to wait in.

public class BarberShop
{
    private int chairNum;
    private int barber;
    private int chairState[];
    static final int FULL = -1;
    static final int EMPTY = 0;
    static final int OCUPIED = 1;
    static final int SLEEPING = 2;
    static final int DONE = 3;
    static final int WAITED = 4;
    
/* ============================================================
Construct a new Barber Shop scenario for customers to get haircuts at. Set the barber shop to have N number of chairs to wait in.  Then make all the chairs to be initially empty.  Also set the barber to be asleep so that when his first customer comes in he can wake him up.
@param pChairNum The number of waiting chairs in this barber shop 
==================================================================*/
    public BarberShop(int pChairNum)
    {
        chairNum = pChairNum;
        chairState = new int[chairNum];
        barber = SLEEPING;
        
        //initialize every chair in the waiting room to be empty
        for(int i = 0; i < chairNum; i++)
            chairState[i] = EMPTY;
    }
    
/* ============================================================
This method is called when a customer sees that the Barber is busy. This means that the customer must wait for the barber.  Therefore, find a chair in the waiting room.  When the customer finds a chair they set its state to OCUPIED (so there's only one customer per chair) and return true. Otherwise, there are no chairs available so return false.
@param pCustomer The customer wanting to find a chair to wait in.
@return boolean True if chair is found; False otherwise.
============================================================ */
    private boolean findChair(int pCustomer) 
    {
        //try to find a chair to wait in
        int test = getFirstEmptyChair();
        
        //if barber shop is full return false
        if(test == FULL)
            return false;
        //otherwise sit down in this chair
        else
            chairState[test] = OCUPIED;
        
        return true;
    }
    
/* ============================================================
This method is called by the Customer thread to get a haircut.  IF the barber's asleep then the customer wakes him up and the customer gets their haircut.  IF the barber's already busy then the customer tries to find a chair to wait in.  However, if there are no chairs and the barbers busy, then the shop is full, so customer will then leave.  If the barber's state is not sleeping or ocupied, then he can take the customer immediately.
This solution doesn't prevent starvation for the waiting customers.  If A customer finnaly gets notified to leave the waiting state they will be the next haircut.  If another customer enters the shop at the same time, then that customer will always have to wair for existing customers to  be serviced.
@param customer The customer wanting to get a haircut
@return int The state of the Barber Shop and/or barber
=========================================================== */

    public synchronized int getHairCut(int customer)
    {
        //if the barber's sleeping, then wake him up and tell him to get to work
        if(barber == SLEEPING)
        {
            barber = OCUPIED;
            return SLEEPING;
        }
        //the barber's busy try to wait for him in the waiting room
        else if(barber == OCUPIED)
        {
            boolean test = findChair(customer);
            
            //if barber shop is full return full and get out.
            if(test == false)
                return FULL;
            else
            {
                //wait as long as the barber is busy
                while(barber == OCUPIED)
                {
                    try{ this.wait(); }
                    catch(InterruptedException e)
                    {}
                }  
                
//waiting customer will get to be the next haircut scheduled.  Therefore they
//stand up and give up their chair.  This is why a chair gets reset to empty.
                for(int i = 0; i < chairNum; i++)
                {
                    if(chairState[i] == OCUPIED)
                    {
                        chairState[i] = EMPTY;
                        break;
                    }
                } 
                
//set barber to ocupied since this waiting customer will get next haircut.
                barber = OCUPIED;
                return WAITED;
            }           
        }
//barber's done.  This customer can get their haircut immediately
        else
        {
            barber = OCUPIED;
            return DONE;
        }
    }
    
/* ============================================================
This method is called when the customer has recieved their haircut and they leave the shop.  They first check to see if anyone else is waiting in the shop.  If there isn't anyone, then they were the last customer.This means the barber must go back to sleep.  Otherwise poeple are waiting, so just set barber's state to done.  Then notify anyone waiting.
@param customer The customer finished with haircut and leaving shop
@return void
============================================================ */
    public synchronized void leaveBarberShop(int customer)
    {
        boolean test = isAnyoneWaiting();        
        if(test == true)
            barber = DONE;
        else
            barber = SLEEPING;
        //notify only one waiting customer (if any exist)
        //this helps on performance of the program.
        notify();
    }
        
/* ============================================================
Find the first empty chair in the waiting room and return it.  If no chairs are found then all chairs are ocupied by a waiting customer.
@return int The number of the empty chair; FULL otherwise
============================================================ */
    private int getFirstEmptyChair()
    {
        //if an empty chair is found return it
        for(int i = 0; i < chairNum; i++)
        {
            if(chairState[i] == EMPTY)
                return i;
        }
        //all chairs are occupied so return FULL
        return FULL;
    }
    
/* ============================================================
Test to see if the waiting room has any customers in it.
@return boolean True if there are customers waiting; False if empty
============================================================ */
    private boolean isAnyoneWaiting()
    {
        //see if anyone is in a chair waiting
        for(int i = 0; i < chairNum; i++)
        {
            if(chairState[i] == OCUPIED)
                return true;
        }
        //couldn't find anyone waiting 
        return false;
    }
}
```
(2) Customer.java
```c
//This class creates a customer who is wanting a haircut.
//Once the haircut is received the customer then has finished their task 
//and they have to be re-instantiated if they want another haircut.
import java.util.*;

public class Customer implements Runnable
{
    private BarberShop shop;
    private int customer;
    private int HAIRCUT_TIME = 5;
    
// Construct a new customer with their unique identity
//and give them a barber shop to enter for their haircut.
    //@param pCustomer This customer's identifier.
//@param pShop The barber shop they will get a haircut in.
    public Customer(int pCustomer, BarberShop pShop)
    {
        shop = pShop;
        customer = pCustomer;
    }
    
    /**
     * The run method for the customer thread tries to get the customer a
     * haircut.  
    */
    public void run()
    {
        int sleeptime = (int) (HAIRCUT_TIME * Math.random() );
        System.out.println("ENTERING SHOP: Customer [" + customer + "] entering barber shop for haircut.");
        int  test = BarberShop.OCUPIED;
        //test for this customer's haircut posibility
        test = shop.getHairCut(customer);
        //if the barber was busy, then this customer has waited in the waiting room.
        //This waiting customer will now get the next haircut.  Entering threads will
        //have to wait for the existing customers to be serviced.  As far as which
        //waiting thread will get serviced next is up to the JVM.
        if(test == BarberShop.WAITED)
            System.out.println("Barber's busy: Customer [" + customer + "] has waited and now wants haircut.");
        //otherwise, no one in barber shop, wake up barber and get haircut
        else if (test == BarberShop.SLEEPING)
            System.out.println("Barber's asleep: Customer [" + customer + "] is waking him up and getting haircut.");
        //this barber shop is full.  Therefore leave and never return.
        else if (test == BarberShop.FULL)
        {
            System.out.println("Barber Shop full: Customer [" + customer + "] is leaving shop.");
            return;
        }
        //Barber's ready to take this customer right away for a haircut.
        else
            System.out.println("HAIRCUT: Customer [" + customer + "] is getting haircut.");               
        //customer is now getting their haircut for an amount of time
        SleepUtilities.nap();
        //haircut finished.  Leave the shop and notify anyone waiting.
        System.out.println("LEAVING SHOP: Customer [" + customer + "] haircut finished: leaving shop.");
        shop.leaveBarberShop(customer);
    }
}
```
(3) SleepUtilities.java
```c
//Utilities for causing a thread to sleep.
//Note, we should be handling interrupted exceptions
//but choose not to do so for code clarity.
public class SleepUtilities
{
	// Nap between zero and NAP_TIME seconds.
	public static void nap() {
		nap(NAP_TIME);
	}

	//Nap between zero and duration seconds.
	public static void nap(int duration) {
        	int sleeptime = (int) (NAP_TIME * Math.random() );
        	try { Thread.sleep(sleeptime*1000); }
        	catch (InterruptedException e) {}
	}
	private static final int NAP_TIME = 5;
}
```
(4) CreateBarberShopTest.java
```c
//Test Stub for the BarberShop\Customer problem. 
//This class adds customers to a constructed barber shop indefinately.

public class CreateBarberShopTest implements Runnable
{
    static final private int WAIT_TIME = 3;
    static public void main(String [] args)
    {
        new Thread(new CreateBarberShopTest()).start();
    }
    
    public void run()
    { 
        //create the barber shop
        BarberShop newShop = new BarberShop(15);
        int customerID = 1;
        
        //add the specifid number of threads to shop
        while(customerID <= 10000)
        {
            //add customers to the barber shop
            new Thread(new Customer(customerID, newShop)).start();
            customerID++;
            //wait this amount of time before adding another customer to shop
            SleepUtilities.nap();
        }      
    }
}
```
