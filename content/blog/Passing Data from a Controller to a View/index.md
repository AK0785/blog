---
title: User Notifications
date: "2021-04-11"
description: TempData can be very useful in particular circumstances, one does not always need a ViewModel...
---
## TempData is a great option

TempData is a storage container for data that needs to be available to a separate HTTP request.

Most often with a POST the controller will contain a <code>RedirectToAction("Index")</code> that will navigate the user to the appropriate controller action - in this example redirecting to the <code>Index</code> Action of the <code>Create</code> Controller.

I needed a means of confirming to the user that their <code>Create</code> request had been successful. I did not want to modify the model to include a:

```cs
public bool formResult {get; set;}
```

and i did not want to create a dedicated ViewModel

```cs
public class viewModel {
    public bool formResult {get; set;}
}

return View(viewModel);
```

just to hold the result of the action e.g. successful create. This is where TempData can be very, very useful for situations such as this where the use case is ***notifying the user***.

```cs
TempData["success"] = $"...passing various properties... created!";
```
and then in the view:
```cs
@if (TempData["success"] != null)
{
    <div class="alert alert-success">
    @TempData["success"]
    </div>
}
```
Once the value has been ***read*** it is ***automatically marked for deletion and is no longer available on subsequent requests***, this makes it parcticularly useful for passing success notifications to the user.