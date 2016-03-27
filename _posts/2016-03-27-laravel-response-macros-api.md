---
layout: post
title: Laravel Response Macros for APIs
---

I would like to show you a use case for response macros when building an API with Laravel.


### Dealing with APIs

Consistency is one of the most important aspects to consider when building an **API**.
You often find yourself returning 2 types of responses; one for success and one for errors.

```php
<?php

class PostsController{
  public function get(){
    try {
      //some code
    }catch (Exception $e){
      //error
      return [
        'errors'  => true,
        'message' => $e->getMessage()
      ];
    }

    $posts = Post::get();
  
    //success
    return [
      'errors' => false,
      'data'   => compact('posts')
    ];
  }
}
```

As you can see, we are returning error responses in a specific format, and success responses in a different format.  
Soon enough you'll notice that you're repeating this over and over again.

### Using response macros

[Response macros](https://laravel.com/docs/5.2/responses#response-macros){:target="_blank"}
allow you to define a macro that can be called on the `response()` function.  

We will create a new `ResponseMacroServiceProvider` and define a `success` and `error` macros inside the `boot` function: 

```php
<?php

class ResponseMacroServiceProvider extends ServiceProvider
{
  public function boot()
  {
    Response::macro('success', function ($data) {
        return Response::json([
          'errors'  => false,
          'data' => $data,
        ]);
    });

    Response::macro('error', function ($message, $status = 400) {
        return Response::json([
          'errors'  => true,
          'message' => $message,
        ], $status);
    });
  }
}
```

So now we can refactor the previous code:

```php
<?php

class PostsController{
  public function get(){
    try {
      //some code
    }catch (Exception $e){
      return response()->error($e->getMessage);
    }

    $posts = Post::get();
  
    return response()->success(compact('posts'));
  }
}
```

As you can see, `response macros` help us remain consistent when returning data from the API. Whether it was a success or an error.  
This means your front-end API calls will expect a consistent response.  


I will write a new tutorial soon on how you can take it a step further in your front-end by implementing Response Interceptors to handle error responses.

{% include subscribe.html %}