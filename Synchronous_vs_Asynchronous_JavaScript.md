# Synchronous vs Asynchronous JavaScript

__Let me start this article by asking, "What is JavaScript"? Well, here's the most confusing yet to-the-point answer I have found so far:__

> JavaScript is a single-threaded, non-blocking, asynchronous, concurrent programming language with lots of flexibility

Hold on a second – did it say single-threaded and asynchronous at the same time? If you understand what single-threaded means, you'll likely mostly associate it with synchronous operations. How can JavaScript be asynchronous, then?

## JavaScript Functions are First-Class Citizens

In JavaScript, you can create and modify a function, use it as an argument, return it from another function, and assign it to a variable. All these abilities allow us to use functions everywhere to place a bunch of code logically.

We need to tell the JavaScript engine to execute functions by invoking them. It'll look like this:

```JavaScript
// Define a function
function f1() {
    // Do something
    // Do something again
    // Again
    // So on...
}

// Invoke the function
f1();
```
By default, every line in a function executes sequentially, one line at a time. The same is applicable even when you invoke multiple functions in your code. Again, line by line.

## Synchronous JavaScript – How the Function Execution Stack Works

So what happens when you define a function and then invoke it? The JavaScript engine maintains a stack data structure called function execution stack. The purpose of the stack is to track the current function in execution. It does the following:

- When the JavaScript engine invokes a function, it adds it to the stack, and the execution starts.

- If the currently executed function calls another function, the engine adds the second function to the stack and starts executing it.

- Once it finishes executing the second function, the engine takes it out from the stack.

- The control goes back to resume the execution of the first function from the point it left it last time.

- Once the execution of the first function is over, the engine takes it out of the stack.

- Continue the same way until there is nothing to put into the stack.

The function execution stack is also known as the Call Stack.

Let's look at an example of three functions that execute one by one:

```JavaScript
function f1() {
  // some code
}
function f2() {
  // some code
}
function f3() {
  // some code
}

// Invoke the functions one by one
f1();
f2();
f3();
```

First, f1() goes into the stack, executes, and pops out. Then f2() does the same, and finally f3(). After that, the stack is empty, with nothing else to execute.

Ok, let's now work through a more complex example. Here is a function f3() that invokes another function f2() that in turn invokes another function f1().

```JavaScript
function f1() {
  // Some code
}
function f2() {
  f1();
}
function f3() {
  f2();
}
f3();
```
Notice that first f3() gets into the stack, invoking another function, f2(). So now f2() gets inside while f3() remains in the stack. The f2() function invokes f1(). So, time for f1() to go inside the stack with both f2() and f3() remaining inside.

First, f1() finishes executing and comes out of the stack. Right after that f2() finishes, and finally f3().

The bottom line is that everything that happens inside the function execution stack is sequential. This is the Synchronous part of JavaScript. JavaScript's main thread makes sure that it takes care of everything in the stack before it starts looking into anything elsewhere.

## Asynchronous JavaScript – How Browser APIs and Promises Work

The word asynchronous means not occurring at the same time. What does it mean in the context of JavaScript?

Typically, executing things in sequence works well. But you may sometimes need to fetch data from the server or execute a function with a delay, something you do not anticipate occurring NOW. So, you want the code to execute asynchronously.

In these circumstances, you may not want the JavaScript engine to halt the execution of the other sequential code. So, the JavaScript engine needs to manage things a bit more efficiently in this case.

We can classify most asynchronous JavaScript operations with two primary triggers:

1. __Browser API/Web API__ events or functions. These include methods like setTimeout, or event handlers like click, mouse over, scroll, and many more.

2. __Promises.__ A unique JavaScript object that allows us to perform asynchronous operations.

## How to Handle Browser APIs/Web APIs

Browser APIs like setTimeout and event handlers rely on callback functions. A callback function executes when an asynchronous operation completes. Here is an example of how a setTimeout function works:

```JavaScript
function printMe() {
  console.log('print me');
}

setTimeout(printMe, 2000);
```
The setTimeout function executes a function after a certain amount of time has elapsed. In the code above, the text print me logs into the console after a delay of 2 seconds.

Now assume we have a few more lines of code right after the setTimeout function like this:
```JavaScript
function printMe() {
  console.log('print me');
}

function test() {
  console.log('test');
}

setTimeout(printMe, 2000);
test();
```
Will the JavaScript engine wait for 2 seconds to go to the invocation of the test() function and output this:

```JavaScript
printMe
test
```
Or will it manage to keep the callback function of setTimeout aside and continue its other executions? So the output could be this, perhaps:
```JavaScript
test
printMe
```
If you guessed the latter, you're right. That's where the asynchronous mechanism kicks in.

