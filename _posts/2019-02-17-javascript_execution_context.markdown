---
layout: post
title:      "JavaScript Execution Context"
date:       2019-02-17 21:02:06 +0000
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

