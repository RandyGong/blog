---
title: How does Axios cancel duplicate requests?
date: 2019-07-17 23:11:21
categories:
- Javascript
tags:
- frontend
- web
- axios
---

In the process of Web project development, we often encounter repeated requests. If the system does not process repeated requests, various problems may occur in the system. For example, duplicate post requests may cause the server to generate two records. So how do duplicate requests come about? Here are two common scenarios:

- Suppose there is a button in the page, and the user will initiate an Ajax request after clicking the button. If the button is not controlled, a repeat request will be issued when the user quickly clicks the button.

- Suppose that in the examination result query page, users can query the examination results according to three query criteria: passed, failed and all. If the response of the request is slow, repeated requests will be generated when users quickly switch before different query conditions.

Now that we know how duplicate requests are generated, we know that it will cause some problems. Next, we will take Axios as an example to solve the problem of repeated requests.

## How to cancel a request

Axios is a promise based HTTP client that supports both browser and node.js environments. It is an excellent HTTP client and is widely used in a large number of web projects. For the browser environment, the underlying layer of Axios uses the XMLHttpRequest object to initiate HTTP requests. If we want to cancel the request, we can cancel the request by calling the abort method on the XMLHttpRequest object:

```Javascript
let xhr = new XMLHttpRequest();
xhr.open("GET", "https://developer.mozilla.org/", true);
xhr.send();
setTimeout(() => xhr.abort(), 300);

```
For Axios, we can cancel the request through the canceltoken provided by Axios:

```Javascript
const CancelToken = axios.CancelToken;
const source = CancelToken.source();

axios.post('/user/12345', {
  name: 'semlinker'
}, {
  cancelToken: source.token
})

source.cancel('Operation canceled by the user.');
```

In addition, you can also create a canceltoken by calling the constructor of canceltoken, as shown below:

```Javascript
const CancelToken = axios.CancelToken;
let cancel;

axios.get('/user/12345', {
  cancelToken: new CancelToken(function executor(c) {
    cancel = c;
  })
});

cancel();
```

Now that we know how to use canceltoken to cancel requests in Axios, how does canceltoken work internally? Let's remember this question first. Later, I will uncover the secret behind canceltoken for you. Next, let's analyze how to judge duplicate requests.

## How to judge duplicate requests

When the request method, request URL address and request parameters are the same, we can think that the request is the same. Therefore, each time a request is initiated, we can generate a unique key according to the request method, request URL address and request parameters of the current request, create a unique canceltoken for each request, and then save the key and cancel functions in the form of key value pairs in the map object, The advantage of using map is that you can quickly judge whether there are repeated requests:

```Javascript
import qs from 'qs'

const pendingRequest = new Map();
// GET -> params；POST -> data
const requestKey = [method, url, qs.stringify(params), qs.stringify(data)].join('&'); 
const cancelToken = new CancelToken(function executor(cancel) {
  if(!pendingRequest.has(requestKey)){
    pendingRequest.set(requestKey, cancel);
  }
})
```

When repeated requests occur, we can use the cancel function to cancel the previous requests. After canceling the requests, we also need to remove the canceled requests from the pendingrequest. Now that we know how to cancel a request and how to judge a duplicate request, let's introduce how to cancel a duplicate request.

## How to cancel duplicate requests

Because we need to process all requests, we can consider using Axios interceptor mechanism to realize the function of canceling duplicate requests. Axios provides developers with request interceptors and response interceptors. Their functions are as follows:

- Request Interceptor: this type of interceptor is used to perform certain operations uniformly before sending the request, such as adding the token field in the request header

- Response Interceptor: this type of interceptor is used to perform certain operations uniformly after receiving the server response. For example, when the response status code is 401, it will automatically jump to the login page

### Define helper functions

Before configuring the request interceptor and response interceptor, let's define three helper functions:

- Generatereqkey: used to generate a request key according to the information of the current request
```Javascript
function generateReqKey(config) {
  const { method, url, params, data } = config;
  return [method, url, Qs.stringify(params), Qs.stringify(data)].join("&");
}
```

