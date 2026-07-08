# JavaScript Runtime, Event Loop, and Execution Order
----
## BTS of Javascript execution

JavaScript is single-threaded. It can only do one thing at a time and yet somehow it manages network requests, timers, user clicks, and animations without freezing the page. This isn't a trick its's a very specific, very predictable execution model.
----
## What is the JavaScript call stack and how it processes synchronous code?
The call stack is javascript's mechanism for tracking what function is currently executing works with the lifo structure.
### Processing Synchronous code:
When a function is called, a new "frame" for it is pushed onto the top of the stack. If that function calls another function, the new function's frame goes on top of it. When a function finishes executing its frame is popped off the stack, and control goes back to whatever function is now on top. This continues until the call stack is completely empty.
----
 ## What are microtasks and macrotasks, and how they are scheduled?
Microtasks and macrotasks are the two queues JavaScript uses to hold asynchronous callbacks until the call stack is free to run them. They exist because asynchronous work can't just interrupt the stack rather it has to wait its turn in line. The two queues aren't equal,they have different priority.

 ### Microtasks includes:
----
 
 - callbacks

 - I/O events

 - UI rendering steps
 
 - setImmediate
 
 - user interaction events

 ### Microtasks includes:
----
 
 - Promise callbacks

 - queueMicrotask()

 - async/await continuations

 - Mutation Observer callbacks

----

Once the call stack becomes empty, the engine doesn't just grab the next macrotask. Instead it checks the microtask queue first. It runs every microtask currently in the queue and if running one of them adds new microtasks, those get run too, before moving on. The microtask queue must be completely empty before anything else happens. Only then does it take a single item from the macrotask queue and execute it. Once that macrotask finishes and the stack empties again, the cycle repeats: drain microtasks fully, then take one more macrotask.

## Why Promises, queueMicrotask(), and setTimeout() behave differently in execution order?

They behave differently because they don't land in the same queue, and the two queues aren't treated equally. One queue is always drained completely before the other gets a single turn.
The scheduling rule that causes the difference:
Once the current synchronous code finishes and the call stack is empty, the engine fully drains the microtask queue — including any new microtasks added while draining — before it's allowed to touch the macrotask queue at all. Only after microtasks are completely empty does it pull one item from the macrotask queue.
So setTimeout(fn, 0) doesn't mean "run immediately" or even "run next" — it means "run as soon as possible, but only after all synchronous code and every currently pending microtask have finished."

A microtask and a macrotask both just run a function when their turn comes. The difference is that microtask queue gets exhaustively processed every single time the stack clears, before the macrotask queue is even considered. That guarantee is what makes Promise based code ahead of setTimeout-based code, no matter how the timings or delays are written.

## How the event loop decides what runs when?
The event loop is like a person running a strict repeating cycle. It doesn't consider anything with importance or anything. It just follows the same sequence again and again.

### Steps of the loop process cycle:
1. Run synchronous code until the call stack is empty:
 Whatever is currently executing runs to completion first.

2. Check the microtask queue:
If there's anything in it, it runs. While running it, if new microtasks get added, those get added to the same and then countinues to run until its empty.

3. Take exactly one task from the macrotask queue and run it. Only one runs at a time. This could be a setTimeout callback, a UI event, or an I/O completion, etc.

4. After that single macrotask finishes and the stack empties again, go back to step 2. Drain microtasks fully before considering another macrotask.

5. Rendering gets a chance to happen at certain points in this cycle.


## Exercise 1 – Microtasks vs Macrotasks
console.log('1');
setTimeout(() => {
   console.log('2');
}, 0);

Promise.resolve().then(() => {
   console.log('3');
});

queueMicrotask(() => {
   console.log('4');
});

console.log('5');

Output
-----
1

5

3

4

2

Explanation
----
console.log('1');  
#Runs first

(Synchronous)


setTimeout(() => {
   console.log('2');  
}, 0);

#Runs last as it is scheduled

(Macrotask)

Promise.resolve().then(() => {  
   console.log('3');
});

#Queued as a microtask

(Microtask)

queueMicrotask(() => {            
   console.log('4');
});

#queued as a microtask

#Microtask

console.log('5');

#Runs right after 1

(Synchronous)

#Once the stack is empty after 5 the engine drains the microtask queue completely. Then it first takes 3 that is queued first then 4 queued second and then only after both are done does the engine look at the macrotask queue and run the setTimeout callback, printing 2.

## How the Event Loop Handles Each

1. Synchronous phase: The call stack executes 1 and 5 directly in order with no interruption. 
2. Microtask draining: Once the stack is empty, the event loop checks the microtask queue and runs everything in it, in the order it was added 3 from the promise then 4 from queueMicrotask. It won't touch the macrotask queue until this is fully empty.
Macrotask turn: With microtasks exhausted, the event loop finally pulls the setTimeout callback from the macrotask queue and runs it prints 2.


## Exercise 2 – Nested and Sequential Tasks
console.log('start');

setTimeout(() => {
   console.log('setTimeout 1');
   Promise.resolve().then(() => console.log('Promise 1'));
}, 0);

setTimeout(() => {
   console.log('setTimeout 2');
}, 0);

Promise.resolve().then(() => console.log('Promise 2'));

console.log('end');

## Output
start

end

Promise 2

setTimeout 1

Promise 1

setTimeout 2

## Explanation
console.log('start');

#Starts Immediately

(Synchronous)

setTimeout(() => {
   console.log('setTimeout 1');

(Macrotask)

   Promise.resolve().then(() => console.log('Promise 1'));
}, 0);

#callback deferred to macrotask queue
(Microtask)

setTimeout(() => {
   console.log('setTimeout 2');
}, 0);

(Macrotask)

#callback deferred to macrotask queue

Promise.resolve().then(() => console.log('Promise 2'));

#deferred to microtask queue

(Microtask)

console.log('end');

#Starts immediately after 'Start'

(Synchronous)

## Event Loop Tick Breakdowm
Synchronous code runs first: start → end.
Then all microtasks run before any macrotask: Promise 2.
First macrotask executes: setTimeout 1, which creates a new microtask (Promise 1).
Before moving to the next macrotask, the engine immediately drains microtasks: Promise 1.
Second macrotask runs last: setTimeout 2.


## Key Takeaways
1. Execution order: Sync code runs first, then all microtasks, then one macrotask at a time.
2. Asynchronous behavior: Promises run before setTimeout, even with a delay, because microtasks always go first.
3. In Real projects: Not knowing this may cause confusing bugs  like UI updates or state changes happening in the wrong order.


