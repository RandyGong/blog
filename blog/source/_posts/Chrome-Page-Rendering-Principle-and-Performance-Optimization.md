---
title: Chrome Page Rendering Principle and Performance Optimization
date: 2020-09-08 12:09:41
tags:
- performance
---

## Introduction to Chrome Basic Architecture
### Overall Structure

![Markdown](https://app.yinxiang.com/shard/s57/res/43829af6-6700-4cdf-b0ec-45f066d9de93/%E6%97%A0%E6%A0%87%E9%A2%98.jpg)

The main function of the browser is to send a request to the server to display the selected network resources in the browser window. The resources mentioned here generally refer to HTML documents, or PDF, pictures or other types. In general, the browser can be divided into five parts:

- The user interface is mainly responsible for displaying the page. In addition to the content of the page itself, we can roughly understand that the interface presented when opening an empty page is the browser's user interface (GUI)

- Browser engine, here I think it mainly refers to the transfer of instructions between the user interface and the rendering engine, as well as scheduling the resources of the browser in various aspects, coordinating the work for rendering pages and completing user instructions

- The rendering engine, as shown in the above figure, contains a compositor and a JavaScript engine. They are responsible for parsing HTML and CSS content, displaying the parsed content on the screen and parsing and executing JavaScript code

- Back end service layer, which contains some back-end services. Such as network, data storage. Browser needs to save all kinds of data on the hard disk, such as cookie, storage and so on

- The special service layer mainly refers to some services provided by the browser. For example, if you fill in the account and password of a certain website, the browser can help you remember the account and password, and open the browser's dark mode and other special services

For the front end, the more important is the rendering engine (some articles also called browser engine). We can look at the kernel of the rendering engine.

| Browser / Runtime        | Kernel (Rendering Engine)     |  Javascript Engine  |
| :--------------------:   | :--------------------------:  | :----------------:  |
| Chrome                   | Blink(28) / Webkit (Chrome27) |   V8     |
| Firefox                  |   Gecko                       |   SpiderMonkey   |
| Edge                     |    EdgeHTML                   |  Chakra (for Javascript)  |
| IE                       |    Trident                    |  Chakra (for Javascript)  |
| PhantomJS                |    Webkit                     |  JavascriptCore  |
| NodeJS                   |    -                          |  V8  |

### Multi Process Architecture

The early web browser is a single process, the improper page behavior, browser errors, browser plug-ins and other errors will cause the entire browser or the currently running tab to close. Therefore, chrome places the chrome application in separate processes, that is, a multi process architecture.

![Browser Architecture](https://app.yinxiang.com/shard/s57/res/f77d1651-dc3b-42b6-beec-dc5b989ff544/Client.jpg)

**The advantages of multi process are:**

- Prevent a page crash from affecting the entire browser

- Security and sandbox, because the operating system provides a method to restrict the process permissions, the browser can sandbox some processes from some functions. For example, chrome - browsers can restrict file access to processes that process user input, such as renderers

- Processes have their own private memory space and can have more memory

**Yet the advantages of multi process are:**

- Each process is allocated a separate memory, although chrome itself has some optimization strategies. For example, to save memory, chrome limits the number of processes it can start. Limits vary depending on the memory and CPU power of the device, but when chrome reaches the limit, it starts running multiple tabs from the same site in one process

- Higher resource utilization. Because each process contains a copy of the common infrastructure (such as the JavaScript runtime environment), this means that the browser consumes more memory resources

Multi process architecture still have some areas for optimization. Therefore, the future architecture of chrome is a service-oriented architecture. Each part of the browser program is run as a service, which can be easily divided into different processes or aggregated into the same process. This can be achieved. When chrome runs on powerful hardware, it may split each service into different processes, thus providing higher stability. However, if it is located on a resource constrained device, chrome will integrate services into one process, thus integrating processes to reduce memory usage.

## Page Rendering Process in Browser
According to the time sequence of rendering, rendering pipeline can be divided into the following sub stages: building DOM tree, style calculation, layout phase, layering, rasterization and display.

```graphLR
    A[HTML Javascript CSS] --> B(Round edge)
    B[Create DOM tree] --> C(Round edge)
    C[Calculate style] --> D(Round edge)
    D[Layout] --> E(Round edge)
    E[Generate LayerTree] --> F(Round edge)
    F[Rasterization ] --> G
    G[Displaying]
```

1. The rendering process converts the HTML content to be able to read the DOM tree structure.
2. The rendering engine converts CSS style sheets into stylesheets that the browser can understand, and calculates the style of DOM nodes.
3. Create a layout tree and calculate the layout information for the elements.
4. The layout tree is layered and a hierarchical tree is generated.
5. Generate a drawing list for each layer and submit it to the composition thread. The composition thread divides the layer into blocks and rasterizes the blocks into bitmaps.
6. The composition thread sends the drawing block command to the browser process. The browser process generates pages according to the instructions and displays them on the display.

### Create Dom Tree
After getting HTML byte data from the network or hard disk, the browser will parse the byte into DOM tree through a process. First, the original byte data of HTML will be converted into characters with specified encoding in the file, and then the browser will convert the string into various token tags, such as HTML, body, etc. according to the HTML specification. Finally, it is resolved into a tree object model, named DOM tree.

``` html
<html>
    <body>
        <p>
            Hello World
        </p>
        <div>
            <img src="example.png" />
        </div>
    </body>
</html>
```

```graphTB
    A[HTMLHtmlElement] --> B
    B[HTMLBodyElement] --> C
    B[HTMLBodyElement] --> D
    C[HTMLParagraphElement] --> E[Text]
    D[HTMLDivElement] --> F
    F[HTMLImageElement] --> G[Image]
```

1. Transcoding (bytes > characters) -- reads the received HTML binary data, and converts the bytes into HTML strings according to the specified encoding format
2. Characters > tokens -- parsing HTML, converting HTML strings into structured tokens. Each token has a special meaning and has its own set of rules
3. Build nodes (tokens > nodes) - each node adds a specific attribute (or property accessor), and the parent, child, sibling relationship and tree scope of the node can be determined by the pointer (for example, the treescope of iframe is different from that of the outer page)
4. Building DOM tree (nodes > DOM tree) - the most important work is to establish the parent-child relationship of each node

### Style Culculation

The rendering engine converts CSS style sheets into stylesheets that the browser can understand, and calculates the style of DOM nodes.

There are three main sources of CSS styles: external CSS files referenced by link, CSS in style tags, and CSS embedded in style attributes of elements.

```CSS
    body { font-size: 2em }
    p { color: blue }
    span { display: none }
    div { font-weight: bold }
```

```CSS
    body { font-size: 32px }
    p { color: rgb(0, 0, 255) }
    span { display: none }
    div { font-weight: 700 }
```

You can see that there are many attribute values in the CSS text above, such as 2em, blue and bold. These types of values are not easy to understand by the rendering engine. Therefore, it is necessary to convert all the values into standardized calculation values that are easy to understand by the rendering engine. This process is called property value standardization. After processing, the inheritance and cascade of styles are processed. Some articles call this process the construction process of CSSOM.

### Page Layout
The layout process is to exclude script, meta and other functional and non visual nodes, exclude display: none nodes, calculate the location information of elements, determine the location of elements, and build a layout tree containing only visible elements.

![Layout](https://app.yinxiang.com/shard/s57/res/983099f1-417a-4c76-9eb3-5fcc58187b3e/640.jpg)

### Generate Layer Tree
There are many complex effects in the page, such as some complex 3D transformation, page scrolling, or z-indexing to sort the z-axis. In order to achieve these effects more conveniently, the rendering engine needs to generate special layers for specific nodes and generate a corresponding layer tree.

If you are familiar with Photoshop, I believe you can easily understand the concept of layers, which are superimposed together to form the final page image. In the browser, you can open Chrome's developer tools and select the layers tab. The rendering engine divides the page into many layers. These layers are superimposed in a certain order to form the final page.

### Rasterize
The composition thread generates bitmaps first according to the blocks near the view. The actual operation of generating bitmaps is performed by rasterization. Rasterization refers to converting blocks into bitmaps.

![Rasterize](https://app.yinxiang.com/shard/s57/res/c2c4119f-0505-4c50-9767-7711dfc773dc/%E6%97%A0%E6%A0%87%E9%A2%98.jpg)

Usually, a page may be very large, but the user can only see a part of it. We call this part of the page that the user can see as a view port. In some cases, some layers can be very large. For example, some pages need to scroll for a long time to scroll to the bottom. However, users can only see a small part of the page by using the scroll bar. Therefore, in this case, it will be too expensive and unnecessary to draw all the contents of the layers.

### Display
Finally, the composition thread sends the drawing block command to the browser process. The browser process generates the page according to the instructions and displays it on the display. The rendering process is completed.

## JavaScript running mechanism in browser
### JS Engine
JavaScript engine is used to execute JS code. The compiler compiles the code into executable machine code for the computer to execute. At present, V8 engine is the most popular engine, which is used by Chrome browser and Node.js. The engine is mainly composed of memory heap and call stack.

### Context
The execution context is the running environment for JavaScript to execute a piece of code. For example, if a function is called, it will enter the execution context of the function, and determine the functions, such as this, variables, objects, and functions, etc. used during the execution of the function.

There are three types of execution context in JavaScript：

- Global execution context - this is the default or basic context in which any code that is not inside a function is in the global context. It does two things: create a global window object (in the case of a browser) and set the value of this equal to the global object. There is only one global execution context in a program
- Function execution context - whenever a function is called, a new context is created for that function. Each function has its own execution context, but is created when the function is called. There can be any number of function contexts. Whenever a new execution context is created, it performs a series of steps in the defined order (discussed later)
- Eval function execution context - code executed inside an eval function also has its own execution context, but because JavaScript developers don't use Eval very often, I won't discuss it here

### Call Stack
The JavaScript engine uses the stack structure to manage the execution context. After the execution context is created, the JavaScript engine will push the execution context into the stack, which is usually referred to as the execution context stack or the call stack.

To view the call stack in the browser:

- When you execute a complex code, it may be difficult to analyze the call relationship from the code file. At this time, you can add breakpoints to the function you want to view, and then when the function is executed, you can view the call stack of the function
- console.trace()

The call stack has a size. When the execution context of the stack exceeds a certain number, the JavaScript engine will report an error. We call this error stack overflow. Generally, stack overflow will not occur in normal business requirements. Stack overflow will occur only when recursion forgets to write boundary. We should pay attention to this when we write code.

### Event Loop
During the execution of JavaScript code, in addition to relying on the function call stack to handle the execution order of functions, it also relies on the task queue to complete the execution of other codes. Throughout the execution process, we become the event loop process. In a thread, the event loop is unique, but the task queue can have more than one. Task queue is divided into macro task and micro task. In the latest standard, they are called task and jobs respectively.