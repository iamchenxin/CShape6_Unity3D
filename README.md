This repository is out of date , please go to the [Original Repository](https://bitbucket.org/alexzzzz/unity-c-5.0-and-6.0-integration/src)

### Install 

1. Copy `bin\smcs.exe` from this repository to your `\Unity\Editor\Data\Mono\lib\mono\2.0` folder. Just copy, there's nothing to replace.

2. Create a new Unity project or open an existing one. Make sure that `Project Settings`/`Player`/`API Compatibility Level` is set to `.Net 2.0`.

3. Copy `bin\mcs.exe` or `bin\Roslyn` folder to your project's root.

### Current issues
Find a way to get Async/Await to work with IL2CPP.Force Unity IL2CPP mode interpret AsyncBridge by mono2x ,or modify AsyncBridge to work correctly with IL2CPP
### CShape Features in Unity3d (with Roslyn compiled)
| Features                  |   mono2x|  IL2CPP|
| :------------             | -------:| ------:|
| Async/Await               |       ✔|      ✘|
| Await In Catch Finally    |       ✔|      ✔|
| Caller Info               |       ✔|      ✔|
| Dictionary Initializer    |       ✔|      ✔|
| Exception Filter          |       ✔|      ✘|
| Expression Bodied Members |       ✔|      ✔|
| Nameof                    |       ✔|      ✔|
| Null Conditional          |       ✔|      ✔|
| Property Initializer      |       ✔|      ✔|
| Static Using              |       ✔|      ✔|
| String Interpolation      |       ✔|      ✔|
| Thread PingPong           |       ✔|      ✔|
| Dynamic                   |       ✘|      ✘|

### How does it work? 

smcs.exe receives and redirects compilation requests from Unity to one of the available C# compilers using the following rules:

1. If the current project contains `Roslyn` folder, then Roslyn C# 6.0 compiler will be used;

2. else if the current project contains `mcs.exe`, then this Mono C# 6.0 compiler will be used;

3. else if there's `AsyncBridge.Net35.dll` somewhere inside the project, then Unity's C# 5.0 compiler will be used (\Unity\Editor\Data\MonoBleedingEdge\lib\mono\4.5\mcs.exe);

4. else the stock compiler will be used (\Unity\Editor\Data\Mono\lib\mono\2.0\gmcs.exe).

It means that Unity will use the alternative compiler only in those projects, where you have explicitely expressed your wish to do so. Otherwise, it will use the stock compiler as usual, and the only bad thing that can happen is if `smcs.exe` crashes for whatever reason instead of doing its job. Then you can just delete it and continue as if nothing ever happened.

### License 

The source code is released under [WTFPL version 2](http://www.wtfpl.net/about/).

### Original talk thread at forum.unity3d.com

http://forum.unity3d.com/threads/c-6-0.314297/#post-2108999

### Original Repository (fork from bitbucket)
https://bitbucket.org/alexzzzz/unity-c-5.0-and-6.0-integration/src


### Known issues 

* Using Mono C# 6.0 compiler may cause Unity crashes while debugging in Visual Studio - http://forum.unity3d.com/threads/c-6-0.314297/page-2#post-2225696

* There are cases when Mono compiler fails to compile fully legit C# 6.0 code:

    * Null-conditional operator *(NullConditionalTest.cs)*

            var foo = new[] { 1, 2, 3 };
            var bar = foo?[0];
            Debug.Log((foo?[0]).HasValue); // error CS1061: Type `int' does not 
            // contain a definition for `HasValue' and no extension method
            // `HasValue' of type `int' could be found. Are you missing an
            // assembly reference?

        Mono compiler thinks that `foo?[0]` is `int` while it's actually `Nullable<int>`. However, `bar`'s type is deduced correctly - `Nullable<int>`. 
    
    * Getter-only auto-property initialization *(PropertyInitializerTest.cs)*
    
            class Abc { }

            class Test
            {
	           public Abc Abc { get; }
	           public Test()
	           {
		          Abc = new Abc(); // error CS0118: `Abc' is a `type' but a `variable' was expected
	           }
            }

* IL2CPP (affects iOS and WebGL):

    * Currently fails to process exception filters *(ExceptionFiltersTest.cs)*.
    * Currently fails to process AsyncBridge library.

* Async/await is not stable on Android. Results may vary - http://forum.unity3d.com/threads/c-6-0.314297/page-2#post-2188292               

### Random notes 

* Roslyn compiler was taken from VS 2015 installation.

* `mcs.exe`, `pdb2mdb.exe` and its dependencies were taken from [Mono 4.0.4][mono] installation. pdb2mdb.exe that comes with Unity is not compatible with the assemblies generated with Roslyn compiler.

* AsyncBridge library provides a set of types that makes it possible to use _async/await_ in projects that target CLR 2.0. For more information, check [this blog post][asyncbridge].

* If you use _async/await_ inside Unity events (Awake, Start, Update etc) you may notice that continuations (the code below `await` keyword) are executed in background threads. Most likely, this is not what you would want. To force `await` to return the execution to the main thread, you'll have to provide it with a synchronization context. Check `UnityScheduler.cs` example located inside the project.

    For more information about what synchronization context is, what it is for and how to use it, see this set of articles by Stephen Toub: [one][synccontext1], [two][synccontext2], [three][synccontext3].

[mono]: http://www.mono-project.com/download/#download-win
[roslyn]: https://github.com/dotnet/roslyn
[asyncbridge]: https://www.simple-talk.com/blogs/2012/04/18/asyncbridge-write-async-code-for-net-3-5/
[synccontext1]: http://blogs.msdn.com/b/pfxteam/archive/2012/01/20/10259049.aspx
[synccontext2]: http://blogs.msdn.com/b/pfxteam/archive/2012/01/21/10259307.aspx
[synccontext3]: http://blogs.msdn.com/b/pfxteam/archive/2012/02/02/await-synchronizationcontext-and-console-apps-part-3.aspx
