---
title: Call JavaScript functions from .NET methods in ASP.NET Core Blazor
author: guardrex
description: Learn how to invoke JavaScript functions from .NET methods in Blazor apps.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 11/08/2022
uid: blazor/js-interop/call-javascript-from-dotnet
---
# Call JavaScript functions from .NET methods in ASP.NET Core Blazor

[!INCLUDE[](~/includes/not-latest-version.md)]

This article explains how to invoke JavaScript (JS) functions from .NET.

:::moniker range=">= aspnetcore-7.0"

For information on how to call .NET methods from JS, see <xref:blazor/js-interop/call-dotnet-from-javascript>.

<xref:Microsoft.JSInterop.IJSRuntime> is registered by the Blazor framework. To call into JS from .NET, inject the <xref:Microsoft.JSInterop.IJSRuntime> abstraction and call one of the following methods:

* <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A?displayProperty=nameWithType>
* <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeAsync%2A?displayProperty=nameWithType>
* <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType>

For the preceding .NET methods that invoke JS functions:

* The function identifier (`String`) is relative to the global scope (`window`). To call `window.someScope.someFunction`, the identifier is `someScope.someFunction`. There's no need to register the function before it's called.
* Pass any number of JSON-serializable arguments in `Object[]` to a JS function.
* The cancellation token (`CancellationToken`) propagates a notification that operations should be canceled.
* `TimeSpan` represents a time limit for a JS operation.
* The `TValue` return type must also be JSON serializable. `TValue` should match the .NET type that best maps to the JSON type returned.
* A [JS `Promise`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) is returned for `InvokeAsync` methods. `InvokeAsync` unwraps the [`Promise`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) and returns the value awaited by the [`Promise`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise).

For Blazor apps with prerendering enabled, calling into JS isn't possible during prerendering. For more information, see the [Prerendering](#prerendering) section.

