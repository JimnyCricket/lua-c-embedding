# Using Lua from Java

This requires the JNI. So, first, we'll look at how to use it for a simple example.

## Hello, World

First, we make a Java class, HelloWorldJNI.java, that loads a native (C) library, helloworld, that contains a method ```hello()```.

```
public class HelloWorldJNI 
{
  private native void hello();
 
  public static void main(String[] args) 
  {
    System.load(args[0]);
    new HelloWorldJNI().hello();  
  }
}
```

Next steps are to compile the java code with ```javac HelloWorldJNI.java``` to get the a class file, HelloWorld.class, and then generate the JNI header file with ```javah HelloWorldJNI```.

Then we create the JNI C implementation, HelloWorldJNI.c:

```
#include <jni.h>
#include <stdio.h>
#include "HelloWorldJNI.h"

JNIEXPORT void JNICALL Java_HelloWorldJNI_hello(JNIEnv *env, jobject thisObj)
{
  printf("Hello, World!\n");
  return;
}
```

Now we need to compile the C code and link everything up. For this we can use:

```gcc -I"$JAVA_HOME/include" -shared -o libhelloworld.dylib HelloWorldJNI.c```

Notice that we're including the location of JAVA_HOME/include, are creating a shared library, and are naming the library libhelloworld.dylib. To run the java application, you use ```java HelloWorldJNI <full path to helloworld.dylib>```.

## Using Torch from Java

### JNI Version of the Factorial Example

We're going to adapt the earlier example where we compute the factorial of a number. This is on the jni-example branch. You can build and run in the ```jni-example``` directory by using this:

```make && java FactorialJNI <full path to libfactorial.dylib> <int of your choice>```

### Using Torch Tensors via the JNI




--


**Note: There is an issue using 64-bit LuaJIT on macOS**, which is why I switched this from using LuaJIT to Lua. Info is found below. An earlier version of this repo contains the LuaJIT version which was affected by the issue mentioned below.

- http://comments.gmane.org/gmane.comp.lang.lua.luajit/4817
- http://stackoverflow.com/questions/14840569/sigsegv-error-in-some-lua-c-code
- http://stackoverflow.com/questions/13400660/binding-lua-in-static-library-segfault?rq=1
- http://nticformation.com/solutions_problems.php?tuto=59091&subCategory=c+lua+jni+luajit&Category=C+Language

Based on the last of the above links, I think the issue is that ```-pagezero_size 10000 -image_base 100000000``` is not included when compiling the JNI stuff. The problem is that when it is included, the compiler says 
>-pagezero_size option can only be used when linking a main executable
which means those params can't be used for shared libraries (which JNI needs).

I've confirmed that the above is the problem via [a thread](http://www.freelists.org/post/luajit/luaL-newstate-fails-on-64bit-Mac-cant-set-linker-flags) involving Lua's maintainer, Mike Pall. This is specifically an issue on 64-bit macOS. If I compile a 32-bit LuaJIT from source, then I should be able to get things to work. But, a 32-bit version might not be as useful for our purposes. To build the 32-bit LuaJIT binary, you can run the following commands inside the LuaJIT directory:

```CFLAGS="-arch i386" GCCFLAGS="-arch i386" LDFLAGS="-arch i386" make && sudo make install```