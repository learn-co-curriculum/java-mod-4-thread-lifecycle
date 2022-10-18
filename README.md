# Thread States and Lifecycle

## Learning Goals

- Explain the various thread states.
- Explain a simplified thread lifecycle.
- Use `sleep()` and `join()` methods to manage thread state.

## Introduction

A thread goes through different states from its creation to termination. These
state changes can be caused by the programmer’s code or by the operating
system’s events. We’ll first take a look at the different states a thread can be
in and then look at a simplified lifecycle.

## Thread States

Thread states are represented by the `Thread.State` enum and a current thread’s
state can be examined using the `getState()` instance method on a thread.

`public Thread.State getState()`

There are six possible values of thread state:

- `NEW`: a new thread instance has just been created but not yet started.
- `RUNNABLE`: the thread is executing in the JVM but may be waiting for operating system resources.
- `BLOCKED`: the thread is waiting for a monitor lock.
- `WAITING`: the thread is waiting for another thread to perform a particular action.
- `TIMED_WAITING`: the thread is waiting for the specified amount of time.
- `TERMINATED`: the thread exited. 

Once terminated, a thread cannot go back to a runnable state.

## Thread Lifecycle

The states in the `Thread.State` are from the JVM’s point of view. The actual
states are more complex. For example, the `RUNNABLE` state can describe a thread
that is ready to run or is running. Here’s a simplified diagram of a
thread lifecycle:

