---
layout: post
title: Angular error interceptors
---

After introducing [response macros for Laravel APIs](/2016/03/27/laravel-response-macros-api/){:target="_blank"}, it's time to keep our angular code DRY.

Most modern web apps consume APIs frequently. Knowing that, we need to avoid writing the same code on every single API call.

## Restangular

We will use `Restangular` to perform our API calls. [Restangular](https://github.com/mgonto/restangular) is a wrapper on top of `$http`. You can write your own error interceptors using `$http`.

### Setup

Below is the setup for an API service based on Restangular, written in ES6.

```javascript
export class APIService {
  constructor(Restangular, ToastService) {
    'ngInject';

    var headers = {
      'Content-Type': 'application/json',
      'Accept': 'application/x.laravel.v1+json'
    };

    return Restangular.withConfig(function(RestangularConfigurer) {

      RestangularConfigurer
        .setBaseUrl('/api/')
        .setDefaultHeaders(headers)
        .setErrorInterceptor(function(response) {

          if (response.status === 422) {
            for (var error in response.data.errors) {
              var error_message = response.data.errors[error][0];
              return ToastService.error(error_message);
            }
          }

        });
    });
  }
}
```
As you can see, we start by setting the **baseUrl** to avoid writing `/api/` on every single request.  

Then we're setting some default headers. We're sending an accept header as part of the content negotiation process. But that's outside the scope of this article. (Let me know in the comments if you'd like to have an article on content negotiation using dingo/api)

Then we're setting an error interceptor. Which is a function that gets called **every-time** the API call encounters an **error**.  
We can then extract the *response status* and perform specific actions based on it.   
We know for sure that Laravel returns a response status of **422** whenever the validation fails.
That's how we're able to show the first error message that we get from the API.  
By the way, `ToastService` is just a helper service that opens Toast messages (using [Angular Material](https://material.angularjs.org/)).

### Usage

Querying our API can now be done as following:

```javascript
API.all('products').post({title, price}).then( (response) => {
  var user = response.data.user;
});
```

There's no need to write the error function anymore, because that's being handled by the error interceptor. We know that if the `title` for example doesn't validate correctly, we'll automatically get a toast message saying that.


### What if ...

What if for a specific scenario, you need to perform another action?
That's easy. Just pass in your custom error handler as the second argument for the `then` function.

```javascript
API.all('products').post({title, price}).then( (response) => {
  var user = response.data.user;
}, (response) => {
    //response.status
    //response.data
});
```

Note that this action will be performed as well as the error interceptor.

## Prefer a screencast?

This topic is also discussed in the below screencast

<iframe width="100%" style="max-width:560px;" height="315" src="https://www.youtube.com/embed/RHQNpw-XmpI" frameborder="0" allowfullscreen></iframe>

{% include subscribe.html %}