The following example is based on [`TextDecoder`](https://developer.mozilla.org/docs/Web/API/TextDecoder), a JS-based decoder. The example demonstrates how to invoke a JS function from a C# method that offloads a requirement from developer code to an existing JS API. The JS function accepts a byte array from a C# method, decodes the array, and returns the text to the component for display.

```html
<script>
  window.convertArray = (win1251Array) => {
    var win1251decoder = new TextDecoder('windows-1251');
    var bytes = new Uint8Array(win1251Array);
    var decodedArray = win1251decoder.decode(bytes);
    console.log(decodedArray);
    return decodedArray;
  };
</script>
```

[!INCLUDE[](~/blazor/includes/js-location.md)]

The following `CallJsExample1` component:

* Invokes the `convertArray` JS function with <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeAsync%2A> when selecting a button (**`Convert Array`**).
* After the JS function is called, the passed array is converted into a string. The string is returned to the component for display (`text`).

`Pages/CallJsExample1.razor`:

:::code language="razor" source="~/../blazor-samples/7.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample1.razor" highlight="2,34":::

## JavaScript API restricted to user gestures

*This section only applies to Blazor Server apps.*

Some browser JavaScript (JS) APIs can only be executed in the context of a user gesture, such as using the [`Fullscreen API` (MDN documentation)](https://developer.mozilla.org/docs/Web/API/Fullscreen_API). These APIs can't be called through the JS interop mechanism in Blazor Server apps because UI event handling is performed asynchronously and generally no longer in the context of the user gesture. The app must handle the UI event completely in JavaScript, so use `onclick` instead of Blazor's `@onclick` directive attribute.

## Invoke JavaScript functions without reading a returned value (`InvokeVoidAsync`)

Use <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A> when:

* .NET isn't required to read the result of a JavaScript (JS) call.
* JS functions return [void(0)/void 0](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/void) or [undefined](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/undefined).

Provide a `displayTickerAlert1` JS function. The function is called with <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A> and doesn't return a value:

```html
<script>
  window.displayTickerAlert1 = (symbol, price) => {
    alert(`${symbol}: $${price}!`);
  };
</script>
```

[!INCLUDE[](~/blazor/includes/js-location.md)]

### Component (`.razor`) example (`InvokeVoidAsync`)

`TickerChanged` calls the `handleTickerChanged1` method in the following `CallJsExample2` component.

`Pages/CallJsExample2.razor`:

:::code language="razor" source="~/../blazor-samples/7.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample2.razor" highlight="2,25":::

### Class (`.cs`) example (`InvokeVoidAsync`)

`JsInteropClasses1.cs`:

:::code language="csharp" source="~/../blazor-samples/7.0/BlazorSample_WebAssembly/JsInteropClasses1.cs":::

`TickerChanged` calls the `handleTickerChanged1` method in the following `CallJsExample3` component.

`Pages/CallJsExample3.razor`:

:::code language="razor" source="~/../blazor-samples/7.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample3.razor":::

## Invoke JavaScript functions and read a returned value (`InvokeAsync`)

Use <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeAsync%2A> when .NET should read the result of a JavaScript (JS) call.

Provide a `displayTickerAlert2` JS function. The following example returns a string for display by the caller:

```html
<script>
  window.displayTickerAlert2 = (symbol, price) => {
    if (price < 20) {
      alert(`${symbol}: $${price}!`);
      return "User alerted in the browser.";
    } else {
      return "User NOT alerted.";
    }
  };
</script>
```

[!INCLUDE[](~/blazor/includes/js-location.md)]

### Component (`.razor`) example (`InvokeAsync`)

`TickerChanged` calls the `handleTickerChanged2` method and displays the returned string in the following `CallJsExample4` component.

`Pages/CallJsExample4.razor`:

:::code language="razor" source="~/../blazor-samples/7.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample4.razor" highlight="2,31-34":::

### Class (`.cs`) example (`InvokeAsync`)

`JsInteropClasses2.cs`:

:::code language="csharp" source="~/../blazor-samples/7.0/BlazorSample_WebAssembly/JsInteropClasses2.cs":::

`TickerChanged` calls the `handleTickerChanged2` method and displays the returned string in the following `CallJsExample5` component.

`Pages/CallJsExample5.razor`:

:::code language="razor" source="~/../blazor-samples/7.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample5.razor" highlight="2-3,25,30,40-42,46":::

## Dynamic content generation scenarios

For dynamic content generation with [BuildRenderTree](xref:blazor/advanced-scenarios#manually-build-a-render-tree-rendertreebuilder), use the `[Inject]` attribute:

```razor
[Inject]
IJSRuntime JS { get; set; }
```

## Prerendering

[!INCLUDE[](~/blazor/includes/prerendering.md)]

## Synchronous JS interop in Blazor WebAssembly apps

[!INCLUDE[](~/blazor/includes/js-interop/synchronous-js-interop-call-js.md)]

## Location of JavaScript

Load JavaScript (JS) code using any of approaches described by the [JavaScript (JS) interoperability (interop) overview article](xref:blazor/js-interop/index#location-of-javascript):

* [Load a script in `<head>` markup](xref:blazor/js-interop/index#load-a-script-in-head-markup) (*Not generally recommended*)
* [Load a script in `<body>` markup](xref:blazor/js-interop/index#load-a-script-in-body-markup)
* [Load a script from an external JavaScript file (`.js`) collocated with a component](xref:blazor/js-interop/index#load-a-script-from-an-external-javascript-file-js-collocated-with-a-component)
* [Load a script from an external JavaScript file (`.js`)](xref:blazor/js-interop/index#load-a-script-from-an-external-javascript-file-js)
* [Inject a script before or after Blazor starts](xref:blazor/js-interop/index#inject-a-script-before-or-after-blazor-starts)

For information on isolating scripts in [JS modules](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules), see the [JavaScript isolation in JavaScript modules](#javascript-isolation-in-javascript-modules) section.

> [!WARNING]
> Don't place a `<script>` tag in a component file (`.razor`) because the `<script>` tag can't be updated dynamically.

## JavaScript isolation in JavaScript modules

Blazor enables JavaScript (JS) isolation in standard [JavaScript modules](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules) ([ECMAScript specification](https://tc39.es/ecma262/#sec-modules)). JavaScript module loading works the same way in Blazor as it does for other types of web apps, and you're free to customize how modules are defined in your app. For a guide on how to use JavaScript modules, see [MDN Web Docs: JavaScript modules](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules).

JS isolation provides the following benefits:

* Imported JS no longer pollutes the global namespace.
* Consumers of a library and components aren't required to import the related JS.

[Dynamic import with the `import()` operator](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/import) is supported with ASP.NET Core and Blazor:

```javascript
if ({CONDITION}) import("/additionalModule.js");
```

In the preceding example, the `{CONDITION}` placeholder represents a conditional check to determine if the module should be loaded.

For browser compatibility, see [Can I use: JavaScript modules: dynamic import](https://caniuse.com/es6-module-dynamic-import).

For example, the following JS module exports a JS function for showing a [browser window prompt](https://developer.mozilla.org/docs/Web/API/Window/prompt). Place the following JS code in an external JS file.

`wwwroot/scripts.js`:

```javascript
export function showPrompt(message) {
  return prompt(message, 'Type anything here');
}
```

Add the preceding JS module to an app or class library as a static web asset in the `wwwroot` folder and then import the module into the .NET code by calling <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> on the <xref:Microsoft.JSInterop.IJSRuntime> instance.

<xref:Microsoft.JSInterop.IJSRuntime> imports the module as an <xref:Microsoft.JSInterop.IJSObjectReference>, which represents a reference to a JS object from .NET code. Use the <xref:Microsoft.JSInterop.IJSObjectReference> to invoke exported JS functions from the module.

`Pages/CallJsExample6.razor`:

:::code language="razor" source="~/../blazor-samples/7.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample6.razor":::

In the preceding example:

* By convention, the `import` identifier is a special identifier used specifically for importing a JS module.
* Specify the module's external JS file using its stable static web asset path: `./{SCRIPT PATH AND FILE NAME (.js)}`, where:
  * The path segment for the current directory (`./`) is required in order to create the correct static asset path to the JS file.
  * The `{SCRIPT PATH AND FILE NAME (.js)}` placeholder is the path and file name under `wwwroot`.
* Disposes the <xref:Microsoft.JSInterop.IJSObjectReference> for [garbage collection](xref:blazor/components/lifecycle#asynchronous-iasyncdisposable) in <xref:System.IAsyncDisposable.DisposeAsync%2A?displayProperty=nameWithType>.

Dynamically importing a module requires a network request, so it can only be achieved asynchronously by calling <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A>.

`IJSInProcessObjectReference` represents a reference to a JS object whose functions can be invoked synchronously in Blazor WebAssembly apps. For more information, see the [Synchronous JS interop in Blazor WebAssembly apps](#synchronous-js-interop-in-blazor-webassembly-apps) section.

> [!NOTE]
> When the external JS file is supplied by a [Razor class library](xref:blazor/components/class-libraries), specify the module's JS file using its stable static web asset path: `./_content/{PACKAGE ID}/{SCRIPT PATH AND FILE NAME (.js)}`:
>
> * The path segment for the current directory (`./`) is required in order to create the correct static asset path to the JS file.
> * The `{PACKAGE ID}` placeholder is the library's [package ID](/nuget/create-packages/creating-a-package-msbuild#set-properties). The package ID defaults to the project's assembly name if `<PackageId>` isn't specified in the project file. In the following example, the library's assembly name is `ComponentLibrary` and the library's project file doesn't specify `<PackageId>`.
> * The `{SCRIPT PATH AND FILE NAME (.js)}` placeholder is the path and file name under `wwwroot`. In the following example, the external JS file (`script.js`) is placed in the class library's `wwwroot` folder.
>
> ```csharp
> var module = await js.InvokeAsync<IJSObjectReference>(
>     "import", "./_content/ComponentLibrary/scripts.js");
> ```
>
> For more information, see <xref:blazor/components/class-libraries>.
  
Throughout the Blazor documentation, examples use the `.js` file extension for module files, not the [newer `.mjs` file extension (RFC 9239)](https://www.rfc-editor.org/rfc/rfc9239). Our documentation continues to use the `.js` file extension for the same reasons the Mozilla Foundation's documentation continues to use the `.js` file extension. For more information, see [Aside — .mjs versus .js (MDN documentation)](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules#aside_%E2%80%94_.mjs_versus_.js).

## Capture references to elements

Some JavaScript (JS) interop scenarios require references to HTML elements. For example, a UI library may require an element reference for initialization, or you might need to call command-like APIs on an element, such as `click` or `play`.

Capture references to HTML elements in a component using the following approach:

* Add an `@ref` attribute to the HTML element.
* Define a field of type <xref:Microsoft.AspNetCore.Components.ElementReference> whose name matches the value of the `@ref` attribute.

The following example shows capturing a reference to the `username` `<input>` element:

```razor
<input @ref="username" ... />

@code {
    private ElementReference username;
}
```

> [!WARNING]
> Only use an element reference to mutate the contents of an empty element that doesn't interact with Blazor. This scenario is useful when a third-party API supplies content to the element. Because Blazor doesn't interact with the element, there's no possibility of a conflict between Blazor's representation of the element and the Document Object Model (DOM).
>
> In the following example, it's *dangerous* to mutate the contents of the unordered list (`ul`) using `MyList` via JS interop because Blazor interacts with the DOM to populate this element's list items (`<li>`) from the `Todos` object:
>
> ```razor
> <ul @ref="MyList">
>     @foreach (var item in Todos)
>     {
>         <li>@item.Text</li>
>     }
> </ul>
> ```
>
>
> Using the `MyList` element reference to merely read DOM content or trigger an event is supported.
>
> If JS interop *mutates the contents* of element `MyList` and Blazor attempts to apply diffs to the element, the diffs won't match the DOM. Modifying the contents of the list via JS interop with the `MyList` element reference is ***not supported***.
>
> For more information, see <xref:blazor/js-interop/index#interaction-with-the-document-object-model-dom>.

An <xref:Microsoft.AspNetCore.Components.ElementReference> is passed through to JS code via JS interop. The JS code receives an [`HTMLElement`](https://developer.mozilla.org/docs/Web/API/HTMLElement) instance, which it can use with normal DOM APIs. For example, the following code defines a .NET extension method (`TriggerClickEvent`) that enables sending a mouse click to an element.

The JS function `clickElement` creates a [`click`](https://developer.mozilla.org/docs/Web/API/Element/click_event) event on the passed HTML element (`element`):

```javascript
window.interopFunctions = {
  clickElement : function (element) {
    element.click();
  }
}
```

To call a JS function that doesn't return a value, use <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType>. The following code triggers a client-side [`click`](https://developer.mozilla.org/docs/Web/API/Element/click_event) event by calling the preceding JS function with the captured <xref:Microsoft.AspNetCore.Components.ElementReference>:

```razor
@inject IJSRuntime JS

<button @ref="exampleButton">Example Button</button>

<button @onclick="TriggerClick">
    Trigger click event on <code>Example Button</code>
</button>

@code {
    private ElementReference exampleButton;

    public async Task TriggerClick()
    {
        await JS.InvokeVoidAsync(
            "interopFunctions.clickElement", exampleButton);
    }
}
```

To use an extension method, create a static extension method that receives the <xref:Microsoft.JSInterop.IJSRuntime> instance:

```csharp
public static async Task TriggerClickEvent(this ElementReference elementRef, 
    IJSRuntime js)
{
    await js.InvokeVoidAsync("interopFunctions.clickElement", elementRef);
}
```

The `clickElement` method is called directly on the object. The following example assumes that the `TriggerClickEvent` method is available from the `JsInteropClasses` namespace:

```razor
@inject IJSRuntime JS
@using JsInteropClasses

<button @ref="exampleButton">Example Button</button>

<button @onclick="TriggerClick">
    Trigger click event on <code>Example Button</code>
</button>

@code {
    private ElementReference exampleButton;

    public async Task TriggerClick()
    {
        await exampleButton.TriggerClickEvent(JS);
    }
}
```

> [!IMPORTANT]
> The `exampleButton` variable is only populated after the component is rendered. If an unpopulated <xref:Microsoft.AspNetCore.Components.ElementReference> is passed to JS code, the JS code receives a value of `null`. To manipulate element references after the component has finished rendering, use the [`OnAfterRenderAsync` or `OnAfterRender` component lifecycle methods](xref:blazor/components/lifecycle#after-component-render-onafterrenderasync).

When working with generic types and returning a value, use <xref:System.Threading.Tasks.ValueTask%601>:

```csharp
public static ValueTask<T> GenericMethod<T>(this ElementReference elementRef, 
    IJSRuntime js)
{
    return js.InvokeAsync<T>("{JAVASCRIPT FUNCTION}", elementRef);
}
```

The `{JAVASCRIPT FUNCTION}` placeholder is the JS function identifier.

`GenericMethod` is called directly on the object with a type. The following example assumes that the `GenericMethod` is available from the `JsInteropClasses` namespace:

```razor
@inject IJSRuntime JS
@using JsInteropClasses

<input @ref="username" />

<button @onclick="OnClickMethod">Do something generic</button>

<p>
    returnValue: @returnValue
</p>

@code {
    private ElementReference username;
    private string? returnValue;

    private async Task OnClickMethod()
    {
        returnValue = await username.GenericMethod<string>(JS);
    }
}
```

## Reference elements across components

An <xref:Microsoft.AspNetCore.Components.ElementReference> can't be passed between components because:

* The instance is only guaranteed to exist after the component is rendered, which is during or after a component's <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A>/<xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> method executes.
* An <xref:Microsoft.AspNetCore.Components.ElementReference> is a [`struct`](/dotnet/csharp/language-reference/builtin-types/struct), which can't be passed as a [component parameter](xref:blazor/components/index#component-parameters).

For a parent component to make an element reference available to other components, the parent component can:

* Allow child components to register callbacks.
* Invoke the registered callbacks during the <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> event with the passed element reference. Indirectly, this approach allows child components to interact with the parent's element reference.

```html
<style>
    .red { color: red }
</style>
```

```html
<script>
  function setElementClass(element, className) {
    var myElement = element;
    myElement.classList.add(className);
  }
</script>
```

[!INCLUDE[](~/blazor/includes/js-location.md)]

`Pages/CallJsExample7.razor` (parent component):

:::code language="razor" source="~/../blazor-samples/7.0/BlazorSample_WebAssembly/Pages/CallJsExample7.razor" highlight="5,9":::

`Pages/CallJsExample7.razor.cs`:

:::code language="csharp" source="~/../blazor-samples/7.0/BlazorSample_WebAssembly/Pages/CallJsExample7.razor.cs":::

In the preceding example, the namespace of the app is `BlazorSample` with components in the `Pages` folder. If testing the code locally, update the namespace.

`Shared/SurveyPrompt.razor` (child component):

:::code language="razor" source="~/../blazor-samples/7.0/BlazorSample_WebAssembly/Shared/SurveyPrompt.razor":::

`Shared/SurveyPrompt.razor.cs`:

:::code language="csharp" source="~/../blazor-samples/7.0/BlazorSample_WebAssembly/Shared/SurveyPrompt.razor.cs":::

In the preceding example, the namespace of the app is `BlazorSample` with shared components in the `Shared` folder. If testing the code locally, update the namespace.

## Harden JavaScript interop calls

*This section primarily applies to Blazor Server apps, but Blazor WebAssembly apps may also set JS interop timeouts if conditions warrant it.*

In Blazor Server apps, JavaScript (JS) interop may fail due to networking errors and should be treated as unreliable. By default, Blazor Server apps use a one minute timeout for JS interop calls. If an app can tolerate a more aggressive timeout, set the timeout using one of the following approaches.

Set a global timeout in the `Program.cs` with <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.JSInteropDefaultCallTimeout?displayProperty=nameWithType>:

```csharp
builder.Services.AddServerSideBlazor(
    options => options.JSInteropDefaultCallTimeout = {TIMEOUT});
```

The `{TIMEOUT}` placeholder is a <xref:System.TimeSpan> (for example, `TimeSpan.FromSeconds(80)`).

Set a per-invocation timeout in component code. The specified timeout overrides the global timeout set by <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.JSInteropDefaultCallTimeout>:

```csharp
var result = await JS.InvokeAsync<string>("{ID}", {TIMEOUT}, new[] { "Arg1" });
```

In the preceding example:

* The `{TIMEOUT}` placeholder is a <xref:System.TimeSpan> (for example, `TimeSpan.FromSeconds(80)`).
* The `{ID}` placeholder is the identifier for the function to invoke. For example, the value `someScope.someFunction` invokes the function `window.someScope.someFunction`.

Although a common cause of JS interop failures are network failures in Blazor Server apps, per-invocation timeouts can be set for JS interop calls in Blazor WebAssembly apps. Although no SignalR circuit exists in a Blazor WebAssembly app, JS interop calls might fail for other reasons that apply in Blazor WebAssembly apps.

For more information on resource exhaustion, see <xref:blazor/security/server/threat-mitigation>.

## Avoid circular object references

Objects that contain circular references can't be serialized on the client for either:

* .NET method calls.
* JavaScript method calls from C# when the return type has circular references.

## JavaScript libraries that render UI

Sometimes you may wish to use JavaScript (JS) libraries that produce visible user interface elements within the browser Document Object Model (DOM). At first glance, this might seem difficult because Blazor's diffing system relies on having control over the tree of DOM elements and runs into errors if some external code mutates the DOM tree and invalidates its mechanism for applying diffs. This isn't a Blazor-specific limitation. The same challenge occurs with any diff-based UI framework.

Fortunately, it's straightforward to embed externally-generated UI within a Razor component UI reliably. The recommended technique is to have the component's code (`.razor` file) produce an empty element. As far as Blazor's diffing system is concerned, the element is always empty, so the renderer does not recurse into the element and instead leaves its contents alone. This makes it safe to populate the element with arbitrary externally-managed content.

The following example demonstrates the concept. Within the `if` statement when `firstRender` is `true`, interact with `unmanagedElement` outside of Blazor using JS interop. For example, call an external JS library to populate the element. Blazor leaves the element's contents alone until this component is removed. When the component is removed, the component's entire DOM subtree is also removed.

```razor
<h1>Hello! This is a Razor component rendered at @DateTime.Now</h1>

<div @ref="unmanagedElement"></div>

@code {
    private ElementReference unmanagedElement;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            ...
        }
    }
}
```

Consider the following example that renders an interactive map using [open-source Mapbox APIs](https://www.mapbox.com/).

The following JS module is placed into the app or made available from a Razor class library.

> [!NOTE]
> To create the [Mapbox](https://www.mapbox.com/) map, obtain an access token from [Mapbox Sign in](https://account.mapbox.com) and provide it where the `{ACCESS TOKEN}` appears in the following code.

`wwwroot/mapComponent.js`:

```javascript
import 'https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.js';

mapboxgl.accessToken = '{ACCESS TOKEN}';

export function addMapToElement(element) {
  return new mapboxgl.Map({
    container: element,
    style: 'mapbox://styles/mapbox/streets-v11',
    center: [-74.5, 40],
    zoom: 9
  });
}

export function setMapCenter(map, latitude, longitude) {
  map.setCenter([longitude, latitude]);
}
```

To produce correct styling, add the following stylesheet tag to the host HTML page.

Add the following `<link>` element to the `<head>` element markup ([location of `<head>` content](xref:blazor/project-structure#location-of-head-content)):

```html
<link href="https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.css" 
    rel="stylesheet" />
```

`Pages/CallJsExample8.razor`:

:::code language="razor" source="~/../blazor-samples/7.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample8.razor":::

The preceding example produces an interactive map UI. The user:

* Can drag to scroll or zoom.
* Select buttons to jump to predefined locations.

![Mapbox street map of Tokyo, Japan with buttons to select Bristol, United Kingdom and Tokyo, Japan](~/blazor/javascript-interoperability/call-javascript-from-dotnet/_static/mapbox-example.png)

In the preceding example:

* The `<div>` with `@ref="mapElement"` is left empty as far as Blazor is concerned. The `mapbox-gl.js` script can safely populate the element and modify its contents. Use this technique with any JS library that renders UI. You can embed components from a third-party JS SPA framework inside Razor components, as long as they don't try to reach out and modify other parts of the page. It is **not** safe for external JS code to modify elements that Blazor does not regard as empty.
* When using this approach, bear in mind the rules about how Blazor retains or destroys DOM elements. The component safely handles button click events and updates the existing map instance because DOM elements are retained where possible by default. If you were rendering a list of map elements from inside a `@foreach` loop, you want to use `@key` to ensure the preservation of component instances. Otherwise, changes in the list data could cause component instances to retain the state of previous instances in an undesirable manner. For more information, see [using @key to preserve elements and components](xref:blazor/components/index#use-key-to-control-the-preservation-of-elements-and-components).
* The example encapsulates JS logic and dependencies within an ES6 module and loads the module dynamically using the `import` identifier. For more information, see [JavaScript isolation in JavaScript modules](#javascript-isolation-in-javascript-modules).

## Byte array support

Blazor supports optimized byte array JavaScript (JS) interop that avoids encoding/decoding byte arrays into Base64. The following example uses JS interop to pass a byte array to JavaScript.

Provide a `receiveByteArray` JS function. The function is called with <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A> and doesn't return a value:

```html
<script>
  window.receiveByteArray = (bytes) => {
    let utf8decoder = new TextDecoder();
    let str = utf8decoder.decode(bytes);
    return str;
  };
</script>
```
[!INCLUDE[](~/blazor/includes/js-location.md)]

`Pages/CallJsExample9.razor`:

```razor
@page "/call-js-example-9"
@inject IJSRuntime JS

<h1>Call JS Example 9</h1>

<p>
    <button @onclick="SendByteArray">Send Bytes</button>
</p>

<p>
    @result
</p>

<p>
    Quote &copy;2005 <a href="https://www.uphe.com">Universal Pictures</a>:
    <a href="https://www.uphe.com/movies/serenity-2005">Serenity</a><br>
    <a href="https://www.imdb.com/name/nm0821612/">Jewel Staite on IMDB</a>
</p>

@code {
    private string? result;

    private async Task SendByteArray()
    {
        var bytes = new byte[] { 0x45, 0x76, 0x65, 0x72, 0x79, 0x74, 0x68, 0x69,
            0x6e, 0x67, 0x27, 0x73, 0x20, 0x73, 0x68, 0x69, 0x6e, 0x79, 0x2c,
            0x20, 0x43, 0x61, 0x70, 0x74, 0x69, 0x61, 0x6e, 0x2e, 0x20, 0x4e,
            0x6f, 0x74, 0x20, 0x74, 0x6f, 0x20, 0x66, 0x72, 0x65, 0x74, 0x2e };

        result = await JS.InvokeAsync<string>("receiveByteArray", bytes);
    }
}
```

For information on using a byte array when calling .NET from JavaScript, see <xref:blazor/js-interop/call-dotnet-from-javascript#byte-array-support>.

## Size limits on JavaScript interop calls

[!INCLUDE[](~/blazor/includes/js-interop/7.0/size-limits.md)]

## Stream from .NET to JavaScript

Blazor supports streaming data directly from .NET to JavaScript. Streams are created using a <xref:Microsoft.JSInterop.DotNetStreamReference>.

<xref:Microsoft.JSInterop.DotNetStreamReference> represents a .NET stream and uses the following parameters:

* `stream`: The stream sent to JavaScript.
* `leaveOpen`: Determines if the stream is left open after transmission. If a value isn't provided, `leaveOpen` defaults to `false`.

In JavaScript, use an array buffer or a readable stream to receive the data:

* Using an [`ArrayBuffer`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer):

  ```javascript
  async function streamToJavaScript(streamRef) {
    const data = await streamRef.arrayBuffer();
  }
  ```

* Using a [`ReadableStream`](https://developer.mozilla.org/docs/Web/API/ReadableStream):

  ```javascript
  async function streamToJavaScript(streamRef) {
    const stream = await streamRef.stream();
  }
  ```

In C# code:

```csharp
using var streamRef = new DotNetStreamReference(stream: {STREAM}, leaveOpen: false);
await JS.InvokeVoidAsync("streamToJavaScript", streamRef);
```

In the preceding example:

* The `{STREAM}` placeholder represents the <xref:System.IO.Stream> sent to JavaScript.
* `JS` is an injected <xref:Microsoft.JSInterop.IJSRuntime> instance.

<xref:blazor/js-interop/call-dotnet-from-javascript#stream-from-javascript-to-net> covers the reverse operation, streaming from JavaScript to .NET.

<xref:blazor/file-downloads> covers how to download a file in Blazor.

## Catch JavaScript exceptions

To catch JS exceptions, wrap the JS interop in a [`try`-`catch` block](/dotnet/csharp/fundamentals/exceptions/exception-handling) and catch a <xref:Microsoft.JSInterop.JSException>.

In the following example, the `nonFunction` JS function doesn't exist. When the function isn't found, the <xref:Microsoft.JSInterop.JSException> is trapped with a <xref:System.Exception.Message> that indicates the following error:

> `Could not find 'nonFunction' ('nonFunction' was undefined).`

`Pages/CallJsExample11.razor`:

:::code language="csharp" source="~/../blazor-samples/7.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample11.razor" highlight="28":::

## Abort a long-running JavaScript function

Use a JS [AbortController](https://developer.mozilla.org/docs/Web/API/AbortController) with a <xref:System.Threading.CancellationTokenSource> in the component to abort a long-running JavaScript function from C# code.

The following JS `Helpers` class contains a simulated long-running function, `longRunningFn`, to count continuously until the [`AbortController.signal`](https://developer.mozilla.org/docs/Web/API/AbortController/signal) indicates that [`AbortController.abort`](https://developer.mozilla.org/docs/Web/API/AbortController/abort) has been called. The `sleep` function is for demonstration purposes to simulate slow execution of the long-running function and wouldn't be present in production code. When a component calls `stopFn`, the `longRunningFn` is signalled to abort via the `while` loop conditional check on [`AbortSignal.aborted`](https://developer.mozilla.org/docs/Web/API/AbortSignal/aborted).

```html
<script>
  class Helpers {
    static #controller = new AbortController();

    static async #sleep(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    }

    static async longRunningFn() {
      var i = 0;
      while (!this.#controller.signal.aborted) {
        i++;
        console.log(`longRunningFn: ${i}`);
        await this.#sleep(1000);
      }
    }

    static stopFn() {
      this.#controller.abort();
      console.log('longRunningFn aborted!');
    }
  }

  window.Helpers = Helpers;
</script>
```

[!INCLUDE[](~/blazor/includes/js-location.md)]

The following `CallJsExample12` component:

* Invokes the JS function `longRunningFn` when the **`Start Task`** button is selected. A <xref:System.Threading.CancellationTokenSource> is used to manage the execution of the long-running function. <xref:System.Threading.CancellationToken.Register%2A?displayProperty=nameWithType> sets a JS interop call delegate to execute the JS function `stopFn` when the <xref:System.Threading.CancellationTokenSource.Token?displayProperty=nameWithType> is cancelled.
* When the **`Cancel Task`** button is selected, the <xref:System.Threading.CancellationTokenSource.Token?displayProperty=nameWithType> is cancelled with a call to <xref:System.Threading.CancellationTokenSource.Cancel%2A>.
* The <xref:System.Threading.CancellationTokenSource> is disposed in the `Dispose` method.

`Pages/CallJsExample12.razor`:

```razor
@page "/call-js-example-12"
@inject IJSRuntime JS

<h1>Cancel long-running JS interop</h1>

<p>
    <button @onclick="StartTask">Start Task</button>
    <button @onclick="CancelTask">Cancel Task</button>
</p>

@code {
    private CancellationTokenSource? cts;

    private async Task StartTask()
    {
        cts = new CancellationTokenSource();
        cts.Token.Register(() => JS.InvokeVoidAsync("Helpers.stopFn"));

        await JS.InvokeVoidAsync("Helpers.longRunningFn");
    }

    private void CancelTask()
    {
        cts?.Cancel();
    }

    public void Dispose()
    {
        cts?.Cancel();
        cts?.Dispose();
    }
}
```

A browser's [developer tools](https://developer.mozilla.org/docs/Glossary/Developer_Tools) console indicates the execution of the long-running JS function after the **`Start Task`** button is selected and when the function is aborted after the **`Cancel Task`** button is selected:

```console
longRunningFn: 1
longRunningFn: 2
longRunningFn: 3
longRunningFn aborted!
```

## JavaScript `[JSImport]`/`[JSExport]` interop

*This section applies to Blazor WebAssembly apps.*

As an alternative to interacting with JavaScript (JS) in Blazor WebAssembly apps using Blazor's JS interop mechanism based on the <xref:Microsoft.JSInterop.IJSRuntime> interface, a JS `[JSImport]`/`[JSExport]` interop API is available to apps targeting .NET 7 or later.

For more information, see <xref:blazor/js-interop/import-export-interop>. 

## Unmarshalled JavaScript interop

*This section applies to Blazor WebAssembly apps.*

Unmarshalled interop using the <xref:Microsoft.JSInterop.IJSUnmarshalledRuntime> interface is obsolete and should be replaced with JavaScript `[JSImport]`/`[JSExport]` interop.

For more information, see <xref:blazor/js-interop/import-export-interop>.

## Disposal of JavaScript interop object references

Examples throughout the JavaScript (JS) interop articles demonstrate typical object disposal patterns:

* When calling JS from .NET, as described in this article, dispose any created <xref:Microsoft.JSInterop.IJSObjectReference>/<xref:Microsoft.JSInterop.IJSInProcessObjectReference>/`JSObjectReference` either from .NET or from JS to avoid leaking JS memory.

* When calling .NET from JS, as described in <xref:blazor/js-interop/call-dotnet-from-javascript>, dispose of a created <xref:Microsoft.JSInterop.DotNetObjectReference> either from .NET or from JS to avoid leaking .NET memory.

JS interop object references are implemented as a map keyed by an identifier on the side of the JS interop call that creates the reference. When object disposal is initiated from either the .NET or JS side, Blazor removes the entry from the map, and the object can be garbage collected as long as no other strong reference to the object is present.

At a minimum, always dispose objects created on the .NET side to avoid leaking .NET managed memory.

## Document Object Model (DOM) cleanup tasks during component disposal

Don't execute JS interop code for DOM cleanup tasks during component disposal. Instead, use the [`MutationObserver`](https://developer.mozilla.org/docs/Web/API/MutationObserver) pattern in JavaScript on the client for the following reasons:

* The component may have been removed from the DOM by the time your cleanup code executes in `Dispose{Async}`.
* In a Blazor Server app, the Blazor renderer may have been disposed by the framework by the time your cleanup code executes in `Dispose{Async}`.

The [`MutationObserver`](https://developer.mozilla.org/docs/Web/API/MutationObserver) pattern allows you to run a function when an element is removed from the DOM.

## JavaScript interop calls without a circuit

[!INCLUDE[](~/blazor/includes/js-interop/circuit-disconnection.md)]

## Additional resources

* <xref:blazor/js-interop/call-dotnet-from-javascript>
* [`InteropComponent.razor` example (dotnet/AspNetCore GitHub repository `main` branch)](https://github.com/dotnet/AspNetCore/blob/main/src/Components/test/testassets/BasicTestApp/InteropComponent.razor): The `main` branch represents the product unit's current development for the next release of ASP.NET Core. To select the branch for a different release (for example, `release/5.0`), use the **Switch branches or tags** dropdown list to select the branch.
* [Blazor samples GitHub repository (`dotnet/blazor-samples`)](https://github.com/dotnet/blazor-samples)
* <xref:blazor/fundamentals/handle-errors> (*JavaScript interop* section) <!-- AUTHOR NOTE: The JavaScript interop section isn't linked because the section title changed across versions of the doc. Prior to 6.0, the section appears twice, once for Blazor Server and once for Blazor WebAssembly, each with the hosting model name in the section name. -->
* [Blazor Server threat mitigation: JavaScript functions invoked from .NET](xref:blazor/security/server/threat-mitigation#javascript-functions-invoked-from-net)

:::moniker-end

:::moniker range=">= aspnetcore-6.0 < aspnetcore-7.0"

For information on how to call .NET methods from JS, see <xref:blazor/js-interop/call-dotnet-from-javascript>.

<xref:Microsoft.JSInterop.IJSRuntime> is registered by the Blazor framework. To call into JS from .NET, inject the <xref:Microsoft.JSInterop.IJSRuntime> abstraction and call one of the following methods:

* <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A?displayProperty=nameWithType>
* <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeAsync%2A?displayProperty=nameWithType>
* <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType>

For the preceding .NET methods that invoke JS functions:

* The function identifier (`String`) is relative to the global scope (`window`). To call `window.someScope.someFunction`, the identifier is `someScope.someFunction`. There's no need to register the function before it's called.
* Pass any number of JSON-serializable arguments in `Object[]` to a JS function.
* The cancellation token (`CancellationToken`) propagates a notification that operations should be canceled.
* `TimeSpan` represents a time limit for a JS operation.
* The `TValue` return type must also be JSON serializable. `TValue` should match the .NET type that best maps to the JSON type returned.
* A [JS `Promise`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) is returned for `InvokeAsync` methods. `InvokeAsync` unwraps the [`Promise`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) and returns the value awaited by the [`Promise`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise).

For Blazor apps with prerendering enabled, calling into JS isn't possible during prerendering. For more information, see the [Prerendering](#prerendering) section.

The following example is based on [`TextDecoder`](https://developer.mozilla.org/docs/Web/API/TextDecoder), a JS-based decoder. The example demonstrates how to invoke a JS function from a C# method that offloads a requirement from developer code to an existing JS API. The JS function accepts a byte array from a C# method, decodes the array, and returns the text to the component for display.

```html
<script>
  window.convertArray = (win1251Array) => {
    var win1251decoder = new TextDecoder('windows-1251');
    var bytes = new Uint8Array(win1251Array);
    var decodedArray = win1251decoder.decode(bytes);
    console.log(decodedArray);
    return decodedArray;
  };
</script>
```

[!INCLUDE[](~/blazor/includes/js-location.md)]

The following `CallJsExample1` component:

* Invokes the `convertArray` JS function with <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeAsync%2A> when selecting a button (**`Convert Array`**).
* After the JS function is called, the passed array is converted into a string. The string is returned to the component for display (`text`).

`Pages/CallJsExample1.razor`:

:::code language="razor" source="~/../blazor-samples/6.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample1.razor" highlight="2,34":::

## JavaScript API restricted to user gestures

*This section only applies to Blazor Server apps.*

Some browser JavaScript (JS) APIs can only be executed in the context of a user gesture, such as using the [`Fullscreen API` (MDN documentation)](https://developer.mozilla.org/docs/Web/API/Fullscreen_API). These APIs can't be called through the JS interop mechanism in Blazor Server apps because UI event handling is performed asynchronously and generally no longer in the context of the user gesture. The app must handle the UI event completely in JavaScript, so use `onclick` instead of Blazor's `@onclick` directive attribute.

## Invoke JavaScript functions without reading a returned value (`InvokeVoidAsync`)

Use <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A> when:

* .NET isn't required to read the result of a JavaScript (JS) call.
* JS functions return [void(0)/void 0](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/void) or [undefined](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/undefined).

Provide a `displayTickerAlert1` JS function. The function is called with <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A> and doesn't return a value:

```html
<script>
  window.displayTickerAlert1 = (symbol, price) => {
    alert(`${symbol}: $${price}!`);
  };
</script>
```

[!INCLUDE[](~/blazor/includes/js-location.md)]

### Component (`.razor`) example (`InvokeVoidAsync`)

`TickerChanged` calls the `handleTickerChanged1` method in the following `CallJsExample2` component.

`Pages/CallJsExample2.razor`:

:::code language="razor" source="~/../blazor-samples/6.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample2.razor" highlight="2,25":::

### Class (`.cs`) example (`InvokeVoidAsync`)

`JsInteropClasses1.cs`:

:::code language="csharp" source="~/../blazor-samples/6.0/BlazorSample_WebAssembly/JsInteropClasses1.cs":::

`TickerChanged` calls the `handleTickerChanged1` method in the following `CallJsExample3` component.

`Pages/CallJsExample3.razor`:

:::code language="razor" source="~/../blazor-samples/6.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample3.razor":::

## Invoke JavaScript functions and read a returned value (`InvokeAsync`)

Use <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeAsync%2A> when .NET should read the result of a JavaScript (JS) call.

Provide a `displayTickerAlert2` JS function. The following example returns a string for display by the caller:

```html
<script>
  window.displayTickerAlert2 = (symbol, price) => {
    if (price < 20) {
      alert(`${symbol}: $${price}!`);
      return "User alerted in the browser.";
    } else {
      return "User NOT alerted.";
    }
  };
</script>
```

[!INCLUDE[](~/blazor/includes/js-location.md)]

### Component (`.razor`) example (`InvokeAsync`)

`TickerChanged` calls the `handleTickerChanged2` method and displays the returned string in the following `CallJsExample4` component.

`Pages/CallJsExample4.razor`:

:::code language="razor" source="~/../blazor-samples/6.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample4.razor" highlight="2,31-34":::

### Class (`.cs`) example (`InvokeAsync`)

`JsInteropClasses2.cs`:

:::code language="csharp" source="~/../blazor-samples/6.0/BlazorSample_WebAssembly/JsInteropClasses2.cs":::

`TickerChanged` calls the `handleTickerChanged2` method and displays the returned string in the following `CallJsExample5` component.

`Pages/CallJsExample5.razor`:

:::code language="razor" source="~/../blazor-samples/6.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample5.razor" highlight="2-3,25,30,40-42,46":::

## Dynamic content generation scenarios

For dynamic content generation with [BuildRenderTree](xref:blazor/advanced-scenarios#manually-build-a-render-tree-rendertreebuilder), use the `[Inject]` attribute:

```razor
[Inject]
IJSRuntime JS { get; set; }
```

## Prerendering

[!INCLUDE[](~/blazor/includes/prerendering.md)]

## Synchronous JS interop in Blazor WebAssembly apps

[!INCLUDE[](~/blazor/includes/js-interop/synchronous-js-interop-call-js.md)]

## Location of JavaScript

Load JavaScript (JS) code using any of approaches described by the [JavaScript (JS) interoperability (interop) overview article](xref:blazor/js-interop/index#location-of-javascript):

* [Load a script in `<head>` markup](xref:blazor/js-interop/index#load-a-script-in-head-markup) (*Not generally recommended*)
* [Load a script in `<body>` markup](xref:blazor/js-interop/index#load-a-script-in-body-markup)
* [Load a script from an external JavaScript file (`.js`) collocated with a component](xref:blazor/js-interop/index#load-a-script-from-an-external-javascript-file-js-collocated-with-a-component)
* [Load a script from an external JavaScript file (`.js`)](xref:blazor/js-interop/index#load-a-script-from-an-external-javascript-file-js)
* [Inject a script before or after Blazor starts](xref:blazor/js-interop/index#inject-a-script-before-or-after-blazor-starts)

For information on isolating scripts in [JS modules](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules), see the [JavaScript isolation in JavaScript modules](#javascript-isolation-in-javascript-modules) section.

> [!WARNING]
> Don't place a `<script>` tag in a component file (`.razor`) because the `<script>` tag can't be updated dynamically.

## JavaScript isolation in JavaScript modules

Blazor enables JavaScript (JS) isolation in standard [JavaScript modules](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules) ([ECMAScript specification](https://tc39.es/ecma262/#sec-modules)). JavaScript module loading works the same way in Blazor as it does for other types of web apps, and you're free to customize how modules are defined in your app. For a guide on how to use JavaScript modules, see [MDN Web Docs: JavaScript modules](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules).

JS isolation provides the following benefits:

* Imported JS no longer pollutes the global namespace.
* Consumers of a library and components aren't required to import the related JS.

[Dynamic import with the `import()` operator](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/import) is supported with ASP.NET Core and Blazor:

```javascript
if ({CONDITION}) import("/additionalModule.js");
```

In the preceding example, the `{CONDITION}` placeholder represents a conditional check to determine if the module should be loaded.

For browser compatibility, see [Can I use: JavaScript modules: dynamic import](https://caniuse.com/es6-module-dynamic-import).

For example, the following JS module exports a JS function for showing a [browser window prompt](https://developer.mozilla.org/docs/Web/API/Window/prompt). Place the following JS code in an external JS file.

`wwwroot/scripts.js`:

```javascript
export function showPrompt(message) {
  return prompt(message, 'Type anything here');
}
```

Add the preceding JS module to an app or class library as a static web asset in the `wwwroot` folder and then import the module into the .NET code by calling <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> on the <xref:Microsoft.JSInterop.IJSRuntime> instance.

<xref:Microsoft.JSInterop.IJSRuntime> imports the module as an <xref:Microsoft.JSInterop.IJSObjectReference>, which represents a reference to a JS object from .NET code. Use the <xref:Microsoft.JSInterop.IJSObjectReference> to invoke exported JS functions from the module.

`Pages/CallJsExample6.razor`:

:::code language="razor" source="~/../blazor-samples/6.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample6.razor":::

In the preceding example:

* By convention, the `import` identifier is a special identifier used specifically for importing a JS module.
* Specify the module's external JS file using its stable static web asset path: `./{SCRIPT PATH AND FILE NAME (.js)}`, where:
  * The path segment for the current directory (`./`) is required in order to create the correct static asset path to the JS file.
  * The `{SCRIPT PATH AND FILE NAME (.js)}` placeholder is the path and file name under `wwwroot`.
* Disposes the <xref:Microsoft.JSInterop.IJSObjectReference> for [garbage collection](xref:blazor/components/lifecycle#asynchronous-iasyncdisposable) in <xref:System.IAsyncDisposable.DisposeAsync%2A?displayProperty=nameWithType>.

Dynamically importing a module requires a network request, so it can only be achieved asynchronously by calling <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A>.

`IJSInProcessObjectReference` represents a reference to a JS object whose functions can be invoked synchronously in Blazor WebAssembly apps. For more information, see the [Synchronous JS interop in Blazor WebAssembly apps](#synchronous-js-interop-in-blazor-webassembly-apps) section.

> [!NOTE]
> When the external JS file is supplied by a [Razor class library](xref:blazor/components/class-libraries), specify the module's JS file using its stable static web asset path: `./_content/{PACKAGE ID}/{SCRIPT PATH AND FILE NAME (.js)}`:
>
> * The path segment for the current directory (`./`) is required in order to create the correct static asset path to the JS file.
> * The `{PACKAGE ID}` placeholder is the library's [package ID](/nuget/create-packages/creating-a-package-msbuild#set-properties). The package ID defaults to the project's assembly name if `<PackageId>` isn't specified in the project file. In the following example, the library's assembly name is `ComponentLibrary` and the library's project file doesn't specify `<PackageId>`.
> * The `{SCRIPT PATH AND FILE NAME (.js)}` placeholder is the path and file name under `wwwroot`. In the following example, the external JS file (`script.js`) is placed in the class library's `wwwroot` folder.
>
> ```csharp
> var module = await js.InvokeAsync<IJSObjectReference>(
>     "import", "./_content/ComponentLibrary/scripts.js");
> ```
>
> For more information, see <xref:blazor/components/class-libraries>.
  
Throughout the Blazor documentation, examples use the `.js` file extension for module files, not the [newer `.mjs` file extension (RFC 9239)](https://www.rfc-editor.org/rfc/rfc9239). Our documentation continues to use the `.js` file extension for the same reasons the Mozilla Foundation's documentation continues to use the `.js` file extension. For more information, see [Aside — .mjs versus .js (MDN documentation)](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules#aside_%E2%80%94_.mjs_versus_.js).

## Capture references to elements

Some JavaScript (JS) interop scenarios require references to HTML elements. For example, a UI library may require an element reference for initialization, or you might need to call command-like APIs on an element, such as `click` or `play`.

Capture references to HTML elements in a component using the following approach:

* Add an `@ref` attribute to the HTML element.
* Define a field of type <xref:Microsoft.AspNetCore.Components.ElementReference> whose name matches the value of the `@ref` attribute.

The following example shows capturing a reference to the `username` `<input>` element:

```razor
<input @ref="username" ... />

@code {
    private ElementReference username;
}
```

> [!WARNING]
> Only use an element reference to mutate the contents of an empty element that doesn't interact with Blazor. This scenario is useful when a third-party API supplies content to the element. Because Blazor doesn't interact with the element, there's no possibility of a conflict between Blazor's representation of the element and the Document Object Model (DOM).
>
> In the following example, it's *dangerous* to mutate the contents of the unordered list (`ul`) because Blazor interacts with the DOM to populate this element's list items (`<li>`) from the `Todos` object:
>
> ```razor
> <ul @ref="MyList">
>     @foreach (var item in Todos)
>     {
>         <li>@item.Text</li>
>     }
> </ul>
> ```
>
> Using the `MyList` element reference to merely read DOM content or trigger an event is supported.
>
> If JS interop *mutates the contents* of element `MyList` and Blazor attempts to apply diffs to the element, the diffs won't match the DOM. Modifying the contents of the list via JS interop with the `MyList` element reference is ***not supported***.
>
> For more information, see <xref:blazor/js-interop/index#interaction-with-the-document-object-model-dom>.

An <xref:Microsoft.AspNetCore.Components.ElementReference> is passed through to JS code via JS interop. The JS code receives an [`HTMLElement`](https://developer.mozilla.org/docs/Web/API/HTMLElement) instance, which it can use with normal DOM APIs. For example, the following code defines a .NET extension method (`TriggerClickEvent`) that enables sending a mouse click to an element.

The JS function `clickElement` creates a [`click`](https://developer.mozilla.org/docs/Web/API/Element/click_event) event on the passed HTML element (`element`):

```javascript
window.interopFunctions = {
  clickElement : function (element) {
    element.click();
  }
}
```

To call a JS function that doesn't return a value, use <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType>. The following code triggers a client-side [`click`](https://developer.mozilla.org/docs/Web/API/Element/click_event) event by calling the preceding JS function with the captured <xref:Microsoft.AspNetCore.Components.ElementReference>:

```razor
@inject IJSRuntime JS

<button @ref="exampleButton">Example Button</button>

<button @onclick="TriggerClick">
    Trigger click event on <code>Example Button</code>
</button>

@code {
    private ElementReference exampleButton;

    public async Task TriggerClick()
    {
        await JS.InvokeVoidAsync(
            "interopFunctions.clickElement", exampleButton);
    }
}
```

To use an extension method, create a static extension method that receives the <xref:Microsoft.JSInterop.IJSRuntime> instance:

```csharp
public static async Task TriggerClickEvent(this ElementReference elementRef, 
    IJSRuntime js)
{
    await js.InvokeVoidAsync("interopFunctions.clickElement", elementRef);
}
```

The `clickElement` method is called directly on the object. The following example assumes that the `TriggerClickEvent` method is available from the `JsInteropClasses` namespace:

```razor
@inject IJSRuntime JS
@using JsInteropClasses

<button @ref="exampleButton">Example Button</button>

<button @onclick="TriggerClick">
    Trigger click event on <code>Example Button</code>
</button>

@code {
    private ElementReference exampleButton;

    public async Task TriggerClick()
    {
        await exampleButton.TriggerClickEvent(JS);
    }
}
```

> [!IMPORTANT]
> The `exampleButton` variable is only populated after the component is rendered. If an unpopulated <xref:Microsoft.AspNetCore.Components.ElementReference> is passed to JS code, the JS code receives a value of `null`. To manipulate element references after the component has finished rendering, use the [`OnAfterRenderAsync` or `OnAfterRender` component lifecycle methods](xref:blazor/components/lifecycle#after-component-render-onafterrenderasync).

When working with generic types and returning a value, use <xref:System.Threading.Tasks.ValueTask%601>:

```csharp
public static ValueTask<T> GenericMethod<T>(this ElementReference elementRef, 
    IJSRuntime js)
{
    return js.InvokeAsync<T>("{JAVASCRIPT FUNCTION}", elementRef);
}
```

The `{JAVASCRIPT FUNCTION}` placeholder is the JS function identifier.

`GenericMethod` is called directly on the object with a type. The following example assumes that the `GenericMethod` is available from the `JsInteropClasses` namespace:

```razor
@inject IJSRuntime JS
@using JsInteropClasses

<input @ref="username" />

<button @onclick="OnClickMethod">Do something generic</button>

<p>
    returnValue: @returnValue
</p>

@code {
    private ElementReference username;
    private string? returnValue;

    private async Task OnClickMethod()
    {
        returnValue = await username.GenericMethod<string>(JS);
    }
}
```

## Reference elements across components

An <xref:Microsoft.AspNetCore.Components.ElementReference> can't be passed between components because:

* The instance is only guaranteed to exist after the component is rendered, which is during or after a component's <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A>/<xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> method executes.
* An <xref:Microsoft.AspNetCore.Components.ElementReference> is a [`struct`](/dotnet/csharp/language-reference/builtin-types/struct), which can't be passed as a [component parameter](xref:blazor/components/index#component-parameters).

For a parent component to make an element reference available to other components, the parent component can:

* Allow child components to register callbacks.
* Invoke the registered callbacks during the <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> event with the passed element reference. Indirectly, this approach allows child components to interact with the parent's element reference.

```html
<style>
    .red { color: red }
</style>
```

```html
<script>
  function setElementClass(element, className) {
    var myElement = element;
    myElement.classList.add(className);
  }
</script>
```

[!INCLUDE[](~/blazor/includes/js-location.md)]

`Pages/CallJsExample7.razor` (parent component):

:::code language="razor" source="~/../blazor-samples/6.0/BlazorSample_WebAssembly/Pages/CallJsExample7.razor" highlight="5,9":::

`Pages/CallJsExample7.razor.cs`:

:::code language="csharp" source="~/../blazor-samples/6.0/BlazorSample_WebAssembly/Pages/CallJsExample7.razor.cs":::

In the preceding example, the namespace of the app is `BlazorSample` with components in the `Pages` folder. If testing the code locally, update the namespace.

`Shared/SurveyPrompt.razor` (child component):

:::code language="razor" source="~/../blazor-samples/6.0/BlazorSample_WebAssembly/Shared/SurveyPrompt.razor":::

`Shared/SurveyPrompt.razor.cs`:

:::code language="csharp" source="~/../blazor-samples/6.0/BlazorSample_WebAssembly/Shared/SurveyPrompt.razor.cs":::

In the preceding example, the namespace of the app is `BlazorSample` with shared components in the `Shared` folder. If testing the code locally, update the namespace.

## Harden JavaScript interop calls

*This section primarily applies to Blazor Server apps, but Blazor WebAssembly apps may also set JS interop timeouts if conditions warrant it.*

In Blazor Server apps, JavaScript (JS) interop may fail due to networking errors and should be treated as unreliable. By default, Blazor Server apps use a one minute timeout for JS interop calls. If an app can tolerate a more aggressive timeout, set the timeout using one of the following approaches.

Set a global timeout in the `Program.cs` with <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.JSInteropDefaultCallTimeout?displayProperty=nameWithType>:

```csharp
builder.Services.AddServerSideBlazor(
    options => options.JSInteropDefaultCallTimeout = {TIMEOUT});
```

The `{TIMEOUT}` placeholder is a <xref:System.TimeSpan> (for example, `TimeSpan.FromSeconds(80)`).

Set a per-invocation timeout in component code. The specified timeout overrides the global timeout set by <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.JSInteropDefaultCallTimeout>:

```csharp
var result = await JS.InvokeAsync<string>("{ID}", {TIMEOUT}, new[] { "Arg1" });
```

In the preceding example:

* The `{TIMEOUT}` placeholder is a <xref:System.TimeSpan> (for example, `TimeSpan.FromSeconds(80)`).
* The `{ID}` placeholder is the identifier for the function to invoke. For example, the value `someScope.someFunction` invokes the function `window.someScope.someFunction`.

Although a common cause of JS interop failures are network failures in Blazor Server apps, per-invocation timeouts can be set for JS interop calls in Blazor WebAssembly apps. Although no SignalR circuit exists in a Blazor WebAssembly app, JS interop calls might fail for other reasons that apply in Blazor WebAssembly apps.

For more information on resource exhaustion, see <xref:blazor/security/server/threat-mitigation>.

## Avoid circular object references

Objects that contain circular references can't be serialized on the client for either:

* .NET method calls.
* JavaScript method calls from C# when the return type has circular references.

## JavaScript libraries that render UI

Sometimes you may wish to use JavaScript (JS) libraries that produce visible user interface elements within the browser Document Object Model (DOM). At first glance, this might seem difficult because Blazor's diffing system relies on having control over the tree of DOM elements and runs into errors if some external code mutates the DOM tree and invalidates its mechanism for applying diffs. This isn't a Blazor-specific limitation. The same challenge occurs with any diff-based UI framework.

Fortunately, it's straightforward to embed externally-generated UI within a Razor component UI reliably. The recommended technique is to have the component's code (`.razor` file) produce an empty element. As far as Blazor's diffing system is concerned, the element is always empty, so the renderer does not recurse into the element and instead leaves its contents alone. This makes it safe to populate the element with arbitrary externally-managed content.

The following example demonstrates the concept. Within the `if` statement when `firstRender` is `true`, interact with `unmanagedElement` outside of Blazor using JS interop. For example, call an external JS library to populate the element. Blazor leaves the element's contents alone until this component is removed. When the component is removed, the component's entire DOM subtree is also removed.

```razor
<h1>Hello! This is a Razor component rendered at @DateTime.Now</h1>

<div @ref="unmanagedElement"></div>

@code {
    private ElementReference unmanagedElement;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            ...
        }
    }
}
```

Consider the following example that renders an interactive map using [open-source Mapbox APIs](https://www.mapbox.com/).

The following JS module is placed into the app or made available from a Razor class library.

> [!NOTE]
> To create the [Mapbox](https://www.mapbox.com/) map, obtain an access token from [Mapbox Sign in](https://account.mapbox.com) and provide it where the `{ACCESS TOKEN}` appears in the following code.

`wwwroot/mapComponent.js`:

```javascript
import 'https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.js';

mapboxgl.accessToken = '{ACCESS TOKEN}';

export function addMapToElement(element) {
  return new mapboxgl.Map({
    container: element,
    style: 'mapbox://styles/mapbox/streets-v11',
    center: [-74.5, 40],
    zoom: 9
  });
}

export function setMapCenter(map, latitude, longitude) {
  map.setCenter([longitude, latitude]);
}
```

To produce correct styling, add the following stylesheet tag to the host HTML page.

Add the following `<link>` element to the `<head>` element markup ([location of `<head>` content](xref:blazor/project-structure#location-of-head-content)):

```html
<link href="https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.css" 
    rel="stylesheet" />
```

`Pages/CallJsExample8.razor`:

:::code language="razor" source="~/../blazor-samples/6.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample8.razor":::

The preceding example produces an interactive map UI. The user:

* Can drag to scroll or zoom.
* Select buttons to jump to predefined locations.

![Mapbox street map of Tokyo, Japan with buttons to select Bristol, United Kingdom and Tokyo, Japan](~/blazor/javascript-interoperability/call-javascript-from-dotnet/_static/mapbox-example.png)

In the preceding example:

* The `<div>` with `@ref="mapElement"` is left empty as far as Blazor is concerned. The `mapbox-gl.js` script can safely populate the element and modify its contents. Use this technique with any JS library that renders UI. You can embed components from a third-party JS SPA framework inside Razor components, as long as they don't try to reach out and modify other parts of the page. It is **not** safe for external JS code to modify elements that Blazor does not regard as empty.
* When using this approach, bear in mind the rules about how Blazor retains or destroys DOM elements. The component safely handles button click events and updates the existing map instance because DOM elements are retained where possible by default. If you were rendering a list of map elements from inside a `@foreach` loop, you want to use `@key` to ensure the preservation of component instances. Otherwise, changes in the list data could cause component instances to retain the state of previous instances in an undesirable manner. For more information, see [using @key to preserve elements and components](xref:blazor/components/index#use-key-to-control-the-preservation-of-elements-and-components).
* The example encapsulates JS logic and dependencies within an ES6 module and loads the module dynamically using the `import` identifier. For more information, see [JavaScript isolation in JavaScript modules](#javascript-isolation-in-javascript-modules).

## Byte array support

Blazor supports optimized byte array JavaScript (JS) interop that avoids encoding/decoding byte arrays into Base64. The following example uses JS interop to pass a byte array to JavaScript.

Provide a `receiveByteArray` JS function. The function is called with <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A> and doesn't return a value:

```html
<script>
  window.receiveByteArray = (bytes) => {
    let utf8decoder = new TextDecoder();
    let str = utf8decoder.decode(bytes);
    return str;
  };
</script>
```
[!INCLUDE[](~/blazor/includes/js-location.md)]

`Pages/CallJsExample9.razor`:

```razor
@page "/call-js-example-9"
@inject IJSRuntime JS

<h1>Call JS Example 9</h1>

<p>
    <button @onclick="SendByteArray">Send Bytes</button>
</p>

<p>
    @result
</p>

<p>
    Quote &copy;2005 <a href="https://www.uphe.com">Universal Pictures</a>:
    <a href="https://www.uphe.com/movies/serenity-2005">Serenity</a><br>
    <a href="https://www.imdb.com/name/nm0821612/">Jewel Staite on IMDB</a>
</p>

@code {
    private string? result;

    private async Task SendByteArray()
    {
        var bytes = new byte[] { 0x45, 0x76, 0x65, 0x72, 0x79, 0x74, 0x68, 0x69,
            0x6e, 0x67, 0x27, 0x73, 0x20, 0x73, 0x68, 0x69, 0x6e, 0x79, 0x2c,
            0x20, 0x43, 0x61, 0x70, 0x74, 0x69, 0x61, 0x6e, 0x2e, 0x20, 0x4e,
            0x6f, 0x74, 0x20, 0x74, 0x6f, 0x20, 0x66, 0x72, 0x65, 0x74, 0x2e };

        result = await JS.InvokeAsync<string>("receiveByteArray", bytes);
    }
}
```

For information on using a byte array when calling .NET from JavaScript, see <xref:blazor/js-interop/call-dotnet-from-javascript#byte-array-support>.

## Size limits on JavaScript interop calls

[!INCLUDE[](~/blazor/includes/js-interop/6.0/size-limits.md)]

## Unmarshalled JavaScript interop

Blazor WebAssembly components may experience poor performance when .NET objects are serialized for JavaScript (JS) interop and either of the following are true:

* A high volume of .NET objects are rapidly serialized. For example, poor performance may result when JS interop calls are made on the basis of moving an input device, such as spinning a mouse wheel.
* Large .NET objects or many .NET objects must be serialized for JS interop. For example, poor performance may result when JS interop calls require serializing dozens of files.

<xref:Microsoft.JSInterop.IJSUnmarshalledObjectReference> represents a reference to an JS object whose functions can be invoked without the overhead of serializing .NET data.

In the following example:

* A [struct](/dotnet/csharp/language-reference/builtin-types/struct) containing a string and an integer is passed unserialized to JS.
* JS functions process the data and return either a boolean or string to the caller.
* A JS string isn't directly convertible into a .NET `string` object. The `unmarshalledFunctionReturnString` function calls `BINDING.js_string_to_mono_string` to manage the conversion of a JS string.

> [!NOTE]
> The following examples aren't typical use cases for this scenario because the [struct](/dotnet/csharp/language-reference/builtin-types/struct) passed to JS doesn't result in poor component performance. The example uses a small object merely to demonstrate the concepts for passing unserialized .NET data.

```javascript
<script>
  window.returnObjectReference = () => {
    return {
      unmarshalledFunctionReturnBoolean: function (fields) {
        const name = Blazor.platform.readStringField(fields, 0);
        const year = Blazor.platform.readInt32Field(fields, 8);
    
        return name === "Brigadier Alistair Gordon Lethbridge-Stewart" &&
            year === 1968;
      },
      unmarshalledFunctionReturnString: function (fields) {
        const name = Blazor.platform.readStringField(fields, 0);
        const year = Blazor.platform.readInt32Field(fields, 8);

        return BINDING.js_string_to_mono_string(`Hello, ${name} (${year})!`);
      }
    };
  }
</script>
```

[!INCLUDE[](~/blazor/includes/js-location.md)]

> [!WARNING]
> The `js_string_to_mono_string` function name, behavior, and existence is subject to change in a future release of .NET. For example:
>
> * The function is likely to be renamed.
> * The function itself might be removed in favor of automatic conversion of strings by the framework.

`Pages/CallJsExample10.razor`:

:::code language="razor" source="~/../blazor-samples/6.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample10.razor":::

If an <xref:Microsoft.JSInterop.IJSUnmarshalledObjectReference> instance isn't disposed in C# code, it can be disposed in JS. The following `dispose` function disposes the object reference when called from JS:

```javascript
window.exampleJSObjectReferenceNotDisposedInCSharp = () => {
  return {
    dispose: function () {
      DotNet.disposeJSObjectReference(this);
    },

    ...
  };
}
```

Array types can be converted from JS objects into .NET objects using `js_typed_array_to_array`, but the JS array must be a typed array. Arrays from JS can be read in C# code as a .NET object array (`object[]`).

Other data types, such as string arrays, can be converted but require creating a new Mono array object (`mono_obj_array_new`) and setting its value (`mono_obj_array_set`).

> [!WARNING]
> JS functions provided by the Blazor framework, such as `js_typed_array_to_array`, `mono_obj_array_new`, and `mono_obj_array_set`, are subject to name changes, behavioral changes, or removal in future releases of .NET.

## Stream from .NET to JavaScript

Blazor supports streaming data directly from .NET to JavaScript. Streams are created using a <xref:Microsoft.JSInterop.DotNetStreamReference>.

<xref:Microsoft.JSInterop.DotNetStreamReference> represents a .NET stream and uses the following parameters:

* `stream`: The stream sent to JavaScript.
* `leaveOpen`: Determines if the stream is left open after transmission. If a value isn't provided, `leaveOpen` defaults to `false`.

In JavaScript, use an array buffer or a readable stream to receive the data:

* Using an [`ArrayBuffer`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer):

  ```javascript
  async function streamToJavaScript(streamRef) {
    const data = await streamRef.arrayBuffer();
  }
  ```

* Using a [`ReadableStream`](https://developer.mozilla.org/docs/Web/API/ReadableStream):

  ```javascript
  async function streamToJavaScript(streamRef) {
    const stream = await streamRef.stream();
  }
  ```

In C# code:

```csharp
using var streamRef = new DotNetStreamReference(stream: {STREAM}, leaveOpen: false);
await JS.InvokeVoidAsync("streamToJavaScript", streamRef);
```

In the preceding example:

* The `{STREAM}` placeholder represents the <xref:System.IO.Stream> sent to JavaScript.
* `JS` is an injected <xref:Microsoft.JSInterop.IJSRuntime> instance.

<xref:blazor/js-interop/call-dotnet-from-javascript#stream-from-javascript-to-net> covers the reverse operation, streaming from JavaScript to .NET.

<xref:blazor/file-downloads> covers how to download a file in Blazor.

## Catch JavaScript exceptions

To catch JS exceptions, wrap the JS interop in a [`try`-`catch` block](/dotnet/csharp/fundamentals/exceptions/exception-handling) and catch a <xref:Microsoft.JSInterop.JSException>.

In the following example, the `nonFunction` JS function doesn't exist. When the function isn't found, the <xref:Microsoft.JSInterop.JSException> is trapped with a <xref:System.Exception.Message> that indicates the following error:

> `Could not find 'nonFunction' ('nonFunction' was undefined).`

`Pages/CallJsExample11.razor`:

:::code language="csharp" source="~/../blazor-samples/6.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample11.razor" highlight="28":::

## Abort a long-running JavaScript function

Use a JS [AbortController](https://developer.mozilla.org/docs/Web/API/AbortController) with a <xref:System.Threading.CancellationTokenSource> in the component to abort a long-running JavaScript function from C# code.

The following JS `Helpers` class contains a simulated long-running function, `longRunningFn`, to count continuously until the [`AbortController.signal`](https://developer.mozilla.org/docs/Web/API/AbortController/signal) indicates that [`AbortController.abort`](https://developer.mozilla.org/docs/Web/API/AbortController/abort) has been called. The `sleep` function is for demonstration purposes to simulate slow execution of the long-running function and wouldn't be present in production code. When a component calls `stopFn`, the `longRunningFn` is signalled to abort via the `while` loop conditional check on [`AbortSignal.aborted`](https://developer.mozilla.org/docs/Web/API/AbortSignal/aborted).

```html
<script>
  class Helpers {
    static #controller = new AbortController();

    static async #sleep(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    }

    static async longRunningFn() {
      var i = 0;
      while (!this.#controller.signal.aborted) {
        i++;
        console.log(`longRunningFn: ${i}`);
        await this.#sleep(1000);
      }
    }

    static stopFn() {
      this.#controller.abort();
      console.log('longRunningFn aborted!');
    }
  }

  window.Helpers = Helpers;
</script>
```

[!INCLUDE[](~/blazor/includes/js-location.md)]

The following `CallJsExample12` component:

* Invokes the JS function `longRunningFn` when the **`Start Task`** button is selected. A <xref:System.Threading.CancellationTokenSource> is used to manage the execution of the long-running function. <xref:System.Threading.CancellationToken.Register%2A?displayProperty=nameWithType> sets a JS interop call delegate to execute the JS function `stopFn` when the <xref:System.Threading.CancellationTokenSource.Token?displayProperty=nameWithType> is cancelled.
* When the **`Cancel Task`** button is selected, the <xref:System.Threading.CancellationTokenSource.Token?displayProperty=nameWithType> is cancelled with a call to <xref:System.Threading.CancellationTokenSource.Cancel%2A>.
* The <xref:System.Threading.CancellationTokenSource> is disposed in the `Dispose` method.

`Pages/CallJsExample12.razor`:

```razor
@page "/call-js-example-12"
@inject IJSRuntime JS

<h1>Cancel long-running JS interop</h1>

<p>
    <button @onclick="StartTask">Start Task</button>
    <button @onclick="CancelTask">Cancel Task</button>
</p>

@code {
    private CancellationTokenSource? cts;

    private async Task StartTask()
    {
        cts = new CancellationTokenSource();
        cts.Token.Register(() => JS.InvokeVoidAsync("Helpers.stopFn"));

        await JS.InvokeVoidAsync("Helpers.longRunningFn");
    }

    private void CancelTask()
    {
        cts?.Cancel();
    }

    public void Dispose()
    {
        cts?.Cancel();
        cts?.Dispose();
    }
}
```

A browser's [developer tools](https://developer.mozilla.org/docs/Glossary/Developer_Tools) console indicates the execution of the long-running JS function after the **`Start Task`** button is selected and when the function is aborted after the **`Cancel Task`** button is selected:

```console
longRunningFn: 1
longRunningFn: 2
longRunningFn: 3
longRunningFn aborted!
```

## Document Object Model (DOM) cleanup tasks during component disposal

Don't execute JS interop code for DOM cleanup tasks during component disposal. Instead, use the [`MutationObserver`](https://developer.mozilla.org/docs/Web/API/MutationObserver) pattern in JavaScript on the client for the following reasons:

* The component may have been removed from the DOM by the time your cleanup code executes in `Dispose{Async}`.
* In a Blazor Server app, the Blazor renderer may have been disposed by the framework by the time your cleanup code executes in `Dispose{Async}`.

The [`MutationObserver`](https://developer.mozilla.org/docs/Web/API/MutationObserver) pattern allows you to run a function when an element is removed from the DOM.

## JavaScript interop calls without a circuit

[!INCLUDE[](~/blazor/includes/js-interop/circuit-disconnection.md)]

## Additional resources

* <xref:blazor/js-interop/call-dotnet-from-javascript>
* [`InteropComponent.razor` example (dotnet/AspNetCore GitHub repository `main` branch)](https://github.com/dotnet/AspNetCore/blob/main/src/Components/test/testassets/BasicTestApp/InteropComponent.razor): The `main` branch represents the product unit's current development for the next release of ASP.NET Core. To select the branch for a different release (for example, `release/5.0`), use the **Switch branches or tags** dropdown list to select the branch.
* [Blazor samples GitHub repository (`dotnet/blazor-samples`)](https://github.com/dotnet/blazor-samples)
* <xref:blazor/fundamentals/handle-errors> (*JavaScript interop* section) <!-- AUTHOR NOTE: The JavaScript interop section isn't linked because the section title changed across versions of the doc. Prior to 6.0, the section appears twice, once for Blazor Server and once for Blazor WebAssembly, each with the hosting model name in the section name. -->
* [Blazor Server threat mitigation: JavaScript functions invoked from .NET](xref:blazor/security/server/threat-mitigation#javascript-functions-invoked-from-net)

:::moniker-end

:::moniker range=">= aspnetcore-5.0 < aspnetcore-6.0"

For information on how to call .NET methods from JS, see <xref:blazor/js-interop/call-dotnet-from-javascript>.

To call into JS from .NET, inject the <xref:Microsoft.JSInterop.IJSRuntime> abstraction and call one of the following methods:

* <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A?displayProperty=nameWithType>
* <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeAsync%2A?displayProperty=nameWithType>
* <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType>

For the preceding .NET methods that invoke JS functions:

* The function identifier (`String`) is relative to the global scope (`window`). To call `window.someScope.someFunction`, the identifier is `someScope.someFunction`. There's no need to register the function before it's called.
* Pass any number of JSON-serializable arguments in `Object[]` to a JS function.
* The cancellation token (`CancellationToken`) propagates a notification that operations should be canceled.
* `TimeSpan` represents a time limit for a JS operation.
* The `TValue` return type must also be JSON serializable. `TValue` should match the .NET type that best maps to the JSON type returned.
* A [JS `Promise`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) is returned for `InvokeAsync` methods. `InvokeAsync` unwraps the [`Promise`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) and returns the value awaited by the [`Promise`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise).

For Blazor apps with prerendering enabled, calling into JS isn't possible during prerendering. For more information, see the [Prerendering](#prerendering) section.

The following example is based on [`TextDecoder`](https://developer.mozilla.org/docs/Web/API/TextDecoder), a JS-based decoder. The example demonstrates how to invoke a JS function from a C# method that offloads a requirement from developer code to an existing JS API. The JS function accepts a byte array from a C# method, decodes the array, and returns the text to the component for display.

Add the following JS code inside the closing `</body>` tag of `wwwroot/index.html` (Blazor WebAssembly) or `Pages/_Host.cshtml` (Blazor Server):

```html
<script>
  window.convertArray = (win1251Array) => {
    var win1251decoder = new TextDecoder('windows-1251');
    var bytes = new Uint8Array(win1251Array);
    var decodedArray = win1251decoder.decode(bytes);
    console.log(decodedArray);
    return decodedArray;
  };
</script>
```

The following `CallJsExample1` component:

* Invokes the `convertArray` JS function with <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeAsync%2A> when selecting a button (**`Convert Array`**).
* After the JS function is called, the passed array is converted into a string. The string is returned to the component for display (`text`).

`Pages/CallJsExample1.razor`:

:::code language="razor" source="~/../blazor-samples/5.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample1.razor" highlight="2,34":::

## JavaScript API restricted to user gestures

*This section only applies to Blazor Server apps.*

Some browser JavaScript (JS) APIs can only be executed in the context of a user gesture, such as using the [`Fullscreen API` (MDN documentation)](https://developer.mozilla.org/docs/Web/API/Fullscreen_API). These APIs can't be called through the JS interop mechanism in Blazor Server apps because UI event handling is performed asynchronously and generally no longer in the context of the user gesture. The app must handle the UI event completely in JavaScript, so use `onclick` instead of Blazor's `@onclick` directive attribute.

## Invoke JavaScript functions without reading a returned value (`InvokeVoidAsync`)

Use <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A> when:

* .NET isn't required to read the result of a JS call.
* JS functions return [void(0)/void 0](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/void) or [undefined](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/undefined).

Inside the closing `</body>` tag of `wwwroot/index.html` (Blazor WebAssembly) or `Pages/_Host.cshtml` (Blazor Server), provide a `displayTickerAlert1` JS function. The function is called with <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A> and doesn't return a value:

```html
<script>
  window.displayTickerAlert1 = (symbol, price) => {
    alert(`${symbol}: $${price}!`);
  };
</script>
```

### Component (`.razor`) example (`InvokeVoidAsync`)

`TickerChanged` calls the `handleTickerChanged1` method in the following `CallJsExample2` component.

`Pages/CallJsExample2.razor`:

:::code language="razor" source="~/../blazor-samples/5.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample2.razor" highlight="2,25":::

### Class (`.cs`) example (`InvokeVoidAsync`)

`JsInteropClasses1.cs`:

:::code language="csharp" source="~/../blazor-samples/5.0/BlazorSample_WebAssembly/JsInteropClasses1.cs" highlight="2,6,10,15":::

`TickerChanged` calls the `handleTickerChanged1` method in the following `CallJsExample3` component.

`Pages/CallJsExample3.razor`:

:::code language="razor" source="~/../blazor-samples/5.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample3.razor" highlight="2-3,20,24,32,35":::

## Invoke JavaScript functions and read a returned value (`InvokeAsync`)

Use <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeAsync%2A> when .NET should read the result of a JS call.

Inside the closing `</body>` tag of `wwwroot/index.html` (Blazor WebAssembly) or `Pages/_Host.cshtml` (Blazor Server), provide a `displayTickerAlert2` JS function. The following example returns a string for display by the caller:

```html
<script>
  window.displayTickerAlert2 = (symbol, price) => {
    if (price < 20) {
      alert(`${symbol}: $${price}!`);
      return "User alerted in the browser.";
    } else {
      return "User NOT alerted.";
    }
  };
</script>
```

### Component (`.razor`) example (`InvokeAsync`)

`TickerChanged` calls the `handleTickerChanged2` method and displays the returned string in the following `CallJsExample4` component.

`Pages/CallJsExample4.razor`:

:::code language="razor" source="~/../blazor-samples/5.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample4.razor" highlight="2,31-34":::

### Class (`.cs`) example (`InvokeAsync`)

`JsInteropClasses2.cs`:

:::code language="csharp" source="~/../blazor-samples/5.0/BlazorSample_WebAssembly/JsInteropClasses2.cs" highlight="2,6,10,15":::

`TickerChanged` calls the `handleTickerChanged2` method and displays the returned string in the following `CallJsExample5` component.

`Pages/CallJsExample5.razor`:

:::code language="razor" source="~/../blazor-samples/5.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample5.razor" highlight="2-3,25,30,38-40,43":::

## Dynamic content generation scenarios

For dynamic content generation with [BuildRenderTree](xref:blazor/advanced-scenarios#manually-build-a-render-tree-rendertreebuilder), use the `[Inject]` attribute:

```razor
[Inject]
IJSRuntime JS { get; set; }
```

## Prerendering

[!INCLUDE[](~/blazor/includes/prerendering.md)]

## Synchronous JS interop in Blazor WebAssembly apps

[!INCLUDE[](~/blazor/includes/js-interop/synchronous-js-interop-call-js.md)]

## Location of JavaScript

Load JavaScript (JS) code using any of approaches described by the [JavaScript (JS) interoperability (interop) overview article](xref:blazor/js-interop/index#location-of-javascript):

* [Load a script in `<head>` markup](xref:blazor/js-interop/index#load-a-script-in-head-markup) (*Not generally recommended*)
* [Load a script in `<body>` markup](xref:blazor/js-interop/index#load-a-script-in-body-markup)
* [Load a script from an external JavaScript file (`.js`)](xref:blazor/js-interop/index#load-a-script-from-an-external-javascript-file-js)
* [Inject a script after Blazor starts](xref:blazor/js-interop/index#inject-a-script-after-blazor-starts)

For information on isolating scripts in [JS modules](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules), see the [JavaScript isolation in JavaScript modules](#javascript-isolation-in-javascript-modules) section.

> [!WARNING]
> Don't place a `<script>` tag in a component file (`.razor`) because the `<script>` tag can't be updated dynamically.

## JavaScript isolation in JavaScript modules

Blazor enables JavaScript (JS) isolation in standard [JavaScript modules](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules) ([ECMAScript specification](https://tc39.es/ecma262/#sec-modules)). JavaScript module loading works the same way in Blazor as it does for other types of web apps, and you're free to customize how modules are defined in your app. For a guide on how to use JavaScript modules, see [MDN Web Docs: JavaScript modules](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules).

JS isolation provides the following benefits:

* Imported JS no longer pollutes the global namespace.
* Consumers of a library and components aren't required to import the related JS.

[Dynamic import with the `import()` operator](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/import) is supported with ASP.NET Core and Blazor:

```javascript
if ({CONDITION}) import("/additionalModule.js");
```

In the preceding example, the `{CONDITION}` placeholder represents a conditional check to determine if the module should be loaded.

For browser compatibility, see [Can I use: JavaScript modules: dynamic import](https://caniuse.com/es6-module-dynamic-import).

For example, the following JS module exports a JS function for showing a [browser window prompt](https://developer.mozilla.org/docs/Web/API/Window/prompt). Place the following JS code in an external JS file.

`wwwroot/scripts.js`:

```javascript
export function showPrompt(message) {
  return prompt(message, 'Type anything here');
}
```

Add the preceding JS module to an app or class library as a static web asset in the `wwwroot` folder and then import the module into the .NET code by calling <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> on the <xref:Microsoft.JSInterop.IJSRuntime> instance.

<xref:Microsoft.JSInterop.IJSRuntime> imports the module as an <xref:Microsoft.JSInterop.IJSObjectReference>, which represents a reference to a JS object from .NET code. Use the <xref:Microsoft.JSInterop.IJSObjectReference> to invoke exported JS functions from the module.

`Pages/CallJsExample6.razor`:

:::code language="razor" source="~/../blazor-samples/5.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample6.razor" highlight="2-3,16,23-24,35,38-44":::

In the preceding example:

* By convention, the `import` identifier is a special identifier used specifically for importing a JS module.
* Specify the module's external JS file using its stable static web asset path: `./{SCRIPT PATH AND FILE NAME (.js)}`, where:
  * The path segment for the current directory (`./`) is required in order to create the correct static asset path to the JS file.
  * The `{SCRIPT PATH AND FILE NAME (.js)}` placeholder is the path and file name under `wwwroot`.

Dynamically importing a module requires a network request, so it can only be achieved asynchronously by calling <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A>.

`IJSInProcessObjectReference` represents a reference to a JS object whose functions can be invoked synchronously in Blazor WebAssembly apps. For more information, see the [Synchronous JS interop in Blazor WebAssembly apps](#synchronous-js-interop-in-blazor-webassembly-apps) section.

> [!NOTE]
> When the external JS file is supplied by a [Razor class library](xref:blazor/components/class-libraries), specify the module's JS file using its stable static web asset path: `./_content/{PACKAGE ID}/{SCRIPT PATH AND FILE NAME (.js)}`:
>
> * The path segment for the current directory (`./`) is required in order to create the correct static asset path to the JS file.
> * The `{PACKAGE ID}` placeholder is the library's [package ID](/nuget/create-packages/creating-a-package-msbuild#set-properties). The package ID defaults to the project's assembly name if `<PackageId>` isn't specified in the project file. In the following example, the library's assembly name is `ComponentLibrary` and the library's project file doesn't specify `<PackageId>`.
> * The `{SCRIPT PATH AND FILE NAME (.js)}` placeholder is the path and file name under `wwwroot`. In the following example, the external JS file (`script.js`) is placed in the class library's `wwwroot` folder.
>
> ```csharp
> var module = await js.InvokeAsync<IJSObjectReference>(
>     "import", "./_content/ComponentLibrary/scripts.js");
> ```
>
> For more information, see <xref:blazor/components/class-libraries>.

Throughout the Blazor documentation, examples use the `.js` file extension for module files, not the [newer `.mjs` file extension (RFC 9239)](https://www.rfc-editor.org/rfc/rfc9239). Our documentation continues to use the `.js` file extension for the same reasons the Mozilla Foundation's documentation continues to use the `.js` file extension. For more information, see [Aside — .mjs versus .js (MDN documentation)](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules#aside_%E2%80%94_.mjs_versus_.js).
  
## Capture references to elements

Some JavaScript (JS) interop scenarios require references to HTML elements. For example, a UI library may require an element reference for initialization, or you might need to call command-like APIs on an element, such as `click` or `play`.

Capture references to HTML elements in a component using the following approach:

* Add an `@ref` attribute to the HTML element.
* Define a field of type <xref:Microsoft.AspNetCore.Components.ElementReference> whose name matches the value of the `@ref` attribute.

The following example shows capturing a reference to the `username` `<input>` element:

```razor
<input @ref="username" ... />

@code {
    private ElementReference username;
}
```

> [!WARNING]
> Only use an element reference to mutate the contents of an empty element that doesn't interact with Blazor. This scenario is useful when a third-party API supplies content to the element. Because Blazor doesn't interact with the element, there's no possibility of a conflict between Blazor's representation of the element and the Document Object Model (DOM).
>
> In the following example, it's *dangerous* to mutate the contents of the unordered list (`ul`) because Blazor interacts with the DOM to populate this element's list items (`<li>`) from the `Todos` object:
>
> ```razor
> <ul @ref="MyList">
>     @foreach (var item in Todos)
>     {
>         <li>@item.Text</li>
>     }
> </ul>
> ```
>
> Using the `MyList` element reference to merely read DOM content or trigger an event is supported.
>
> If JS interop *mutates the contents* of element `MyList` and Blazor attempts to apply diffs to the element, the diffs won't match the DOM. Modifying the contents of the list via JS interop with the `MyList` element reference is ***not supported***.
>
> For more information, see <xref:blazor/js-interop/index#interaction-with-the-document-object-model-dom>.

An <xref:Microsoft.AspNetCore.Components.ElementReference> is passed through to JS code via JS interop. The JS code receives an [`HTMLElement`](https://developer.mozilla.org/docs/Web/API/HTMLElement) instance, which it can use with normal DOM APIs. For example, the following code defines a .NET extension method (`TriggerClickEvent`) that enables sending a mouse click to an element.

The JS function `clickElement` creates a [`click`](https://developer.mozilla.org/docs/Web/API/Element/click_event) event on the passed HTML element (`element`):

```javascript
window.interopFunctions = {
  clickElement : function (element) {
    element.click();
  }
}
```

To call a JS function that doesn't return a value, use <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType>. The following code triggers a client-side [`click`](https://developer.mozilla.org/docs/Web/API/Element/click_event) event by calling the preceding JS function with the captured <xref:Microsoft.AspNetCore.Components.ElementReference>:

```razor
@inject IJSRuntime JS

<button @ref="exampleButton">Example Button</button>

<button @onclick="TriggerClick">
    Trigger click event on <code>Example Button</code>
</button>

@code {
    private ElementReference exampleButton;

    public async Task TriggerClick()
    {
        await JS.InvokeVoidAsync(
            "interopFunctions.clickElement", exampleButton);
    }
}
```

To use an extension method, create a static extension method that receives the <xref:Microsoft.JSInterop.IJSRuntime> instance:

```csharp
public static async Task TriggerClickEvent(this ElementReference elementRef, 
    IJSRuntime js)
{
    await js.InvokeVoidAsync("interopFunctions.clickElement", elementRef);
}
```

The `clickElement` method is called directly on the object. The following example assumes that the `TriggerClickEvent` method is available from the `JsInteropClasses` namespace:

```razor
@inject IJSRuntime JS
@using JsInteropClasses

<button @ref="exampleButton">Example Button</button>

<button @onclick="TriggerClick">
    Trigger click event on <code>Example Button</code>
</button>

@code {
    private ElementReference exampleButton;

    public async Task TriggerClick()
    {
        await exampleButton.TriggerClickEvent(JS);
    }
}
```

> [!IMPORTANT]
> The `exampleButton` variable is only populated after the component is rendered. If an unpopulated <xref:Microsoft.AspNetCore.Components.ElementReference> is passed to JS code, the JS code receives a value of `null`. To manipulate element references after the component has finished rendering, use the [`OnAfterRenderAsync` or `OnAfterRender` component lifecycle methods](xref:blazor/components/lifecycle#after-component-render-onafterrenderasync).

When working with generic types and returning a value, use <xref:System.Threading.Tasks.ValueTask%601>:

```csharp
public static ValueTask<T> GenericMethod<T>(this ElementReference elementRef, 
    IJSRuntime js)
{
    return js.InvokeAsync<T>("{JAVASCRIPT FUNCTION}", elementRef);
}
```

The `{JAVASCRIPT FUNCTION}` placeholder is the JS function identifier.

`GenericMethod` is called directly on the object with a type. The following example assumes that the `GenericMethod` is available from the `JsInteropClasses` namespace:

```razor
@inject IJSRuntime JS
@using JsInteropClasses

<input @ref="username" />

<button @onclick="OnClickMethod">Do something generic</button>

<p>
    returnValue: @returnValue
</p>

@code {
    private ElementReference username;
    private string returnValue;

    private async Task OnClickMethod()
    {
        returnValue = await username.GenericMethod<string>(JS);
    }
}
```

## Reference elements across components

An <xref:Microsoft.AspNetCore.Components.ElementReference> can't be passed between components because:

* The instance is only guaranteed to exist after the component is rendered, which is during or after a component's <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A>/<xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> method executes.
* An <xref:Microsoft.AspNetCore.Components.ElementReference> is a [`struct`](/dotnet/csharp/language-reference/builtin-types/struct), which can't be passed as a [component parameter](xref:blazor/components/index#component-parameters).

For a parent component to make an element reference available to other components, the parent component can:

* Allow child components to register callbacks.
* Invoke the registered callbacks during the <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> event with the passed element reference. Indirectly, this approach allows child components to interact with the parent's element reference.

Add the following style to the `<head>` of `wwwroot/index.html` (Blazor WebAssembly) or `Pages/_Host.cshtml` (Blazor Server):

```html
<style>
    .red { color: red }
</style>
```

Add the following script inside closing `</body>` tag of `wwwroot/index.html` (Blazor WebAssembly) or `Pages/_Host.cshtml` (Blazor Server):

```html
<script>
  function setElementClass(element, className) {
    var myElement = element;
    myElement.classList.add(className);
  }
</script>
```

`Pages/CallJsExample7.razor` (parent component):

:::code language="razor" source="~/../blazor-samples/5.0/BlazorSample_WebAssembly/Pages/CallJsExample7.razor" highlight="5,9":::

`Pages/CallJsExample7.razor.cs`:

:::code language="csharp" source="~/../blazor-samples/5.0/BlazorSample_WebAssembly/Pages/CallJsExample7.razor.cs":::

In the preceding example, the namespace of the app is `BlazorSample` with components in the `Pages` folder. If testing the code locally, update the namespace.

`Shared/SurveyPrompt.razor` (child component):

:::code language="razor" source="~/../blazor-samples/5.0/BlazorSample_WebAssembly/Shared/SurveyPrompt.razor":::

`Shared/SurveyPrompt.razor.cs`:

:::code language="csharp" source="~/../blazor-samples/5.0/BlazorSample_WebAssembly/Shared/SurveyPrompt.razor.cs":::

In the preceding example, the namespace of the app is `BlazorSample` with shared components in the `Shared` folder. If testing the code locally, update the namespace.

## Harden JavaScript interop calls

*This section primarily applies to Blazor Server apps, but Blazor WebAssembly apps may also set JS interop timeouts if conditions warrant it.*

In Blazor Server apps, JavaScript (JS) interop may fail due to networking errors and should be treated as unreliable. By default, Blazor Server apps use a one minute timeout for JS interop calls. If an app can tolerate a more aggressive timeout, set the timeout using one of the following approaches.

Set a global timeout in the `Startup.ConfigureServices` method of `Startup.cs` with <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.JSInteropDefaultCallTimeout?displayProperty=nameWithType>:

```csharp
services.AddServerSideBlazor(
    options => options.JSInteropDefaultCallTimeout = {TIMEOUT});
```

The `{TIMEOUT}` placeholder is a <xref:System.TimeSpan> (for example, `TimeSpan.FromSeconds(80)`).

Set a per-invocation timeout in component code. The specified timeout overrides the global timeout set by <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.JSInteropDefaultCallTimeout>:

```csharp
var result = await JS.InvokeAsync<string>("{ID}", {TIMEOUT}, new[] { "Arg1" });
```

In the preceding example:

* The `{TIMEOUT}` placeholder is a <xref:System.TimeSpan> (for example, `TimeSpan.FromSeconds(80)`).
* The `{ID}` placeholder is the identifier for the function to invoke. For example, the value `someScope.someFunction` invokes the function `window.someScope.someFunction`.

Although a common cause of JS interop failures are network failures in Blazor Server apps, per-invocation timeouts can be set for JS interop calls in Blazor WebAssembly apps. Although no SignalR circuit exists in a Blazor WebAssembly app, JS interop calls might fail for other reasons that apply in Blazor WebAssembly apps.

For more information on resource exhaustion, see <xref:blazor/security/server/threat-mitigation>.

## Avoid circular object references

Objects that contain circular references can't be serialized on the client for either:

* .NET method calls.
* JavaScript method calls from C# when the return type has circular references.

## JavaScript libraries that render UI

Sometimes you may wish to use JavaScript (JS) libraries that produce visible user interface elements within the browser Document Object Model (DOM). At first glance, this might seem difficult because Blazor's diffing system relies on having control over the tree of DOM elements and runs into errors if some external code mutates the DOM tree and invalidates its mechanism for applying diffs. This isn't a Blazor-specific limitation. The same challenge occurs with any diff-based UI framework.

Fortunately, it's straightforward to embed externally-generated UI within a Razor component UI reliably. The recommended technique is to have the component's code (`.razor` file) produce an empty element. As far as Blazor's diffing system is concerned, the element is always empty, so the renderer does not recurse into the element and instead leaves its contents alone. This makes it safe to populate the element with arbitrary externally-managed content.

The following example demonstrates the concept. Within the `if` statement when `firstRender` is `true`, interact with `unmanagedElement` outside of Blazor using JS interop. For example, call an external JS library to populate the element. Blazor leaves the element's contents alone until this component is removed. When the component is removed, the component's entire DOM subtree is also removed.

```razor
<h1>Hello! This is a Razor component rendered at @DateTime.Now</h1>

<div @ref="unmanagedElement"></div>

@code {
    private ElementReference unmanagedElement;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            ...
        }
    }
}
```

Consider the following example that renders an interactive map using [open-source Mapbox APIs](https://www.mapbox.com/).

The following JS module is placed into the app or made available from a Razor class library.

> [!NOTE]
> To create the [Mapbox](https://www.mapbox.com/) map, obtain an access token from [Mapbox Sign in](https://account.mapbox.com) and provide it where the `{ACCESS TOKEN}` appears in the following code.

`wwwroot/mapComponent.js`:

```javascript
import 'https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.js';

mapboxgl.accessToken = '{ACCESS TOKEN}';

export function addMapToElement(element) {
  return new mapboxgl.Map({
    container: element,
    style: 'mapbox://styles/mapbox/streets-v11',
    center: [-74.5, 40],
    zoom: 9
  });
}

export function setMapCenter(map, latitude, longitude) {
  map.setCenter([longitude, latitude]);
}
```

To produce correct styling, add the following stylesheet tag to the host HTML page.

In `wwwroot/index.html` (Blazor WebAssembly) or `Pages/_Host.cshtml` (Blazor Server), add the following `<link>` element to the `<head>` element markup:

```html
<link href="https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.css" 
    rel="stylesheet" />
```

`Pages/CallJsExample8.razor`:

:::code language="razor" source="~/../blazor-samples/5.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample8.razor":::

The preceding example produces an interactive map UI. The user:

* Can drag to scroll or zoom.
* Select buttons to jump to predefined locations.

![Mapbox street map of Tokyo, Japan with buttons to select Bristol, United Kingdom and Tokyo, Japan](~/blazor/javascript-interoperability/call-javascript-from-dotnet/_static/mapbox-example.png)

In the preceding example:

* The `<div>` with `@ref="mapElement"` is left empty as far as Blazor is concerned. The `mapbox-gl.js` script can safely populate the element and modify its contents. Use this technique with any JS library that renders UI. You can embed components from a third-party JS SPA framework inside Razor components, as long as they don't try to reach out and modify other parts of the page. It is **not** safe for external JS code to modify elements that Blazor does not regard as empty.
* When using this approach, bear in mind the rules about how Blazor retains or destroys DOM elements. The component safely handles button click events and updates the existing map instance because DOM elements are retained where possible by default. If you were rendering a list of map elements from inside a `@foreach` loop, you want to use `@key` to ensure the preservation of component instances. Otherwise, changes in the list data could cause component instances to retain the state of previous instances in an undesirable manner. For more information, see [using @key to preserve elements and components](xref:blazor/components/index#use-key-to-control-the-preservation-of-elements-and-components).
* The example encapsulates JS logic and dependencies within an ES6 module and loads the module dynamically using the `import` identifier. For more information, see [JavaScript isolation in JavaScript modules](#javascript-isolation-in-javascript-modules).

## Size limits on JavaScript interop calls

[!INCLUDE[](~/blazor/includes/js-interop/5.0/size-limits.md)]

## Unmarshalled JavaScript interop

Blazor WebAssembly components may experience poor performance when .NET objects are serialized for JavaScript (JS) interop and either of the following are true:

* A high volume of .NET objects are rapidly serialized. For example, poor performance may result when JS interop calls are made on the basis of moving an input device, such as spinning a mouse wheel.
* Large .NET objects or many .NET objects must be serialized for JS interop. For example, poor performance may result when JS interop calls require serializing dozens of files.

<xref:Microsoft.JSInterop.IJSUnmarshalledObjectReference> represents a reference to an JS object whose functions can be invoked without the overhead of serializing .NET data.

In the following example:

* A [struct](/dotnet/csharp/language-reference/builtin-types/struct) containing a string and an integer is passed unserialized to JS.
* JS functions process the data and return either a boolean or string to the caller.
* A JS string isn't directly convertible into a .NET `string` object. The `unmarshalledFunctionReturnString` function calls `BINDING.js_string_to_mono_string` to manage the conversion of a JS string.

> [!NOTE]
> The following examples aren't typical use cases for this scenario because the [struct](/dotnet/csharp/language-reference/builtin-types/struct) passed to JS doesn't result in poor component performance. The example uses a small object merely to demonstrate the concepts for passing unserialized .NET data.

Place the following `<script>` block in `wwwroot/index.html` (Blazor WebAssembly) or `Pages/_Host.cshtml` (Blazor Server). Alternatively, you can place the JS in an external JS file referenced inside the closing `</body>` tag with `<script src="{SCRIPT PATH AND FILE NAME (.js)}></script>`, where the `{SCRIPT PATH AND FILE NAME (.js)}` placeholder is the path and file name of the script.

```javascript
<script>
  window.returnObjectReference = () => {
    return {
      unmarshalledFunctionReturnBoolean: function (fields) {
        const name = Blazor.platform.readStringField(fields, 0);
        const year = Blazor.platform.readInt32Field(fields, 8);
    
        return name === "Brigadier Alistair Gordon Lethbridge-Stewart" &&
            year === 1968;
      },
      unmarshalledFunctionReturnString: function (fields) {
        const name = Blazor.platform.readStringField(fields, 0);
        const year = Blazor.platform.readInt32Field(fields, 8);

        return BINDING.js_string_to_mono_string(`Hello, ${name} (${year})!`);
      }
    };
  }
</script>
```

> [!WARNING]
> The `js_string_to_mono_string` function name, behavior, and existence is subject to change in a future release of .NET. For example:
>
> * The function is likely to be renamed.
> * The function itself might be removed in favor of automatic conversion of strings by the framework.

`Pages/CallJsExample10.razor`:

:::code language="razor" source="~/../blazor-samples/5.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample10.razor":::

If an <xref:Microsoft.JSInterop.IJSUnmarshalledObjectReference> instance isn't disposed in C# code, it can be disposed in JS. The following `dispose` function disposes the object reference when called from JS:

```javascript
window.exampleJSObjectReferenceNotDisposedInCSharp = () => {
  return {
    dispose: function () {
      DotNet.disposeJSObjectReference(this);
    },

    ...
  };
}
```

Array types can be converted from JS objects into .NET objects using `js_typed_array_to_array`, but the JS array must be a typed array. Arrays from JS can be read in C# code as a .NET object array (`object[]`).

Other data types, such as string arrays, can be converted but require creating a new Mono array object (`mono_obj_array_new`) and setting its value (`mono_obj_array_set`).

> [!WARNING]
> JS functions provided by the Blazor framework, such as `js_typed_array_to_array`, `mono_obj_array_new`, and `mono_obj_array_set`, are subject to name changes, behavioral changes, or removal in future releases of .NET.

## Catch JavaScript exceptions

To catch JS exceptions, wrap the JS interop in a [`try`-`catch` block](/dotnet/csharp/fundamentals/exceptions/exception-handling) and catch a <xref:Microsoft.JSInterop.JSException>.

In the following example, the `nonFunction` JS function doesn't exist. When the function isn't found, the <xref:Microsoft.JSInterop.JSException> is trapped with a <xref:System.Exception.Message> that indicates the following error:

> `Could not find 'nonFunction' ('nonFunction' was undefined).`

`Pages/CallJsExample11.razor`:

:::code language="csharp" source="~/../blazor-samples/5.0/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample11.razor" highlight="28":::

## Document Object Model (DOM) cleanup tasks during component disposal

Don't execute JS interop code for DOM cleanup tasks during component disposal. Instead, use the [`MutationObserver`](https://developer.mozilla.org/docs/Web/API/MutationObserver) pattern in JavaScript on the client for the following reasons:

* The component may have been removed from the DOM by the time your cleanup code executes in `Dispose{Async}`.
* In a Blazor Server app, the Blazor renderer may have been disposed by the framework by the time your cleanup code executes in `Dispose{Async}`.

The [`MutationObserver`](https://developer.mozilla.org/docs/Web/API/MutationObserver) pattern allows you to run a function when an element is removed from the DOM.

## JavaScript interop calls without a circuit

[!INCLUDE[](~/blazor/includes/js-interop/circuit-disconnection.md)]

## Additional resources

* <xref:blazor/js-interop/call-dotnet-from-javascript>
* [`InteropComponent.razor` example (dotnet/AspNetCore GitHub repository `main` branch)](https://github.com/dotnet/AspNetCore/blob/main/src/Components/test/testassets/BasicTestApp/InteropComponent.razor): The `main` branch represents the product unit's current development for the next release of ASP.NET Core. To select the branch for a different release (for example, `release/5.0`), use the **Switch branches or tags** dropdown list to select the branch.
* [Blazor samples GitHub repository (`dotnet/blazor-samples`)](https://github.com/dotnet/blazor-samples)
* <xref:blazor/fundamentals/handle-errors> (*JavaScript interop* section) <!-- AUTHOR NOTE: The JavaScript interop section isn't linked because the section title changed across versions of the doc. Prior to 6.0, the section appears twice, once for Blazor Server and once for Blazor WebAssembly, each with the hosting model name in the section name. -->
* [Blazor Server threat mitigation: JavaScript functions invoked from .NET](xref:blazor/security/server/threat-mitigation#javascript-functions-invoked-from-net)

:::moniker-end

:::moniker range="< aspnetcore-5.0"

For information on how to call .NET methods from JS, see <xref:blazor/js-interop/call-dotnet-from-javascript>.

To call into JS from .NET, inject the <xref:Microsoft.JSInterop.IJSRuntime> abstraction and call one of the following methods:

* <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A?displayProperty=nameWithType>
* <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeAsync%2A?displayProperty=nameWithType>
* <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType>

For the preceding .NET methods that invoke JS functions:

* The function identifier (`String`) is relative to the global scope (`window`). To call `window.someScope.someFunction`, the identifier is `someScope.someFunction`. There's no need to register the function before it's called.
* Pass any number of JSON-serializable arguments in `Object[]` to a JS function.
* The cancellation token (`CancellationToken`) propagates a notification that operations should be canceled.
* `TimeSpan` represents a time limit for a JS operation.
* The `TValue` return type must also be JSON serializable. `TValue` should match the .NET type that best maps to the JSON type returned.
* A [JS `Promise`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) is returned for `InvokeAsync` methods. `InvokeAsync` unwraps the [`Promise`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) and returns the value awaited by the [`Promise`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise).

For Blazor apps with prerendering enabled, calling into JS isn't possible during prerendering. For more information, see the [Prerendering](#prerendering) section.

The following example is based on [`TextDecoder`](https://developer.mozilla.org/docs/Web/API/TextDecoder), a JS-based decoder. The example demonstrates how to invoke a JS function from a C# method that offloads a requirement from developer code to an existing JS API. The JS function accepts a byte array from a C# method, decodes the array, and returns the text to the component for display.

Add the following JS code inside the closing `</body>` tag of `wwwroot/index.html` (Blazor WebAssembly) or `Pages/_Host.cshtml` (Blazor Server):

```html
<script>
  window.convertArray = (win1251Array) => {
    var win1251decoder = new TextDecoder('windows-1251');
    var bytes = new Uint8Array(win1251Array);
    var decodedArray = win1251decoder.decode(bytes);
    console.log(decodedArray);
    return decodedArray;
  };
</script>
```

The following `CallJsExample1` component:

* Invokes the `convertArray` JS function with <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeAsync%2A> when selecting a button (**`Convert Array`**).
* After the JS function is called, the passed array is converted into a string. The string is returned to the component for display (`text`).

`Pages/CallJsExample1.razor`:

:::code language="razor" source="~/../blazor-samples/3.1/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample1.razor" highlight="2,34-35":::

## JavaScript API restricted to user gestures

*This section only applies to Blazor Server apps.*

Some browser JavaScript (JS) APIs can only be executed in the context of a user gesture, such as using the [`Fullscreen API` (MDN documentation)](https://developer.mozilla.org/docs/Web/API/Fullscreen_API). These APIs can't be called through the JS interop mechanism in Blazor Server apps because UI event handling is performed asynchronously and generally no longer in the context of the user gesture. The app must handle the UI event completely in JavaScript, so use `onclick` instead of Blazor's `@onclick` directive attribute.

## Invoke JavaScript functions without reading a returned value (`InvokeVoidAsync`)

Use <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A> when:

* .NET isn't required to read the result of a JS call.
* JS functions return [void(0)/void 0](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/void) or [undefined](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/undefined).

Inside the closing `</body>` tag of `wwwroot/index.html` (Blazor WebAssembly) or `Pages/_Host.cshtml` (Blazor Server), provide a `displayTickerAlert1` JS function. The function is called with <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A> and doesn't return a value:

```html
<script>
  window.displayTickerAlert1 = (symbol, price) => {
    alert(`${symbol}: $${price}!`);
  };
</script>
```

### Component (`.razor`) example (`InvokeVoidAsync`)

`TickerChanged` calls the `handleTickerChanged1` method in the following `CallJsExample2` component.

`Pages/CallJsExample2.razor`:

:::code language="razor" source="~/../blazor-samples/3.1/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample2.razor" highlight="2,25":::

### Class (`.cs`) example (`InvokeVoidAsync`)

`JsInteropClasses1.cs`:

:::code language="csharp" source="~/../blazor-samples/3.1/BlazorSample_WebAssembly/JsInteropClasses1.cs" highlight="2,6,10,15":::

`TickerChanged` calls the `handleTickerChanged1` method in the following `CallJsExample3` component.

`Pages/CallJsExample3.razor`:

:::code language="razor" source="~/../blazor-samples/3.1/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample3.razor" highlight="2-3,20,24,32,35":::

## Invoke JavaScript functions and read a returned value (`InvokeAsync`)

Use <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeAsync%2A> when .NET should read the result of a JS call.

Inside the closing `</body>` tag of `wwwroot/index.html` (Blazor WebAssembly) or `Pages/_Host.cshtml` (Blazor Server), provide a `displayTickerAlert2` JS function. The following example returns a string for display by the caller:

```html
<script>
  window.displayTickerAlert2 = (symbol, price) => {
    if (price < 20) {
      alert(`${symbol}: $${price}!`);
      return "User alerted in the browser.";
    } else {
      return "User NOT alerted.";
    }
  };
</script>
```

### Component (`.razor`) example (`InvokeAsync`)

`TickerChanged` calls the `handleTickerChanged2` method and displays the returned string in the following `CallJsExample4` component.

`Pages/CallJsExample4.razor`:

:::code language="razor" source="~/../blazor-samples/3.1/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample4.razor" highlight="2,31-34":::

### Class (`.cs`) example (`InvokeAsync`)

`JsInteropClasses2.cs`:

:::code language="csharp" source="~/../blazor-samples/3.1/BlazorSample_WebAssembly/JsInteropClasses2.cs" highlight="2,6,10,15":::

`TickerChanged` calls the `handleTickerChanged2` method and displays the returned string in the following `CallJsExample5` component.

`Pages/CallJsExample5.razor`:

:::code language="razor" source="~/../blazor-samples/3.1/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample5.razor" highlight="2-3,25,30,38-40,43":::

## Dynamic content generation scenarios

For dynamic content generation with [BuildRenderTree](xref:blazor/advanced-scenarios#manually-build-a-render-tree-rendertreebuilder), use the `[Inject]` attribute:

```razor
[Inject]
IJSRuntime JS { get; set; }
```

## Prerendering

[!INCLUDE[](~/blazor/includes/prerendering.md)]

## Synchronous JS interop in Blazor WebAssembly apps

[!INCLUDE[](~/blazor/includes/js-interop/synchronous-js-interop-call-js.md)]

## Location of JavaScript

Load JavaScript (JS) code using any of approaches described by the [JavaScript (JS) interoperability (interop) overview article](xref:blazor/js-interop/index#location-of-javascript):

* [Load a script in `<head>` markup](xref:blazor/js-interop/index#load-a-script-in-head-markup) (*Not generally recommended*)
* [Load a script in `<body>` markup](xref:blazor/js-interop/index#load-a-script-in-body-markup)
* [Load a script from an external JavaScript file (`.js`)](xref:blazor/js-interop/index#load-a-script-from-an-external-javascript-file-js)
* [Inject a script after Blazor starts](xref:blazor/js-interop/index#inject-a-script-after-blazor-starts)

> [!WARNING]
> Don't place a `<script>` tag in a component file (`.razor`) because the `<script>` tag can't be updated dynamically.

Throughout the Blazor documentation, examples use the `.js` file extension for module files, not the [newer `.mjs` file extension (RFC 9239)](https://www.rfc-editor.org/rfc/rfc9239). Our documentation continues to use the `.js` file extension for the same reasons the Mozilla Foundation's documentation continues to use the `.js` file extension. For more information, see [Aside — .mjs versus .js (MDN documentation)](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules#aside_%E2%80%94_.mjs_versus_.js).

## Capture references to elements

Some JavaScript (JS) interop scenarios require references to HTML elements. For example, a UI library may require an element reference for initialization, or you might need to call command-like APIs on an element, such as `click` or `play`.

Capture references to HTML elements in a component using the following approach:

* Add an `@ref` attribute to the HTML element.
* Define a field of type <xref:Microsoft.AspNetCore.Components.ElementReference> whose name matches the value of the `@ref` attribute.

The following example shows capturing a reference to the `username` `<input>` element:

```razor
<input @ref="username" ... />

@code {
    private ElementReference username;
}
```

> [!WARNING]
> Only use an element reference to mutate the contents of an empty element that doesn't interact with Blazor. This scenario is useful when a third-party API supplies content to the element. Because Blazor doesn't interact with the element, there's no possibility of a conflict between Blazor's representation of the element and the Document Object Model (DOM).
>
> In the following example, it's *dangerous* to mutate the contents of the unordered list (`ul`) because Blazor interacts with the DOM to populate this element's list items (`<li>`) from the `Todos` object:
>
> ```razor
> <ul @ref="MyList">
>     @foreach (var item in Todos)
>     {
>         <li>@item.Text</li>
>     }
> </ul>
> ```
>
> Using the `MyList` element reference to merely read DOM content or trigger an event is supported.
>
> If JS interop *mutates the contents* of element `MyList` and Blazor attempts to apply diffs to the element, the diffs won't match the DOM. Modifying the contents of the list via JS interop with the `MyList` element reference is ***not supported***.
>
> For more information, see <xref:blazor/js-interop/index#interaction-with-the-document-object-model-dom>.

An <xref:Microsoft.AspNetCore.Components.ElementReference> is passed through to JS code via JS interop. The JS code receives an [`HTMLElement`](https://developer.mozilla.org/docs/Web/API/HTMLElement) instance, which it can use with normal DOM APIs. For example, the following code defines a .NET extension method (`TriggerClickEvent`) that enables sending a mouse click to an element.

The JS function `clickElement` creates a [`click`](https://developer.mozilla.org/docs/Web/API/Element/click_event) event on the passed HTML element (`element`):

```javascript
window.interopFunctions = {
  clickElement : function (element) {
    element.click();
  }
}
```

To call a JS function that doesn't return a value, use <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType>. The following code triggers a client-side [`click`](https://developer.mozilla.org/docs/Web/API/Element/click_event) event by calling the preceding JS function with the captured <xref:Microsoft.AspNetCore.Components.ElementReference>:

```razor
@inject IJSRuntime JS

<button @ref="exampleButton">Example Button</button>

<button @onclick="TriggerClick">
    Trigger click event on <code>Example Button</code>
</button>

@code {
    private ElementReference exampleButton;

    public async Task TriggerClick()
    {
        await JS.InvokeVoidAsync(
            "interopFunctions.clickElement", exampleButton);
    }
}
```

To use an extension method, create a static extension method that receives the <xref:Microsoft.JSInterop.IJSRuntime> instance:

```csharp
public static async Task TriggerClickEvent(this ElementReference elementRef, 
    IJSRuntime js)
{
    await js.InvokeVoidAsync("interopFunctions.clickElement", elementRef);
}
```

The `clickElement` method is called directly on the object. The following example assumes that the `TriggerClickEvent` method is available from the `JsInteropClasses` namespace:

```razor
@inject IJSRuntime JS
@using JsInteropClasses

<button @ref="exampleButton">Example Button</button>

<button @onclick="TriggerClick">
    Trigger click event on <code>Example Button</code>
</button>

@code {
    private ElementReference exampleButton;

    public async Task TriggerClick()
    {
        await exampleButton.TriggerClickEvent(JS);
    }
}
```

> [!IMPORTANT]
> The `exampleButton` variable is only populated after the component is rendered. If an unpopulated <xref:Microsoft.AspNetCore.Components.ElementReference> is passed to JS code, the JS code receives a value of `null`. To manipulate element references after the component has finished rendering, use the [`OnAfterRenderAsync` or `OnAfterRender` component lifecycle methods](xref:blazor/components/lifecycle#after-component-render-onafterrenderasync).

When working with generic types and returning a value, use <xref:System.Threading.Tasks.ValueTask%601>:

```csharp
public static ValueTask<T> GenericMethod<T>(this ElementReference elementRef, 
    IJSRuntime js)
{
    return js.InvokeAsync<T>("{JAVASCRIPT FUNCTION}", elementRef);
}
```

The `{JAVASCRIPT FUNCTION}` placeholder is the JS function identifier.

`GenericMethod` is called directly on the object with a type. The following example assumes that the `GenericMethod` is available from the `JsInteropClasses` namespace:

```razor
@inject IJSRuntime JS
@using JsInteropClasses

<input @ref="username" />

<button @onclick="OnClickMethod">Do something generic</button>

<p>
    returnValue: @returnValue
</p>

@code {
    private ElementReference username;
    private string returnValue;

    private async Task OnClickMethod()
    {
        returnValue = await username.GenericMethod<string>(JS);
    }
}
```

## Reference elements across components

An <xref:Microsoft.AspNetCore.Components.ElementReference> can't be passed between components because:

* The instance is only guaranteed to exist after the component is rendered, which is during or after a component's <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A>/<xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> method executes.
* An <xref:Microsoft.AspNetCore.Components.ElementReference> is a [`struct`](/dotnet/csharp/language-reference/builtin-types/struct), which can't be passed as a [component parameter](xref:blazor/components/index#component-parameters).

For a parent component to make an element reference available to other components, the parent component can:

* Allow child components to register callbacks.
* Invoke the registered callbacks during the <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> event with the passed element reference. Indirectly, this approach allows child components to interact with the parent's element reference.

Add the following style to the `<head>` of `wwwroot/index.html` (Blazor WebAssembly) or `Pages/_Host.cshtml` (Blazor Server):

```html
<style>
    .red { color: red }
</style>
```

Add the following script inside closing `</body>` tag of `wwwroot/index.html` (Blazor WebAssembly) or `Pages/_Host.cshtml` (Blazor Server):

```html
<script>
  function setElementClass(element, className) {
    var myElement = element;
    myElement.classList.add(className);
  }
</script>
```

`Pages/CallJsExample7.razor` (parent component):

:::code language="razor" source="~/../blazor-samples/3.1/BlazorSample_WebAssembly/Pages/CallJsExample7.razor" highlight="5,9":::

`Pages/CallJsExample7.razor.cs`:

:::code language="csharp" source="~/../blazor-samples/3.1/BlazorSample_WebAssembly/Pages/CallJsExample7.razor.cs":::

In the preceding example, the namespace of the app is `BlazorSample` with components in the `Pages` folder. If testing the code locally, update the namespace.

`Shared/SurveyPrompt.razor` (child component):

:::code language="razor" source="~/../blazor-samples/3.1/BlazorSample_WebAssembly/Shared/SurveyPrompt.razor":::

`Shared/SurveyPrompt.razor.cs`:

:::code language="csharp" source="~/../blazor-samples/3.1/BlazorSample_WebAssembly/Shared/SurveyPrompt.razor.cs":::

In the preceding example, the namespace of the app is `BlazorSample` with shared components in the `Shared` folder. If testing the code locally, update the namespace.

## Harden JavaScript interop calls

*This section primarily applies to Blazor Server apps, but Blazor WebAssembly apps may also set JS interop timeouts if conditions warrant it.*

In Blazor Server apps, JavaScript (JS) interop may fail due to networking errors and should be treated as unreliable. By default, Blazor Server apps use a one minute timeout for JS interop calls. If an app can tolerate a more aggressive timeout, set the timeout using one of the following approaches.

Set a global timeout in the `Startup.ConfigureServices` method of `Startup.cs` with <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.JSInteropDefaultCallTimeout?displayProperty=nameWithType>:

```csharp
services.AddServerSideBlazor(
    options => options.JSInteropDefaultCallTimeout = {TIMEOUT});
```

The `{TIMEOUT}` placeholder is a <xref:System.TimeSpan> (for example, `TimeSpan.FromSeconds(80)`).

Set a per-invocation timeout in component code. The specified timeout overrides the global timeout set by <xref:Microsoft.AspNetCore.Components.Server.CircuitOptions.JSInteropDefaultCallTimeout>:

```csharp
var result = await JS.InvokeAsync<string>("{ID}", {TIMEOUT}, new[] { "Arg1" });
```

In the preceding example:

* The `{TIMEOUT}` placeholder is a <xref:System.TimeSpan> (for example, `TimeSpan.FromSeconds(80)`).
* The `{ID}` placeholder is the identifier for the function to invoke. For example, the value `someScope.someFunction` invokes the function `window.someScope.someFunction`.

Although a common cause of JS interop failures are network failures in Blazor Server apps, per-invocation timeouts can be set for JS interop calls in Blazor WebAssembly apps. Although no SignalR circuit exists in a Blazor WebAssembly app, JS interop calls might fail for other reasons that apply in Blazor WebAssembly apps.

For more information on resource exhaustion, see <xref:blazor/security/server/threat-mitigation>.

## Avoid circular object references

Objects that contain circular references can't be serialized on the client for either:

* .NET method calls.
* JavaScript method calls from C# when the return type has circular references.

## Size limits on JavaScript interop calls

[!INCLUDE[](~/blazor/includes/js-interop/3.1/size-limits.md)]

## Catch JavaScript exceptions

To catch JS exceptions, wrap the JS interop in a [`try`-`catch` block](/dotnet/csharp/fundamentals/exceptions/exception-handling) and catch a <xref:Microsoft.JSInterop.JSException>.

In the following example, the `nonFunction` JS function doesn't exist. When the function isn't found, the <xref:Microsoft.JSInterop.JSException> is trapped with a <xref:System.Exception.Message> that indicates the following error:

> `Could not find 'nonFunction' ('nonFunction' was undefined).`

`Pages/CallJsExample11.razor`:

:::code language="csharp" source="~/../blazor-samples/3.1/BlazorSample_WebAssembly/Pages/call-js-from-dotnet/CallJsExample11.razor" highlight="28":::

## Document Object Model (DOM) cleanup tasks during component disposal

Don't execute JS interop code for DOM cleanup tasks during component disposal. Instead, use the [`MutationObserver`](https://developer.mozilla.org/docs/Web/API/MutationObserver) pattern in JavaScript on the client for the following reasons:

* The component may have been removed from the DOM by the time your cleanup code executes in `Dispose{Async}`.
* In a Blazor Server app, the Blazor renderer may have been disposed by the framework by the time your cleanup code executes in `Dispose{Async}`.

The [`MutationObserver`](https://developer.mozilla.org/docs/Web/API/MutationObserver) pattern allows you to run a function when an element is removed from the DOM.

## JavaScript interop calls without a circuit

[!INCLUDE[](~/blazor/includes/js-interop/circuit-disconnection.md)]

## Additional resources

* <xref:blazor/js-interop/call-dotnet-from-javascript>
* [`InteropComponent.razor` example (dotnet/AspNetCore GitHub repository `main` branch)](https://github.com/dotnet/AspNetCore/blob/main/src/Components/test/testassets/BasicTestApp/InteropComponent.razor): The `main` branch represents the product unit's current development for the next release of ASP.NET Core. To select the branch for a different release (for example, `release/5.0`), use the **Switch branches or tags** dropdown list to select the branch.
* <xref:blazor/file-uploads>
* [Blazor samples GitHub repository (`dotnet/blazor-samples`)](https://github.com/dotnet/blazor-samples)
* <xref:blazor/fundamentals/handle-errors> (*JavaScript interop* section) <!-- AUTHOR NOTE: The JavaScript interop section isn't linked because the section title changed across versions of the doc. Prior to 6.0, the section appears twice, once for Blazor Server and once for Blazor WebAssembly, each with the hosting model name in the section name. -->
* [Blazor Server threat mitigation: JavaScript functions invoked from .NET](xref:blazor/security/server/threat-mitigation#javascript-functions-invoked-from-net)

:::moniker-end
