# UE5Coro

This repo is a fork of [UE5Coro](https://github.com/landelare/ue5coro), changed to work with UE4 out of the box.

I only tested this on a source build of the engine so i cannot guarantee that it will work on a launcher build.

There is an arguably important feature missing out of this version of UE5Coro (Http coroutines) that i did not manage to port to UE4. Maybe one day.

## Installing

For project-based installations, `git clone` this repository to your project's
Plugins folder.
If you want to install to the engine, you'll need to clone this repository under
`Engine/Plugins/Marketplace`.

You'll obviously need C\+\+20 support enabled in your project.
In your Build.cs file, add or change this line:
```c#
CppStandard = CppStandardVersion.Latest;
```
Add `"UE5Coro"` to your dependency module names, enable the plugin, and you're
ready to go!

## Features

Click these links for the detailed description of the main features provided
by this plugin, or keep reading for a quick overview.

* [Generators](Docs/Generator.md) are caller-controlled and `co_yield` a
sequence of objects, also known as iterators in C#.
* [Async coroutines](Docs/Async.md) control their own resumption by
`co_await`ing various awaiter objects. They can be used to implement BP latent
actions or as a generic fork in code execution like AsyncTask, but not
necessarily involving multithreading.
* [Overview of built-in awaiters](Docs/Awaiters.md) that you can use with async
coroutines.

### Generators

Generators can be used to return an arbitrary number of items from a function
without having to pass them through temp arrays, etc.
In C# they're known as iterators.

Returning `UE5Coro::TGenerator<T>` makes a function coroutine enabled, supporting
`co_yield`:

```cpp
using namespace UE5Coro;

TGenerator<FString> MakeParkingSpaces(int Num)
{
    for (int i = 1; i <= Num; ++i)
        co_yield FString::Printf(TEXT("🅿️ %d"), i);
}

// Elsewhere
for (const FString& Str : MakeParkingSpaces(123))
    Process(Str);
```

### Async coroutines

Return `FAsyncCoroutine` from a function (regular or UFUNCTION) to make it
coroutine enabled and support `co_await`. There's special handling in place that
automatically implements BP latent actions for you but it works for everything:

```cpp
using namespace UE5Coro;

UFUNCTION(BlueprintCallable, Meta = (Latent, LatentInfo = "LatentInfo"))
FAsyncCoroutine UExampleFunctionLibrary::K2_Foo(int EpicPleaseFixUE22342,
                                                FLatentActionInfo LatentInfo)
{
    // You can freely hop between threads:
    co_await Async::MoveToThread(ENamedThreads::AnyBackgroundThreadNormalTask);
    DoExpensiveThingOnBackgroundThread();
    co_await Async::MoveToGameThread();

    // Delay for 1 more second without blocking the game thread:
    co_await Latent::Seconds(1.0f);

    // The BP node will fire its latent exec pin once control leaves the coroutine.
}
```
