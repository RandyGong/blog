---
title: How to write async await without try-catch blocks in Javascript
date: 2022-09-21 19:45:47
tags:
- Error handling
- async/await
---

This is the introduction from await-to-js

ES7 Async/await allows us as developers to write asynchronous JS code that look synchronous. In current JS version we were introduced to Promises, that allows us to simplify our Async flow and avoid Callback-hell.

A callback-hell is a term used to describe the following situation in JS:

```Javascript
function AsyncTask() {
   asyncFuncA(function(err, resultA){
      if(err) return cb(err);

      asyncFuncB(function(err, resultB){
         if(err) return cb(err);

          asyncFuncC(function(err, resultC){
               if(err) return cb(err);

               // And so it goes....
          });
      });
   });
}
```

This makes it hard to maintain code and managing control flow really hard. Just think about an if statement that needs to execute other Async method if some result from callbackA equals to 'foo'.

### Promises to the rescue

With promises and ES6 on board, we can simplify our previous code nightmare to something like this:

```Javascript
function asyncTask(cb) {

   asyncFuncA.then(AsyncFuncB)
      .then(AsyncFuncC)
      .then(AsyncFuncD)
      .then(data => cb(null, data)
      .catch(err => cb(err));
}
```

Looks much nicer don't you think?

But in real world scenarios the async flow might get a little more complex, for instance in your server model (nodejs) you might want to save an entity to a database, then look for some other entity based on saved value, if that value exists, do some other async task, after all tasks finished you might want to respond to the user with the created object from step 1. And if an error occurred during one of the steps you want to inform the user with the exact error.

With promises of-course it would look much cleaner then with plain callbacks, but still, it can get a little messy IMHO.

### ES7 Async/await

That's where I find async await really useful, it allows you to write code like this:

```Javascript
async function asyncTask(cb) {
    const user = await UserModel.findById(1);
    if(!user) return cb('No user found');

    const savedTask = await TaskModel({userId: user.id, name: 'Demo Task'});
    
    if(user.notificationsEnabled) {
         await NotificationService.sendNotification(user.id, 'Task Created');  
    }

    if(savedTask.assignedUser.id !== user.id) {
        await NotificationService.sendNotification(savedTask.assignedUser.id, 'Task was created for you');
    }

    cb(null, savedTask);
}
```

The code above looks much cleaner, but, what about error handling?

When making async calls something may happen during the execution of the promise(DB connection error, db model validation error, etc..)

Since async functions are waiting for Promises, when a promise encounters an error it throws an exception that will be catched inside a catch method on the promise.

In async/await functions it is common to use try/catch blocks to catch such errors.

I'm not coming from a typed language background, so the try/catch adds for me additional code that in my opinion doesnt look that clean. I'm sure it's a matter of personal preference, but that's my opinion.

So the previous code will look something like this:

```Javascript
async function asyncTask(cb) {
    try {
       const user = await UserModel.findById(1);
       if(!user) return cb('No user found');
    } catch(e) {
        return cb('Unexpected error occurred');
    }

    try {
       const savedTask = await TaskModel({userId: user.id, name: 'Demo Task'});
    } catch(e) {
        return cb('Error occurred while saving task');
    }

    if(user.notificationsEnabled) {
        try {
            await NotificationService.sendNotification(user.id, 'Task Created');  
        } catch(e) {
            return cb('Error while sending notification');
        }
    }

    if(savedTask.assignedUser.id !== user.id) {
        try {
            await NotificationService.sendNotification(savedTask.assignedUser.id, 'Task was created for you');
        } catch(e) {
            return cb('Error while sending notification');
        }
    }

    cb(null, savedTask);
}
```

### A different way of doing things

Recently I've been coding with go-lang and really liked their solution that looks something like this:

```
data, err := db.Query("SELECT ...")
if err != nil { return err }
```

I think it's cleaner than using the try-catch block and clusters the code less, which makes it readable and maintainable.

But the problem with await is that it will silently exit your function if no try-catch block was provided for it. And you won't have a way to control it unless providing the catch clause.

When me and Tomer Barnea a good friend of mine sat and tried to find a cleaner solution we finished using the next approach:

Remember that await is waiting on a promise to resolve?

With that knowledge we can make small utility function to help us catch those errors:

```Javascript
// to.js
export default function to(promise) {
   return promise.then(data => {
      return [null, data];
   })
   .catch(err => [err]);
}
```

The utility function receives a promise, and then resolve the success response to an array with the return data as second item. And the Error received from the catch as the first.

And then we can make our async code to look like this:

```Javascript
import to from './to.js';

async function asyncTask() {
     let err, user, savedTask;

     [err, user] = await to(UserModel.findById(1));
     if(!user) throw new CustomerError('No user found');

     [err, savedTask] = await to(TaskModel({userId: user.id, name: 'Demo Task'}));
     if(err) throw new CustomError('Error occurred while saving task');

    if(user.notificationsEnabled) {
       const [err] = await to(NotificationService.sendNotification(user.id, 'Task Created'));  
       if (err) console.error('Just log the error and continue flow');
    }
}
```

The example above is just a simple use-case for the solution, you can attach interceptor inside the to.js method which will receive the raw error object, log it or do whatever you need to do with it before passing it back.

There is a simple NPM package we created for this library, you can install it using:
[Github Repo][1]

> npm i await-to-js

The source code of await-to-js:

```Javascript
/**
 * @param { Promise } promise
 * @param { Object= } errorExt - Additional Information you can pass to the err object
 * @return { Promise }
 */
export function to<T, U = Error> (
  promise: Promise<T>,
  errorExt?: object
): Promise<[U, undefined] | [null, T]> {
  return promise
    .then<[null, T]>((data: T) => [null, data])
    .catch<[U, undefined]>((err: U) => {
      if (errorExt) {
        const parsedError = Object.assign({}, err, errorExt);
        return [parsedError, undefined];
      }

      return [err, undefined];
    });
}

export default to;
```

  [1]: https://github.com/scopsy/await-to-js