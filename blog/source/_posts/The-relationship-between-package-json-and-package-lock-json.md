---
title: The relationship between package.json and package-lock.json
date: 2021-11-24 23:50:15
tags:
- npm
- package.json
- package-lock.json
---

Modular development is becoming more and more popular in the front end. Using node and NPM can easily download the dependent modules required by the management project. package.json is used to describe the project and the module information that the project depends on.

What is the relationship between package.json and package-lock.json?

## package.json

As we all know, **package.json is used to describe the project and the module information that the project depends on.**, It helps us manage the dependency packages in the project, and keeps us away from the dependency hell.

Through NPM management, use some simple commands to automatically generate package.json, installation package dependencies are managed by package.json, and we hardly need to consider them.

## Semantic version control

First, let's understand the definition of the version number of the dependent package

The version number consists of three parts: major.minor.patch, major version number, minor version number and patch version number.

For example: 1.2.3, major version 1, minor version 2, patch 3.

- Changes in the patch indicate bug fixes that do not destroy anything.
- A minor version change means a new feature that will not destroy anything.
- The major version changes represent a major change that undermines compatibility. If users do not adapt to major version changes, the content will not work properly.

## How to specify the version of the installation dependent package

I believe you will experience that when we install some dependent packages, the version number will be preceded by the symbol ^ or ~. What do these two symbols mean?

~It will match the latest minor version dependent packages. For example, ~1.2.3 will match all 1.2. X versions, but not 1.3.0

^It will match the latest large version dependent packages. For example, ^1.2.3 will match all 1.x.x packages, including 1.3.0, but excluding 2.0.0

*Install the latest version of dependent packages. For example, *1.2.3 will match x.x.x,

So how to choose? Of course, you can specify a specific version number, write 1.2.3 directly, and there is no prefix in front of it.

This is no problem, but if you rely on the package to release a new version and fix some small bugs, you need to modify the package.json file manually. Then ~ and ^ can solve this problem.

However, it should be noted that ^ version update may be relatively large, which will cause project code errors, so it is recommended to use ~ to mark the version number, so as to ensure that there will be no major problems in the project and that small bugs in the package can be repaired.

The version number is written as *, which means that the latest version of the dependent package is installed, but the disadvantages are the same as above, which may cause version incompatibility. Use with caution!

## The problem of relying on package installation in multi person development

After looking at the above specification of version number, we can know that when we use ^ or ~ to control the version number of dependent packages, there may be different versions of dependent packages installed by everyone, and the results of project operation will be different.

Let's take an example:

Suppose Vue is installed in our project. When we run **npm install vue -save**, the package in the project The Vue version of package.json is vue: ^3.0.0. The Vue version installed on our computer is 3.0.0. After we submitted the project code, a period of time later, Vue released a new version 3.0.1. At this time, a new colleague cloned the project from the **git clone**. When **npm install** was executed, the Vue version on his computer was 3.0.1, because ^ only locked the main version, so the Vue version in our computer would be different, Theoretically (if everyone follows semantic version control), they should still be compatible, but maybe bugfix will affect the functions we are using, and our applications will produce different results when running with Vue versions 3.0.0 and 3.0.1.


Let's think about it. In this way, whether the dependent version projects installed on different people's computers may be different, which will lead to different results for the applications running on each person's computer. There will be hidden dangers of bugs.

At this time, some students may think that we are in package Just lock the version number of the dependent package on package.json? Write Vue directly: 3.0.0 is locked, so the version of Vue installed by everyone is 3.0.0.

This idea is certainly good, but you can only control your own project to lock the version number. What about the dependent packages in your project? How do you control and restrict others to lock the version number?

In order to solve this problem, all dependent versions installed on different human computers are consistent, and ensure that the running results of the project code during installation are the same. At this time, **package-lock.json** came into being

## package-lock.json

**package-lock.json** is only available after NPM (^5.x.x.x), and there are several changes in the middle

The official document explains this: package-lock.json, it will change the node in node_modules directory tree or package.json, which accurately describes the dependency tree of the NPM package of the current project, and will be based on package-lock.json to ensure the same dependency tree, regardless of whether a dependency has a small version update in the process.

Its generation is to fix the version of the entire dependency tree (lock).

When we run **npm install** in a project, a **package-lock.json** file will be automatically generated, which is in the same directory as **package.json**. **package-lock.json** records some information of the project and the modules it depends on. In this way, the same results will appear in each installation No matter what machine you install it on or when.