## How the JavaScript Callback Queue Works (aka Task Queue)

JavaScript maintains a queue of callback functions. It is called a callback queue or task queue. A queue data structure is First-In-First-Out(FIFO). So, the callback function that first gets into the queue has the opportunity to go out first.

![Callback Queue](./callbackQueue.png "Callback Queue/Task Queue")

The above image shows the regular call stack. There are two additional sections to track if a browser API (like setTimeout) kicks in and queues the callback function from that API.

The JavaScript engine keeps executing the functions in the call stack. As it doesn't put the callback function straight into the stack, there is no question of any code waiting for/blocking execution in the stack.

The engine creates a loop to look into the queue periodically to find what it needs to pull from there. It pulls a callback function from the queue to the call stack when the stack is empty. Now the callback function executes generally as any other function in the stack. The loop continues. This loop is famously known as the Event Loop.

So, the moral of the story is:

- When a Browser API occurs, park the callback functions in a queue.

- Keep executing code as usual in the stack.

- The event loop checks if there is a callback function in the queue.

- If so, pull the callback function from the queue to the stack and execute.

- Continue the loop.

Alright, let's see how it works with the code below:

```JavaScript
function f1() {
    console.log('f1');
}

function f2() {
    console.log('f2');
}

function main() {
    console.log('main');
    
    setTimeout(f1, 0);
    
    f2();
}

main();
```
The code executes a setTimeout function with a callback function f1(). Note that we have given zero delays to it. This means that we expect the function f1() to execute immediately. Right after setTimeout, we execute another function, f2().

So, what do you think the output will be? Here it is:

```JavaScript
main
f2
f1
```
But, you may think that f1 should print before f2 as we do not delay f1 to execute. But no, that's not the case. Remember the event loop mechanism we discussed above?

Here are steps written out:

1. The main() function gets inside the call stack.

2. It has a console log to print the word main. The console.log('main') executes and goes out of the stack.

3. The setTimeout browser API takes place.

4. The callback function puts it into the callback queue.

5. In the stack, execution occurs as usual, so f2() gets into the stack. The console log of f2() executes. Both go out of the stack.

6. The main() also pops out of the stack.

7. The event loop recognizes that the call stack is empty, and there is a callback function in the queue.

8. The callback function f1() then goes into the stack. Execution starts. The console log executes, and f1() also comes out of the stack.

9. At this point, nothing else is in the stack and queue to execute further.

I hope it's now clear to you how the asynchronous part of JavaScript works internally. But, that's not all. We have to look at promises.

## How the JavaScript Engine Handles Promises

In JavaScript, promises are special objects that help you perform asynchronous operations.

You can create a promise using the Promise constructor. You need to pass an executor function to it. In the executor function, you define what you want to do when a promise returns successfully or when it throws an error. You can do that by calling the resolve and reject methods, respectively.

Here is an example of a promise in JavaScript:
```JavaScript
const promise = new Promise((resolve,reject) =>
    resolve("I am a resolved promise");
);
```
After the promise is executed, we can handle the result using the .than() method and any errors with the .catch() method.

```JavaScript
promise.then(result=> console.log(result))
```
You use promises every time you use the fetch() method to get some data from a store.

The point here is that JavaScript engine doesn't use the same callback queue we have seen earlier for browser APIs. It uses another special queue called the Job Queue.

## What is the Job Queue in JavaScript?
Every time a promise occurs in the code, the executor function gets into the job queue. The event loop works, as usual, to look into the queues but gives priority to the job queue items over the callback queue items when the stack is free.

The item in the callback queue is called a macro task, whereas the item in the job queue is called a micro task.

So the entire flow goes like this:

- For each loop of the event loop, one task is completed out of the callback queue.

- Once that task is complete, the event loop visits the job queue. It completes all the micro tasks in the job queue before it looks into the next thing.

- If both the queues got entries at the same point in time, the job queue gets preference over the callback queue.
Now, let's look at an example to understand this sequence better:

```JavaScript
function f1() {
    console.log('f1');
}

function f2() {
    console.log('f2');
}

function main() {
    console.log('main');
    
    setTimeout(f1, 0);
    
    new Promise((resolve, reject) =>
        resolve('I am a promise')
    ).then(resolve => console.log(resolve))
    
    f2();
}

main();
```
In the above code, we have a setTimeout() function as before, but we have introduced a promise right after it. Now remember all that we have learned and guess the output.

If your answer matches this, you are correct:
```JavaScript
main
f2
I am a promise
f1
```
The flow is almost the same as above, but it is crucial to notice how the items from the job queue prioritize the items from the task queue. Also note that it doesn't even matter if the setTimeout has zero delay. It is always about the job queue that comes before the callback queue.