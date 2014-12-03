---
layout: post
title:  "LLVM Quirks Part I"
date:   2014-12-03 10:36:04
categories: llvm
---
As I've recently been working on a project in [LLVM](http://llvm.org/) which is a very powerful piece of software, but the documentation I've found lacking.
The APIs have changed and other than a few simple examples, there isn't an up-to-date resource I've found for using the LLVM C++ API to generate byte-code.
I wanted to share a few of the poorly-documented "quirks" I've run into and put them all into one place to aid in other LLVM adventurers.

## LLVM JIT Crashes
I followed a [tutorial online](http://gnuu.org/2009/09/18/writing-your-own-toy-compiler/6/) to get a feel of the C++ API, and used the following code:
{% highlight c++ linenos %}
/* Executes the AST by running the main function */
GenericValue CodeGenContext::runCode() {
	     std::cout << "Running code...\n";
	     ExecutionEngine *ee = ExecutionEngine::create(rootModule, false);
	     vector<GenericValue> noargs;
	     GenericValue v = ee->runFunction(mainFunction, noargs);
	     std::cout << "Code was run.\n";
	     return v;
}
{% endhighlight %}

Alas, on LLVM 3.5, this code will segfault. There are two changes which I made to remedy this:

1. Adding the line ```LLVMInitializeNativeTarget();``` before the ExecutionEngine is created
2. Adding an error string to the engine (using the EngineBuilder API instead) and checking for a NULL return value:
{% highlight c++ %}
   std::string err;
   LLVMInitializeNativeTarget();
   llvm::ExecutionEngine *ee = llvm::EngineBuilder(rootModule).setErrorStr(&err).create();
   if (!ee)
   {
       std::cout << "Error: " << err << std::endl;
       exit(-1);
   }
{% endhighlight %}

This now allowed me to run small generated functions! However, when I tried to call another function I had generated, the JIT would segfault once more :(
I was able to verify my generated module was valid by running it through the ```lli``` utility, but the JIT continue to crash. Throwing in the towel (for now),
my solution was to change the ExecutionEngine to use the interpreter instead of the JIT. This was done by changing the include line from ```#include <llvm/ExecutionEngine/JIT.h>```
to ```#include <llvm/ExecutionEngine/Interpreter.h>``` and modifying my Makefile to link against the interpreter libraries.

Once those changes were made, I added a call to ```.setEngineKind(llvm::EngineKind::Interpreter)``` on the EngineBuilder chain and now my code was properly being run!