![Thread lifecycle](https://curriculum-content.s3.amazonaws.com/java-mod-5/thread-lifecycle.png)

When a thread is initialized, it is in the `NEW` state and remains in that state
until the `start()` method is called. When the thread is
started it goes into the `RUNNABLE` state waiting for the thread scheduler to
allocate it resources for executing the instructions in the `run()` method.
The thread switches between “Ready to Run” and “Running" based on resource availability.

The waiting states means that a thread’s execution is paused
while it waits for another thread, time duration, or monitor lock. A thread must be moved
from this state to a “Ready to Run” state to continue execution.

The `TERMINATED` state means the `run()` method has finished
execution or an uncaught exception is thrown.

Let’s look at some of these states with example code.

```java
public class Example {
    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName() + " " + Thread.currentThread().getState());  //main RUNNABLE

        Thread t0 = new Thread( () -> System.out.println(Thread.currentThread().getName() + " hello") );  //Thread-0 hello
        System.out.println(t0.getName() + " " + t0.getState());  //Thread-0 NEW

        Thread t1 = new Thread( () -> System.out.println(Thread.currentThread().getName() + " howdy") ); //Thread-1 howdy
        System.out.println(t1.getName() + " " + t1.getState());  //Thread-1 NEW

        t0.start();
        System.out.println(t0.getName() + " " + t0.getState());  //Thread-0 RUNNABLE

        t1.start();
        System.out.println(t1.getName() + " " + t1.getState());  //Thread-1 RUNNABLE

        System.out.println(Thread.currentThread().getName() + " goodbye"); //main goodbye
    }
}
```

The `main` method creates two new threads, then calls `start()`
on each thread. The output shows a new thread begins in the `NEW` state
and moves into the `RUNNABLE` state after the `start()` method is called.

```text
main RUNNABLE
Thread-0 NEW
Thread-1 NEW
Thread-0 RUNNABLE
Thread-1 RUNNABLE
main goodbye
Thread-0 hello
Thread-1 howdy
```

Notice the last print statement in the `main` method that prints "main goodbye"
may execute before the other threads execute their print statements
("Thread-0 hello" and "Thread-1 howdy").
While the statement `t0.start()` starts the thread, it does not result in
immediate execution of the `run()` method (the lambda expression).
The situation is similar with `t1.start()`,
resulting in a possible delay in execution.
Because the `main` thread is in the `RUNNABLE` state,
the scheduler may allow the thread to execute to completion
prior to running the other threads.

##  Sleeping threads

`public static void sleep(long millis) throws InterruptedException`

We can invoke the `sleep()` method on a thread to temporarily stop
the thread's execution for at least the specified number of milliseconds.
The sleeping thread's state changes from `RUNNABLE` to `TIMED_WAITING` and the
scheduler picks another thread to execute.  The `sleep()` method
may throw an exception, so the call is surrounded in with `try-catch` statements.


```java
public class Example {
    public static void main(String[] args)  {
        System.out.println(Thread.currentThread().getName() + " " + Thread.currentThread().getState());  //main RUNNABLE

        Thread t0 = new Thread( () -> System.out.println(Thread.currentThread().getName() + " hello") );  //Thread-0 hello
        System.out.println(t0.getName() + " " + t0.getState());  //Thread-0 NEW

        Thread t1 = new Thread( () -> System.out.println(Thread.currentThread().getName() + " howdy") ); //Thread-1 howdy
        System.out.println(t1.getName() + " " + t1.getState());  //Thread-1 NEW

        t0.start();
        System.out.println(t0.getName() + " " + t0.getState());  //Thread-0 RUNNABLE

        t1.start();
        System.out.println(t1.getName() + " " + t1.getState());  //Thread-1 RUNNABLE

        try {
            Thread.sleep(1000);  //changes state from RUNNABLE to TIMED-WAITING to allow another thread to run
        }
        catch(InterruptedException e) {
            System.out.println(e.getMessage());
        }

        System.out.println(Thread.currentThread().getName() + " goodbye"); //main goodbye
    }
}
```

The updated code puts the `main` thread to sleep
for 1 second, which should be enough time for the other threads
to run to completion before "main goodbye" is printed:

```text
main RUNNABLE
Thread-0 NEW
Thread-1 NEW
Thread-0 RUNNABLE
Thread-1 RUNNABLE
Thread-1 howdy
Thread-0 hello
main goodbye
```

##  Joining threads

Instead of waiting for a specified amount of time, the `join()` method
forces the current thread to wait until another thread terminates.

`public final void join() throws InterruptedException`

For example, `t0.join()` forces the `main` thread to wait until the
thread referenced by `t0` finishes executing:

```java
public class Example {
    
    public static void main(String[] args)  {

        System.out.println(Thread.currentThread().getName() + " " + Thread.currentThread().getState());  //main RUNNABLE

        Thread t0 = new Thread( () -> System.out.println(Thread.currentThread().getName() + " hello") );  //Thread-0 hello
        System.out.println(t0.getName() + " " + t0.getState());  //Thread-0 NEW

        Thread t1 = new Thread( () -> System.out.println(Thread.currentThread().getName() + " howdy") ); //Thread-1 howdy
        System.out.println(t1.getName() + " " + t1.getState());  //Thread-1 NEW

        t0.start();
        System.out.println(t0.getName() + " " + t0.getState());  //Thread-0 RUNNABLE

        t1.start();
        System.out.println(t1.getName() + " " + t1.getState());  //Thread-1 RUNNABLE
        
        try {
            t0.join(); //main thread put in WAITING state until t0 terminates
        } catch (InterruptedException e) {
            System.out.println(e.getMessage());
        }
        
        System.out.println(t0.getName() + " " + t0.getState());  //Thread-0 TERMINATED

        System.out.println(Thread.currentThread().getName() + " goodbye"); //main goodbye
    }
}
```

The output will never print "main goodbye" before "Thread-0 hello" because the `main` thread
must wait for the thread referenced by `t0` to terminate.

```text
main RUNNABLE
Thread-0 NEW
Thread-1 NEW
Thread-0 RUNNABLE
Thread-1 RUNNABLE
Thread-1 howdy
Thread-0 hello
Thread-0 TERMINATED
main goodbye
```

## Conclusion

We’ve learned about the various states of a thread and their lifecycle. As we
learn how to synchronize data and create safe, concurrent programs, this idea
about different thread states will be helpful in understanding more advanced
concepts.

## Resources

[Java 11 Thread](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Thread.html)  
[Java 11 Thread.State](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Thread.State.html)  
