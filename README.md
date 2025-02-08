# mozart oz system

# profiler

How do the best do profiling of an application ? 

```
http://mozart2.org/mozart-v1/doc-1.4.0/profiler/index.html
```

## three piggies

## 1 What Is Profiling

Once your application works, you may wish to optimize it for speed and memory consumption. For this, you need to identify the parts of your application that may significantly benefit from such optimizations; it would be pointless to optimize a procedure that is called only once. Profiling automatically instruments your program to gather statistical data on procedure invocations and memory allocation.

The profiler collects information in a per procedure basis. This information consists of the following quantities:

#### heap
Heap memory allocated by the procedure

#### calls
How many times the procedure was called

#### samples
Statistical estimation of the time spent in the procedure. This works as follows: every 10ms a signal is delivered and the emulator increases the ``samples'' counter of the procedure currently executing.


This is broken and disabled in the current implementation! ? huh ?

#### closures
How many times the corresponding closure was created. Note that nested procedure declarations like Bar in

```
proc {Foo X Y}
  proc {Bar U V} ... end 
  ... 
end
```

both consume runtime and memory since a new closure for Bar has to be created at runtime whenever Foo ist called. So one might consider lifting the definition of Bar.


# 2 How To Compile For Profiling
In order to gather the profiling information, your code has to be instrumented with additional profiling code. This code is automatically inserted by the compiler when it is invoked with the --profiler option. This option can also be abbreviated -p. There is however an unfortunate limitation when compiling code for profiling: tail-call optimization is turned off (except for self applications). Besides this instrumented code runs in general a bit slower than code that was not compiled for profiling.

As an example, let's consider the following rather pointless application below. I call it ``The 3 Little Piggies'', and it does nothing but waste time and memory:
```
functor 
import Application
define 
   Args = {Application.getCmdArgs
           record(size( single type:int optional:false)
                  times(single type:int optional:false))}
   proc {FirstPiggy}
      {List.make Args.size _}
      {For 1 Args.times 1 SecondPiggy}
   end 
   proc {SecondPiggy _}
      {List.make Args.size _}
      {For 1 Args.times 1 ThirdPiggy}
   end 
   proc {ThirdPiggy _}
      {List.make Args.size _}
   end 
   {FirstPiggy}
   {Application.exit 0}
end 
```

The application can be compiled for profiling as follows:
```
ozc -px piggies.oz -o piggies
```


## How to invoke the profiler
the profiler interface is integrated in the Oz debugger tool ozd and can be invoked using the -p option. We can profile ``The 3 Little Piggies'' as follows:
```
ozd -p piggies -- --size 1000 --times 100
```
Note how the double dash separates ozd's arguments from the application's arguments. Shortly thereafter, the window shown below pops up:


