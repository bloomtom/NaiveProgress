# NaiveProgress

>Provides a non-threaded implementation of `IProgress`.

The canonical implementation of [`IProgress`](https://github.com/dotnet/corefx/blob/master/src/Common/src/CoreLib/System/IProgress.cs) is [`Progress`](https://github.com/dotnet/corefx/blob/master/src/Common/src/CoreLib/System/Progress.cs), which uses a `SynchronizationContext` if you have one, and the thread pool if you don't. ASP.NET Core, Console and Test applications don't. The result is your progress reports might end up coming in out-of-order.

`NaiveProgress` doesn't do anything fancy. No `SynchronizationContext`, no thread pool. All reports are passed along to an event in the order they are received.

## Nuget Packages

Package Name | Target Framework | Version
---|---|---
[NaiveProgress](https://www.nuget.org/packages/bloomtom.NaiveProgress) | .NET Standard 2.0 | ![NuGet](https://img.shields.io/nuget/v/bloomtom.NaiveProgress.svg)

## Usage

Whenever you need an `IProgress`, make a `NaiveProgress<T>` instead of a `Progress<T>`; it was designed to be a drop in replacement.

#### Proof of Concept
Consider this function which delays for a period of time, and frequently reports progress.
``` csharp
private static void Delay(TimeSpan t, IProgress<TimeSpan> progress)
{
	var sw = new System.Diagnostics.Stopwatch();
	sw.Start();

	while (sw.Elapsed < t)
	{
		System.Threading.Thread.Sleep(1);
		progress.Report(sw.Elapsed);
	}
}
```

Using the normal `Progress` implementation is as follows
``` csharp
var progress = new Progress<TimeSpan>((e) =>
{
	Console.WriteLine(e.Ticks);
});

Delay(TimeSpan.FromMilliseconds(20), progress);
```
The following is the entire real output from running the above in a console application.
```
12618
468586
69860
117134
```
Ouch. As you can see, the console writes are out-of-order. Otherwise you'd expect Ticks to always increase.

To get in-order reports, simply change `Progress` to `NaiveProgress`
``` csharp
var progress = new NaiveProgress<TimeSpan>((e) =>
{
	Console.WriteLine(e.Ticks);
});

Delay(TimeSpan.FromMilliseconds(20), progress);
```
The result:
```
12986
62947
78007
93076
108074
123171
138230
153266
168358
183371
198416
213470
```
Every event is handled, and they're handled in-order.