- Addpendingrequest: used to add the current request information to the pendingrequest object
```Javascript
const pendingRequest = new Map();
function addPendingRequest(config) {
  const requestKey = generateReqKey(config);
  config.cancelToken = config.cancelToken || new axios.CancelToken((cancel) => {
    if (!pendingRequest.has(requestKey)) {
       pendingRequest.set(requestKey, cancel);
    }
  });
}
```

- Removependingrequest: check whether there are duplicate requests. If so, cancel the sent requests
```Javascript
function removePendingRequest(config) {
  const requestKey = generateReqKey(config);
  if (pendingRequest.has(requestKey)) {
     const cancelToken = pendingRequest.get(requestKey);
     cancelToken(requestKey);
     pendingRequest.delete(requestKey);
  }
}
```

After creating the generatereqkey, addpendingrequest and removependingrequest functions, we can set the request interceptor and response interceptor.

### Set request interceptor
```Javascript
axios.interceptors.request.use(
  function (config) {
    removePendingRequest(config); // Check whether there are duplicate requests. If so, cancel the sent requests
    addPendingRequest(config); // Add the current request information to the pendingrequest object
    return config;
  },
  (error) => {
     return Promise.reject(error);
  }
);
```

### Set response interceptor
```Javascript
axios.interceptors.response.use(
  (response) => {
     removePendingRequest(response.config); // Remove the request from the pendingrequest object
     return response;
   },
   (error) => {
      removePendingRequest(error.config || {}); // Remove the request from the pendingrequest object
      if (axios.isCancel(error)) {
        console.log("Cancelled duplicate requests：" + error.message);
      } else {
        // Add exception handling
      }
      return Promise.reject(error);
   }
);
```

Here, let's take a look at the running results of the Axios de duplicated request example:


![Cancel duplicated requests](https://downloads.intercomcdn.com/i/o/377532508/40a53729b9e72880185fb234/%E5%9B%BE%E7%89%87.png)

As can be seen from the above figure, when a duplicate request occurs, the previously sent and unfinished requests will be cancelled. Let's use a flowchart to summarize the process of canceling duplicate requests:

![Process](https://downloads.intercomcdn.com/i/o/377533619/8550a4b40d6760dcaf20f1a8/%E5%9B%BE%E7%89%87.png)

Finally, let's answer the previous question, that is, how does canceltoken work internally?

## How canceltoken works

In the previous example, we created the canceltoken object by calling the canceltoken constructor:
```Javascript
new axios.CancelToken((cancel) => {
  if (!pendingRequest.has(requestKey)) {
    pendingRequest.set(requestKey, cancel);
  }
})
```

So next, let's analyze the canceltoken constructor, which is defined in the Lib / cancel / canceltoken.js file:

```Javascript
function CancelToken(executor) {
  if (typeof executor !== 'function') {
    throw new TypeError('executor must be a function.');
  }

  var resolvePromise;
  this.promise = new Promise(function promiseExecutor(resolve) {
    resolvePromise = resolve;
  });

  var token = this;
  executor(function cancel(message) {
    if (token.reason) {
      return; // Cancellation has already been requested
    }
    token.reason = new Cancel(message);
    resolvePromise(token.reason);
  });
}
```

From the above code, we can see that the cancel object is a function. When we call this function, we will create the cancel object and call the resolvepromise method. After the method is executed, the state of the promise object pointed to by the promise attribute on the canceltoken object will change to resolved. So what is the purpose of this? Here we find the answer from the Lib / adapters / xhr.js file:

```Javascript
if (config.cancelToken) {
  config.cancelToken.promise.then(function onCanceled(cancel) {
    if (!request) { return; }
    request.abort();
    reject(cancel);
    request = null;
  });
}
```

After reading the above content, some friends may not understand the working principle of canceltoken very well, so I drew another picture to help you understand the working principle of canceltoken:

![Principle](https://downloads.intercomcdn.com/i/o/377539247/958fdd6425d79614c485376a/%E5%9B%BE%E7%89%87.png)
