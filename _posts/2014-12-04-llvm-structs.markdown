---
layout: post
title:  "LLVM Structure Code Generation"
date:   2014-12-04 13:36:04
categories: llvm
---
Continuing in the vein of LLVM frustrations, I've been working on generating LLVM IR bytecode for structures. Creating a structure is generally pretty straightforward, but accessing the
members is not very well documented. This post should help use the LLVM C++ API to generate LLVM bytecode for structs.

## Creating a structure
This was the most straight-forward, creating a structure type is as easy as loading up an ```ArrayRef<Value *>``` with the ```Type```s for each member and then passing that (called "members_array_ref"
in the below example) to:
{% highlight c++ %}
llvm::StructType::create(llvm::getGlobalContext(), members_array_ref, struct_name, false)
{% endhighlight %}

Once this ```StructType``` is created, it can be passed to the constructor to ```AllocaInst``` and allocate specific instances of the structure type:
{% highlight c++ %}
new llvm::AllocaInst(struct_type, var_name, block)
{% endhighlight %}

## The Often Misunderstood GEP Instruction
To access an individual field of the structure (either for ```StoreInst``` or ```LoadInst``` access), you must use a [very poorly documented instruction: ```GetElementPtrInst```](http://llvm.org/docs/doxygen/html/classllvm_1_1GetElementPtrInst.html).
This instruction is so confusing that [it has its own page on the main LLVM site](http://llvm.org/docs/GetElementPtr.html) which provides a helpful overview of what the IR instruction does, but not how to create 
one through the C++ API. There is a decent start to a solution found on [this Stackoverflow question](http://stackoverflow.com/questions/17409216/llvm-how-to-access-to-struct-fields-based-on-their-names) however 
it fails to explain the generated code.

From the official Doxygen page for the class, you create a GEP instruction using the ```Create``` method:
{% highlight c++ %}
static GetElementPtrInst * Create (Value *Ptr, ArrayRef< Value * > IdxList, const Twine &NameStr, BasicBlock *InsertAtEnd)
{% endhighlight %}
The most confusing of these arguments is the ```IdxList```. From the Stackoverflow question, I was able to deduce that you need to create an ```ArrayRef<Value *>``` of the indexes (0, then the index into the struct).
For example, the indexes for the first element of the struct will have the indexes (0, 0), the second element will be (0, 1) and so forth. To calculate the second index, I looped through the members and determined
the index. Once I had the indexes, I needed to transform them into a ```Value *```, which I first attempted by calling: 
{% highlight c++ %}
llvm::ConstantInt::get(llvm::getGlobalContext(), llvm::APInt(64, index, false));
{% endhighlight %}
Which would cause a segfault in the ```Create``` call! After banging my head against this for a few hours, I tried **to change these indexes from ```i64```s to ```i32```s which stopped the segfaulting**!
Now that I could create the GEP instruction (```*Ptr``` is set to the return value from the ```AllocaInst``` call with the ```StructType```), I could pass that to either my ```StoreInst``` or ```LoadInst```
and access the structure members!