---
title: ValidateAntiForgeryToken
date: "2021-04-07"
description: Notes on testing with ValidateAntiForgeryToken using Postman.
---
## Posting via Postman with [ValidateAntiForgeryToken] Decorators within ASP.NET MVC Core

Working on side projects is a great way to ensure that your skills remain up to date. It is also a great opportunity to use some of the newest versions of C#, .NET Core and Entity Framework - especially when you are developing a heavy back-end solution!

Over the Easter Weekend, I committed to getting the core of my side project in place - EF Code First, Controllers, Views and some Form Submissions!

```cs
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Create()
{
    // ...
    return View();
}
```

Then, an interesting challenge. Testing a controller action that is decorated with `[ValidateAntiForgeryToken]`. Disabling the decorator is one option, but <strong>not recommended</strong>. After all, it will be running in production, so best to check it is protecting against XSRF!

### How to get the token

Within a form, `__RequestVerificationToken` is injected into a hidden field and used server side in the anti-forgery validation.

```js
pm.sendRequest("https://....", function (err, res) {
    if (err)
    {
        return console.error(err);
    }  else {
        var body = res.text();
        var html = cheerio(body);
        var token = html.find('input[name="__RequestVerificationToken"]');
        var input = token.first();
        var token = input.val();
        pm.environment.set('xsrf-token', token);    
    }    
});
```

Using the above JavaScript, Postman can execute a pre-request script:
1. Issue a request to the form page
2. Parse the HTML response
3. Find a hidden input named `___RequestVerificationToken`
4. Retrieve the **first** token it finds (assuming it might find > 1)
5. Sets an environment variable to match the retrieved token

### Posting with Postman

Now that we have the `___RequestVerificationToken` we can use it in our POST requests and therefore *pass* the anti-forger validator.

### Do I really need to decorate every, single, POST, request?

No and a far better way is to set `[AutoValidateAntiForgeryToken]` as a global filter within the `ConfigureServices` method of the `Startup` class. Safe requests (GET/HEAD) are automatically ignored, Unsafe requests (POST/PUT/PATCH/DELETE) are required to validate.

```cs
services.AddMvc(options => 
options.Filters.Add(new AutoValidateAntiforgeryTokenAttribute())
);
```
> The key benefit with following this pattern is that the XSRF protection is automatically applied. If you forget to decorate a controller method and you're not configured to use `[AutoValidateAntiForgeryToken]` then your web app is at risk. And, you get to write a little less code in the process!

#### Attribution(s) & Recommended Reading

- Marius Schulz (https://mariusschulz.com/blog/global-antiforgery-token-validation-in-asp-net-core)
- Andrew Lock (https://andrewlock.net/automatically-validating-anti-forgery-tokens-in-asp-net-core-with-the-autovalidateantiforgerytokenattribute/)
- Ankur Sheel (https://www.ankursheel.com/blog/postman-ajax-endpoints-xsrf-tokens)
- Wikipedia (https://en.wikipedia.org/wiki/Cross-site_request_forgery)
- MS Docs (https://docs.microsoft.com/en-us/aspnet/core/security/anti-request-forgery?view=aspnetcore-5.0)