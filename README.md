# BlazorObservers

BlazorObservers is a set of Blazor wrappers around the JavaScript Observer functionality. 

Currently only the ResizeObserver is supported.  

## Introduction
Blazor not support resize events. 
This means that usually, a complicated JSInterop will be required, which adds a lot of boilerplate code to the project.

This library attempts to prevent this issue, by providing a manager, which creates and deletes [JavaScript ResizeObservers](https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver). 
This allows efficient and clean code to execute dotNET methods on element(s) resize.

## Usage

First add the ResizeObserverRegistrationService to the dependency injection.
```csharp
using BlazorObservers.ObserverLibrary.DI;

...

builder.Services.AddResizeObserverRegistrationService();
```

And add the using statements to the imports.razor
```razor
@using BlazorObservers.ObserverLibrary.JsModels
@using BlazorObservers.ObserverLibrary.Services
@using BlazorObservers.ObserverLibrary.Tasks
```

Then you can inject the ResizeObserverRegistrationService into your razor component.

Now register an element with the OnAfterRenderAsync method, and make sure the registration is removed on disposal.
```csharp

@inject ResizeObserverRegistrationService ResizeObserverRegistrationService
@implements IAsyncDisposable

<div @ref="targetElement" style="width: 100%; height: 100%; background-color: green;"></div>

private ElementReference targetElement { get; set; }
private ObserverTask? taskReference;

protected override async Task OnAfterRenderAsync(bool firstRender)
{
    if (firstRender)
        taskReference = await ResizeObserverRegistrationService.RegisterObserver(
            async (entries) => Console.WriteLine("Hello resizable world"), 
            targetElement);
}

public async ValueTask DisposeAsync()
{
    if (taskReference is not null)
        await ResizeObserverRegistrationService.DeregisterObserver(taskReference);
}

```

When changing the size of (one of) the observed element(s) in your callback method, make sure to halt callback execution, as otherwise an infinite loop can appear, and crash your application.
```csharp
protected override async Task OnAfterRenderAsync(bool firstRender)
{
    if (firstRender)
    {
        taskReference = await ResizeObserverRegistrationService.RegisterObserver(ObserverResizeEvent, TabBar);
    }
}

private async Task ObserverResizeEvent(JsResizeObserverEntry[] entries)
{
    if (taskReference is null) return;
    taskReference.HaltTaskTriggering();
    await MethodThatChangesTheElementSize();
    taskReference.ResumeTaskTriggering();
}

```


## Future
Completed:
- [x] Make ResizeObserver callback parameters available
- [x] Auto-package

Feature backlog:
- [ ] Add compatibility with element rerendering, so that the ResizeObserver stops observing the original element, and starts observing the newly rendered element.
- [ ] Add ExecuteFinal option to HaltTaskTriggering, so that the last callback with the accurate size change is executed.
- [ ] Add more observers, such as [MutationObserver](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver) and [IntersectionObserver](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserver)
- [ ] Unit testing
- [ ] Performance analysis