When we run the next **npm install**, NPM finds that if there is **package-lock.json** in the project, according to **package-lock.json** instead of **package.json**.

> Note that **package-lock.json** will not be generated when **cnpm install** is used, and will not be based on **package-lock.json** to install dependent packages, it will still use the **package.json**.

## package-lock.json generation logic

Briefly describe **package-lock.json** generated logic. Suppose that we now have three packages. In the project lock-test, the installation depends on A. there is B in project a and C in project B

```Javascript
// package lock-test
{ "name": "lock-test", "dependencies": { "A": "^1.0.0" }}
// package A
{ "name": "A", "version": "1.0.0", "dependencies": { "B": "^1.0.0" }}
// package B
{ "name": "B", "version": "1.0.0", "dependencies": { "C": "^1.0.0" }}
// package C
{ "name": "C", "version": "1.0.0" }
```

In this case, **package-lock.json**, which will generate a structure similar to the following flattening

```Javascript
// package-lock.json
{ 
    "name": "lock-test",  
    "version": "1.0.0",  
    "dependencies": {    
        "A": { "version": "1.0.0" },
        "B": { "version": "1.0.0" },
        "C": { "version": "1.0.0" }  
    }
}
```

If we do not change the package.json in the future, whether it is directly dependent on the a release or indirectly dependent on the B and c release, **package-lock.json** will not be regenerated.

A has released a new version 1.1.0, although our package JSON wrote ^1.0.0, but because of **package-lock.json**, NPM I will not be upgraded automatically,

We can run NPM I manually A@1.1.0 To upgrade.

Because 1.1.0 **package-lock.json** A@1.0.0 Is inconsistent, so **package-lock.json** will be updated The version of 1.1.0.

B has released new versions 1.0.1, 1.0.2, 1.1.0. At this moment, if we don't do anything, we won't automatically upgrade the version of B. But if a releases 1.1.1 at this moment, although it doesn't rely on upgrading B, if we upgrade in our project A@1.1.1 At this time, **package-lock.json** will directly upgrade B to 1.1.0, because at this moment, the latest version of ^1.0.0 is 1.1.0.

After these operations, the package.json of the project lock-test JSON becomes

```Javascript
// package 
lock-test{ "dependencies": { "A": "^1.1.0" }}
```

Corresponding **package-lock.json** file is

```Javascript
{  
    "name": "lock-test",  
    "version": "1.0.0",
    "dependencies": {  
        "A": { "version": "1.1.0" },
        "B": { "version": "1.1.0" },
        "C": { "version": "1.0.0" }
    }
}
```

At this time, we add B to the dependency of our lock-test project. B@^1.0.0, package.json is as follows

```Javascript
{ "dependencies": { "A": "^1.1.0", "B": "^1.0.0" }}
```

After we perform this operation, **package-lock.json** has not been changed, because now **package-lock.json** B@1.1.0 Meet the requirements of ^1.0.0

But if we fix the version of B to 2.x, **package-lock.json** will change to

```Javascript
{ "dependencies": { "A": "^1.1.0", "B": "^2.0.0" }}
```

Because there are two conflicting versions of B, **package-lock.json** will become the following form

```Javascript
{  
    "name": "lock-test",
    "version": "1.0.0",  
    "dependencies": {    
        "A": {      
            "version": "1.1.0",      
            "dependencies": {        
                "B": { "version": "1.1.0" }      
            }    
        },    
        "B": { "version": "2.0.0" },    
        "C": { "version": "1.0.0" }  
    }
}
```

NPM uses nesting to describe this behavior because there is a conflict in the version of B

We don't need to pay attention to this generated algorithm logic in actual development. We just need to understand the
generation logic of **package-lock.json** is to accurately reflect our node_modules structure, and ensure that this structure can be restored.

## Reasons that package-lock.json may be changed unexpectedly

- package.json file was modified
- moved the location of the package

Moving the location of some packages from dependencies to devdependencies will also affect **package-lock.json**, although the packages remain unchanged, the dev field of some packages will be set to true

## Suggestions for development

Generally, NPM install is OK, and it can guarantee that according to **package-lock.json** restores the node at the time of node_modules。

However, in order to prevent the accidents mentioned just now, unless it involves package adjustment, it is recommended to use **npm ci** to install dependencies in other cases, which will avoid abnormal modification of **package-lock.json**,

**npm ci** is more recommended in the continuous integration tool to ensure the accuracy of the construction environment. For the difference between **npm i** and **npm ci**, please refer to the official document.