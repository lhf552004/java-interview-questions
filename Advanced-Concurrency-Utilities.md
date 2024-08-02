Advanced Concurrency Utilities in Java
Introduction

Java provides a rich set of concurrency utilities to help developers build high-performance, scalable applications. These utilities abstract away much of the complexity associated with thread management, making it easier to write efficient concurrent programs. This document introduces key high-level concurrency utilities available in the Java standard library.
1. Executor Framework

The Executor framework, introduced in Java 5, provides a simple way to decouple task submission from the mechanics of how each task will be run, including details of thread use, scheduling, etc.
Key Classes

    Executor: A simple interface with a single execute method to submit a task for execution.
    ExecutorService: Extends Executor with methods to manage lifecycle and track progress of asynchronous tasks.
    Executors: A factory class with methods to create different types of executor services (e.g., thread pools).

Example

```

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ExecutorExample {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 10; i++) {
            executor.submit(() -> {
                System.out.println("Task is running: " + Thread.currentThread().getName());
            });
        }
        executor.shutdown();
    }
}
```

2. ForkJoinPool

ForkJoinPool is designed for work-stealing algorithms, making it well-suited for divide-and-conquer tasks. It helps in parallelizing tasks by breaking them into smaller pieces.
Key Concepts

    ForkJoinTask: A lightweight task that can be forked and joined.
    RecursiveTask: A subclass of ForkJoinTask used for tasks that return a result.
    RecursiveAction: A subclass of ForkJoinTask used for tasks that do not return a result.

Example

```

import java.util.concurrent.RecursiveTask;
import java.util.concurrent.ForkJoinPool;

public class Fibonacci extends RecursiveTask<Integer> {
    private final int n;

    public Fibonacci(int n) {
        this.n = n;
    }

    @Override
    protected Integer compute() {
        if (n <= 1) return n;
        Fibonacci f1 = new Fibonacci(n - 1);
        f1.fork();
        Fibonacci f2 = new Fibonacci(n - 2);
        return f2.compute() + f1.join();
    }

    public static void main(String[] args) {
        ForkJoinPool pool = new ForkJoinPool();
        System.out.println(pool.invoke(new Fibonacci(10)));
    }
}
```
3. Future

Future represents the result of an asynchronous computation. Methods are provided to check if the computation is complete, to wait for its completion, and to retrieve the result.
Example

```

import java.util.concurrent.*;

public class FutureExample {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executor = Executors.newSingleThreadExecutor();
        Future<Integer> future = executor.submit(() -> {
            TimeUnit.SECONDS.sleep(2);
            return 123;
        });

        System.out.println("Future result: " + future.get()); // Blocks until the result is available
        executor.shutdown();
    }
}
```
4. CompletableFuture

CompletableFuture is an extension of Future that allows for explicitly setting its value and chaining multiple asynchronous computations.
Key Features

    Asynchronous Execution: Methods like supplyAsync, runAsync.
    Completion Stage Methods: Methods like thenApply, thenAccept, thenRun.

Example

```

import java.util.concurrent.CompletableFuture;

public class CompletableFutureExample {
    public static void main(String[] args) {
        CompletableFuture.supplyAsync(() -> {
            return "Hello";
        }).thenApply(result -> {
            return result + " World";
        }).thenAccept(result -> {
            System.out.println(result);
        });
    }
}
```
5. RecursiveTask

RecursiveTask is a subclass of ForkJoinTask used for tasks that return a result.
Example

```

import java.util.concurrent.RecursiveTask;
import java.util.concurrent.ForkJoinPool;

public class SumTask extends RecursiveTask<Integer> {
    private final int[] array;
    private final int start, end;

    public SumTask(int[] array, int start, int end) {
        this.array = array;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        if (end - start <= 10) {
            int sum = 0;
            for (int i = start; i < end; i++) {
                sum += array[i];
            }
            return sum;
        } else {
            int mid = (start + end) / 2;
            SumTask left = new SumTask(array, start, mid);
            SumTask right = new SumTask(array, mid, end);
            left.fork();
            return right.compute() + left.join();
        }
    }

    public static void main(String[] args) {
        ForkJoinPool pool = new ForkJoinPool();
        int[] array = new int[100];
        for (int i = 0; i < array.length; i++) {
            array[i] = i;
        }
        SumTask task = new SumTask(array, 0, array.length);
        System.out.println(pool.invoke(task));
    }
}
```
Conclusion

Java’s high-level concurrency utilities provide powerful tools for managing parallel processing and asynchronous tasks. By using these utilities, developers can write more efficient and maintainable concurrent applications. Each utility serves specific purposes and choosing the right one depends on the nature of the task and the required level of parallelism.
