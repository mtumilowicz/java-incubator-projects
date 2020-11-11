[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

# java-incubator-projects
* references
    * [Why Continuations are Coming to Java](https://www.youtube.com/watch?v=9vupFNsND6o)
    * [Project Loom: Helping Write Concurrent Applications on the Java Platform by Ron Pressler](https://www.youtube.com/watch?v=lIq-x_iI-kc)
    * [Java, Today and Tomorrow by Mark Reinhold](https://www.youtube.com/watch?v=Csc2JRs6470)
    * [Records and Sealed Types - Coming Soon to a JVM Near You!](https://www.youtube.com/watch?v=DfNnlWqjXiI)
    * [Does Java Need Inline Types? What Project Valhalla Can Bring to Java](https://www.youtube.com/watch?v=jGjWs2xpZrY)
    * [Java Futures, 2019 Edition](https://www.youtube.com/watch?v=hryQIIasGY4)
    * [[VDT19] Java - Quo Vadis? by Tobias Hartmann](https://www.youtube.com/watch?v=149Q1Xbud2I)
    * [Java keeps throttling up! by José Paumard & Remi Forax](https://www.youtube.com/watch?v=Y-744emVGoo)

## preface
* the goal of this paper is to show main java incubator projects
* note that this paper is not updated on a daily manner, so don't hold your breath
* up-to-date: 11.11.2020

## project amber
* https://openjdk.java.net/projects/amber/
* explore and incubate smaller, productivity-oriented Java language features that have been 
accepted as candidate JEPs under the OpenJDK JEP process
* adapting to rising developer expectations
* exemplary shipped features
    * diamond operator
        ```
        List<String> list = new ArrayList<>()
        ```
    * static methods type inference
        ```
        List<String> empty = Collections.emptyList() // instead of Collections.<String>emptyList()
        ```
    * lambda args type inference
        ```
        Predicate<String> isEmpty = s -> s.length() == 0 // instead of (String s) -> s.length() == 0
        ```
    * local variable type inference
        ```
        var i = 1;
        ```
    * enhanced switch expressions
        ```
        static void howMany(int k) {
            switch (k) {
                case 1  -> System.out.println("one");
                case 2  -> System.out.println("two");
                default -> System.out.println("many");
            }
        }
        ```
* incubating
    * record
        ```
        record Point(double x, double y)
        ```
        * first-class support for modeling data-only aggregates
            * the state, the whole state and nothing but the state
        * close a possible gap in java's type system
        * language-level syntax for a common pattern
        * reduce class boilerplate
        * something like data in kotlin
    * sealed type
        * defines a closed hierarchy
    * pattern matching
    * smart casts
        ```
        if (obj instanceof String s) {
            // can use s here
        } else {
            // can't use s here
        }
        ```

## project loom
* https://openjdk.java.net/projects/loom/
* adapting to rising scale expectations
    * threads in java
        * thread scheduling is done by the OS
        * thread stack is pre-reserved
        * so far, one implementation: OS threads
        * OS threads must support all applications in all languages
            * cannot be specialized for the task, are good overall but not specialized
        * task-switching requires switch to kernel
        * ram-heavy - megabyte-scale
        * scheduling is a compromise for all usages
            * bad cache locality
    * many lightweight/virtual threads
        * small number "carrier" heavyweight/kernel threads managed by scheduler
    * the use of Thread.currentThread() and ThreadLocal is pervasive
        * without support, or with changed behaviour - little existing code would run
        * scope locals expected to supplant most used of ThreadLocal
        * since Java 5 we've encouraged people not to use the Thread API directly anyway
            * use Executor and Future, so the baggage and past API mistakes are largely inconspicuous
* main goal
    * continuations & fibers
* continuation is the ability to suspend computation and then resume it
    * example
        ```
        public class ContinuationExample {
            public static void main(String[] args) {
                var scope = new ContinuationScope("example");
                var continuation = new Continuation(scope, () -> {
                    System.out.println("hello");
                    Continuation.yield(scope);
                    System.out.println("hello again");
                });
                System.out.println("start continuation");
                continuation.run();
                System.out.println("run it again");
                continuation.run();
            }
        }
        ```
        will print
        ```
        start continuation
        hello
        run it again
        hello again
        ```
    * wraps a runnable
    * `Continuation.run()` executes the runnable
        * can stop itself with a yield
        * a call to `run()` will restart the runnable just after the `yield`
    * `yield` copy the all stack frames on the heap
        * then `run()` move some stack frames back
    * scheduling explicit (yield, run)
        * no OS context switch
    * no heap reservation
        * only store what is actually needed
* fiber = continuation + scheduler
    * scheduling is done by application not OS
    * wraps a runnable
    * scheduler using an `ExecutorService`
        * a fiber is scheduled on the thread
    * blocking calls (read, write, sleep)
        * freeze the fiber, schedule another one
        * re-scheduled
            * when a read/write/locks/etc will not block
            * maybe scheduled on another thread
* fiber vs continuation
    * fiber delegates the suspension (yield) to the continuation
        * all blocking calls internally a yield
        * the jdk code calls `run()` on the continuation when it can be rescheduled
* code like sync, works like async
    * synchronous
        * easy to read
        * fits well with language (control flow, exceptions)
        * fits well with tooling (debuggers, profilers)
        * but
            * costly to create and block
            * a limited resource
    * asynchronous (reactive)
        * scalable
        * but
            * hard to read
            * very hard to debug and profile
            * often obscures business logic

## project panama
* https://openjdk.java.net/projects/panama/
* improving and enriching the connections between the Java virtual machine and well-defined but 
"foreign" (non-Java) APIs
* better interop with native code and data
* example for target improvements
    * native function calling from JVM
    * new data layouts in JVM heap

## project valhalla
* https://openjdk.java.net/projects/valhalla/
* adapting to modern hardware
* every object in java has identity
* goal: pure data aggregator without identity
    * code like a class, works like int
* valhalla goals
    * provide denser memory layout (inline/value types)
    * specialized generics (including primitive, value types)
    * pure data aggregator without identity
        * code like a class, works like int
    * inline classes
    * jvm cleanup

## project metropolis
* big idea: java-on-java
    * reduce dependency on c/cpp/assembly and rely on self-optimization
    * reduce complexity and cost of maintenance
* first goal: replace JIT (C2)
    * currently experimenting with graalvm + substratevm
    * graal already part of the openJDK
* potentially replace other components in the future

## project portola
* java in a world of containers
* java's characteristics make it ideal for container deployment
    * safe, performant, reliable, rich ecosystem
* having java remain the first choice for deployments in the cloud