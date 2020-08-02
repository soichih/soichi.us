---
title: "nodejs tips"
date: 2020-08-01T18:26:57-04:00
draft: true
---

## Add `return` for `setTimeout` call

One of the first thing that node developer needs to gets used to is to deal with callback functions.

Like..

```javascript
function dosomething(input, cb) {
  console.log("doing something with", input)
  //... doing something ...
  cb();
}

dosomething(123, function(err) {
  if(err) throw err;
  console.log("success! now doing another thing..")
});
```

When `dosomething` gets more complicated, though, we often forget to add `return` when calling cb().

Like..

```javascript
//BROKEN

function dosomething(input, cb) {
  if(input < 0) {
    console.log("doing something with input<0")
    //... doing something ...
    cb();
  }
  
  console.log("doing something with normal input");
  //... doing another thing ...
  cb();
}
```

In this case, both "something" and "another thing" blocks gets executed because the first cb() didn't return. This could also lead to double callback calls which could cause some libraries to throw exceptions - like async.

It's a good havit to always add `return` after cb() is called.

```javascript
function dosomething(input, cb) {
  if(input < 0) {
    //... doing something ...
    return cb();
  }
}
```

It's returning the value from cb() not because it's really expecting cb() to return any values, 
but just to make it logically clear that calling cb() is the last thing that this function does.

So far so good..

Now, there is a similar but somewhat less common pattern using `setTimeout` to defer some action and calling of callbacks.

Like..

```
//BROKEN.. do you spot an issue?

function dosomething_when_i_can(cb) {
  if(busy()) {
    console.log("resource is busy.. waiting for a second");
    setTimeout(function() {
      dosomething_when_i_can(cb);
    }, 1000);
  }

  //... do something 
  return cb();
}
```

This function checks to see if resource is busy, and if it is, it calls setTimeout() to call itself to *postpone* the execution of this function. Do you spot a mistake here? This function not only fails to delay the action, but it ends up making the same action over and over again (while it's busy)

When `setTimeout` is used to delay some action, it should almost always be accompanied by `return`, just like cb(). Even if it's the last thing that the funtion does currently, you might go ahead and return because you might add things later and forget that `setTimeout` isn't returning..

So, above function shouldn've been written like this

```
function dosomething_when_i_can(cb) {
  if(busy()) {
    console.log("resource is busy.. waiting for a second");
    return setTimeout(function() {
      dosomething_when_i_can(cb);
    }, 1000);
  }

  //... do something 
  return cb();
}
```

If setTimeout isn't used to delay any action, you still probably want to keep the timeout object so that you can cancel or do something with it later. 

```
var to = setTimeout(stuck_handler, 1000);

//... do something

//finished normally.. so cancel stuck_handler
clearTimeout(to);

```

So, if you see a naked `setTimeout` call without a return or assignment, you should watch out!

## When to use else or not to use

I often have trouble deciding if I should use if() block with return at the end, or use else statement. Like..

```
fs.readdir((err, files)=>{
  if(err) {
    console.error(err);
    return;
  }
  //do something with files
});

```

OR

```
fs.readdir((err, files)=>{
  if(err) {
    console.error(err);
  } else {
    //do something with files
  }
});

```
