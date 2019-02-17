---
layout: post
title:      "JavaScript Execution Context"
date:       2019-02-17 16:02:07 -0500
permalink:  javascript_execution_context
---


Have you been asked this —or a variant of the following question?

“Given the following code, please write the order in which the logs will appear in the console:”

```
const a = () => {
	b()
	console.log("a")
	
};

const b = () => {
	console.log("b")
	c()
};

const c = () => {
	console.log("c")
};

c()
console.log("d")
a()
```

Can you answer the question?

There is a fundamental principle in JavaScript that makes answering this questions very simple; The Execution Context and Execution Stack.

JavaScript is a a single threaded language, as therefore, only one one task can run at a time. The environment where this task or code is evaluated and executed is referred to as the execution context. In other words, wherever —or whenever a line of code runs in JavaScript, it runs in its execution context.

To understand how the execution context is created, we must first understand how the JavaScript engines run code in general. 
Every time JavaScript code runs, the engine does several things. One of them is to read the code line by line and use the syntax parser. In this phase, the engine parses the code and compiles it to instruct the computer on what to do. It also checks for errors in syntax and if there are none, it stores variable and function declarations to memory and creates the execution contexts.

The two main types of execution contexts in JavaScript are the:

* Global Execution Context: this is the base execution context and is where EVERYTHING in JavaScript is available. All of the codebase is wrapped inside the global execution context; variable declarations, variable assignments, function calls, and function declarations for example, all run in the global execution context. In JavaScript “global” means not inside a function and there can only be one global execution context in a particular program. 

* Functional Execution Context: this is the context that exists inside a particular function. Whenever a function is invoked or called, the JavaScrip engine creates another execution context that belongs to that function’s environment only. This functional execution context is then stacked on top of the global execution context. All following function call will generate their own execution context and will be stacked on top of the previous execution context.

Let’s take a look at the question and the code again to see the execution contexts in our code:


```
///////////////GLOBAL ENVIRONMENT//////////////

const a = () => {
  ////////////////////////////
	b()              ///////function a execution context/////// 
  console.log("a") ///////function a execution context///////
  ///////////////////////////
};

const b = () => {
  ////////////////////////////
	console.log("b")  ///////function b execution context///////
  c()               ///////function b execution context///////
  ////////////////////////////
};

const c = () => {
  ////////////////////////////
  console.log("c")  ///////function c execution context///////
  ////////////////////////////
};

c() ///////global execution context///////
console.log("d")  ///////global execution context///////
a() ///////global execution context///////

//////////////GLOBAL ENVIRONMENT///////////////
```

ok, so we’ve determined our execution contexts, but how do we determine the order of our console.logs and answer the question? 
To do so we need to understand the execution stack.

When the JavaScript engine parses our code in its first pass, it reads code from top to bottom in the global environment, creates the global execution context and stores it at the bottom of what is known as the execution stack:

```
—EXECUTION STACK—
///////////////////////////////////////
///////////////////////////////////////
///////////////////////////////////////
///////////////////////////////////////
///////////////////////////////////////
///GLOBAL EXECUTION///
```


If there is a function call, the engine creates and execution context for that function and stacks it on top of the execution stack. If it finds another function, it creates another execution context for that function and stacks it on top of the execution stack:

```
—EXECUTION STACK—
///////////////////////////////////////
///////////////////////////////////////
///////////////////////////////////////
///FUNC 2 EXECTUTION//
///FUNC 1 EXECTUTION//
///GLOBAL EXECUTION///
```

This process is repeated for until no other function calls are found. The order of the stack is determined by the order in which the engine finds the function calls, and the order of code execution is determined by the order of the execution stack. In the example above, function 2, will be executed first, then function 2, and finally, whatever code is left in the global environment/global execution context will be executed.

Let’s take a look again at our code in question and stack our execution contexts:

```
///////////////GLOBAL ENVIRONMENT//////////////

const a = () => {
  ////////////////////////////
	b()              ///////function a execution context/////// 
  console.log("a") ///////function a execution context///////
  ///////////////////////////
};

const b = () => {
  ////////////////////////////
	console.log("b")  ///////function b execution context///////
  c()               ///////function b execution context///////
  ////////////////////////////
};

const c = () => {
  ////////////////////////////
  console.log("c")  ///////function c execution context///////
  ////////////////////////////
};

c() ///////global execution context///////
console.log("d")  ///////global execution context///////
a() ///////global execution context///////

//////////////GLOBAL ENVIRONMENT///////////////
```


Our first tack looks like this:

```
—EXECUTION STACK—
///////////////////////////////////////
///FUNC C EXECTUTION//
///GLOBAL EXECUTION///
```


The engine finds a call to function c, stacks it on top of the stack and executes function c because there are no other functions to call, we only console log “c”. Once the function is called and executed, it gets removed from the stack, and so we are back in the global execution context:

```
—EXECUTION STACK—
///////////////////////////////////////
///////////////////////////////////////
///////////////////////////////////////
///////////////////////////////////////
///////////////////////////////////////
///GLOBAL EXECUTION///
```

The engine now finds console.log(“d”). At this point there is nothing to stack because we are still in the global execution context, so no new execution context is created. After the engine logs “d”, it finds a call to function a. At this point an execution context is created for function a and it’s put on top of the stack. Our stack now looks like this:

```
—EXECUTION STACK—
///////////////////////////////////////
///////////////////////////////////////
///////////////////////////////////////
///////////////////////////////////////
///FUNC A EXECTUTION//
///GLOBAL EXECUTION///
```

Now the engine attempts to execute function a but it finds a call to function b inside that context, so it won’t execute function a and instead creates a new execution context for function b and stacks it on top of the  execution stack like so:

```
—EXECUTION STACK—
///////////////////////////////////////
///////////////////////////////////////
///////////////////////////////////////
///FUNC B EXECTUTION//
///FUNC A EXECTUTION//
///GLOBAL EXECUTION///
```

Now inside function b’s execution context, the engine logs “b” and it encounters a call to function c which needs to stack at the top fo the execution stack. Because this is the last line of code in function b, function b is fully executed and removed from the stack by then. So our execution stack now looks like this:

```
—EXECUTION STACK—
///////////////////////////////////////
///////////////////////////////////////
///////////////////////////////////////
///FUNC C EXECTUTION//
///FUNC A EXECTUTION//
///GLOBAL EXECUTION///
```

Notice that function a is still in the execution stack because it hasn’t been executed yet. Remember the engine first found a call to function b in function a and that needed to be executed first. As of now function a is not executed and remains in the execution stack.

So at this point the next execution in the stack is for function c. The engine logs “c” and function c gets fully executed and removed from the stack, leaving function a at the top of the stack and next in line to be executed:

```
—EXECUTION STACK—
///////////////////////////////////////
///////////////////////////////////////
///////////////////////////////////////
///////////////////////////////////////
///FUNC A EXECTUTION//
///GLOBAL EXECUTION///
```

What remains to be executed in function a is the console.log(“a”). So the engines logs “a” and function a is fully executed. It gets removed from the stack and the engine is left with the global execution only. Since there is nothing else to execute in the global environment, the global execution context gets removed from the stack, and all our code is fully executed and complete.

Our answer then should be:

c
d
b
c
a

I think this is a very important concept to grasp in JavaScript. It helped me understand the basics of how JavaScript code runs and works. It’s helped me understand Scope, variable assignment and declaration on ES6,, and other important concepts. Hopefully is given you a similar understanding that you can use to become a better JavaScript developer.

