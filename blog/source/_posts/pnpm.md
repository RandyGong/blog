---
title: pnpm
date: 2022-07-25 18:06:39
tags:
- npm
- pnpm
---

First of all, let's briefly introduce pnpm, which is NodeJS package management tool, but it is not the optimization compatibility mode of NPM or yarn. Through the optimization of node_modules The processing of modules makes the management of modules faster and more efficient.

First, let's take a look at the characteristics of pnpm

- Fast: pnpm is twice as fast as the traditional solution (yarn, NPM) installation package, and even faster than yarn2, PNP mode. It is more strict and efficient: node_ The files in modules are linked from a single content addressable storage, and the code cannot access any package. 
- Monorepo: natural built-in support when the warehouse has multiple packages

From the above three points, this paper explains why we can use pnpm to solve the problems that NPM cannot solve, and how to save disk space and improve installation speed.

**fast**
It is twice as fast as the installation package of the traditional scheme. The following are the official benchmarks. In many common cases, the speed of installing is compared.

![image.png-45.8kB][1]

It can be seen that in different combinations, the installation speed of pnpm is better than that of NPM and yarn (including the PNP mode of yarn). In our own project, when cache and lock exist, the installation rate is significantly improved.
Comparison of installation packages in the 50+ sub application monorepo project.

![image.png-57.5kB][2]

pnpm：node_ The size of modules is 1.6g, and the installation takes 23S
yarn：node_ The size of modules is 1.9g, and the installation takes 59S

The installation time improves the efficiency by more than 20%.

Pnpm can have such a fast installation speed thanks to its package management mechanism, which saves disk space and improves installation speed.
When using NPM or yarn, if you have 100 projects that use a dependency, 100 copies of the dependency will be saved on disk. For pnpm, the dependency will be stored in a content addressable warehouse. To do this, you need to

- All files will be stored in the same location on the hard disk. When multiple packages are installed, all files will create hard links from the same location, which will not occupy additional disk space, and it is allowed to share the same version dependency across projects.

![image.png-159.8kB][3]

Comparing the installation of express by yarn and pnpm, each project of yarn will be reinstalled. When pnpm add express is executed, it does not reinstall express, but links the content addressed resources to the virtual file storage in the project in the form of hard connection, references 52 modules, and does not download modules.

- Different versions of a dependency are used in the project, and only different files will be added to the warehouse. For example, it has 100 files, and the new version only changes one of them. Executing pnpm update will only add a new file to the storage center, and will not clone the entire project because of a single change.

![image.png-168.2kB][4]

When verifying the first point, we installed express4.17.1. At this time, we verified the second point by installing different versions of express. We can see that through hard link express, only three modules were downloaded, and 53 modules were repeatedly referenced.

Through the above operations, pnpm manages local projects through centralization and hard link, which saves a lot of disk space and has faster installation speed.

**Strict and efficient**
Strict and efficient is through comparison. Before explaining the characteristics of pnpm, first sort out how NPM implements dependency management and what problems exist.

**Npm**
Speaking of how NPM manages nodes_modules, there are two cases.

![image.png-186.6kB][5]

![image.png-159.7kB][6]

In npm 1 and 2, packages are managed through nested structures. Managing packages in this way has many disadvantages

- Dependencies cannot be shared. For example, bar and bar1 refer to the same version of foo, and the nested structure will install foo twice, resulting in a waste of disk space and inefficiency in installing dependencies.
- The dependency level is too deep, resulting in long file paths. There are problems under different operating systems. Copying directories in windows will report errors, and the path length exceeds the limit.

From npm3
Through flattening, we can solve the problem that dependencies cannot be shared and the dependency level is too deep. All dependencies are tiled on the node_modules First level directory in modules. In this way, when installing new dependencies, the node is recursively searched upward according to the path search algorithm of the node loading node_module The core of package in modules is simply

- Find node in the same level directory node_modules folder
- If there is no node under the same level directory node_modules or the dependency of the relevant version is not found. It will continue to search in the upper directory, which solves the problem of repeated package installation, and the size of local development projects has been improved.

With node_modules With the introduction of flat modules, the solution of problems is also accompanied by the emergence of problems.

**Phatom**
Because of flattening, all dependencies are promoted to node_modules The primary directory of modules, resulting in undeclared packages in the workspace that can be directly referenced by the project. Doing so will cause the following two problems, take express as an example.

- The project can directly reference debug, but the debug version cannot be specified, which leads to hidden dangers every time the project executes install, releases and goes online
- For local development, the dependency declared in devdependence is also promoted to node_modules The primary directory of modules was introduced without declaration in the project, resulting in online problems

![image.png-138.6kB][7]

**Doppelgangers**
Because flattening will raise the directory level of the package, but there are different versions of the same package. How does NPM solve it. The solution is shown in the following figure

![image.png-181.7kB][8]

Select a version to upgrade to node_modules level-1 directory. Other versions are nested and installed. This will lead to the use of two instances of B when referring to C and D in the project. In some boundary conditions, the project will crash (typescript and webpack may make errors, so they can only be internally compatible).

**Pnpm management**
Pnpm can solve the problems mentioned above. What we have been discussing is node_modules, only jump out of this circle and be compatible with node_ Modules can solve these problems. To sum up, pnpm realizes the management of NPM modules through automatic hard link and sybolic link.

**Inside the workspace (sybbolic link)**
Take express as an example. At node_ There is only one express in the root directory of modules. Unlike NPM, which flattens all dependencies, it links to.Pnpm in the form of a soft chain, and then manages the dependent versions in detail, so as to solve Phatom and doppelgangers together Pnpm is also tiled internally, but it allows different versions of dependencies to be tiled at one level. At the same time, many dependencies on express are also directly soft linked to the tiled module.

![image.png-176.7kB][9]

**Global store (hard link)**
Through the analysis of the interior of the workspace, we can see that they are all realized through soft links. Now let's continue to see how hard links work Pnpm directory, the first-class files of this directory, are linked to the global storage space through direct hard links, so that multiple projects can share a dependency.

![image.png-890.1kB][10]

**Monorepo**

Traditional scheme
The commonly used monorepo management tools include lerna and yarn workspace. These monorepo management tools mainly solve the problems encountered in managing multiple packages in one warehouse

- The third party relies on repeated installation. Packages a and B managed by the workspace refer to express, which will be installed twice in the two packages, resulting in a waste of disk space and an increase in local installation time
- Reference between modules: if different packages in a workspace need to reference each other, you need to manually perform the link operation

Both yarn and lerna can be summarized as

- Install all package dependencies in the root directory node of the workspace in a flat node_modules. At the same time, for different versions of the same dependency, install one version in the root directory and the other versions in the node under their respective node_modules, which solves the conflict problem of relying on different versions
- By soft chaining all packages to the root directory node_modules, each package can import other packages by using the recursive search mechanism of node without manual link
- By combining the nodes in each node_modules The bin folder of modules is soft linked to the node in the root directory node_modules to ensure that the NPM script of each package can run normally

Although the core problem is solved, other problems are introduced

- Phantom dependencies are amplified. For example, all dependency declarations are flattened in the root directory node_modules, because of the recursive search of node, you can access the dependencies of any other package and the dependencies of dependencies.
- Doppelgangers are more likely to appear. A large number of them depend on different versions and appear randomly on node_modules The first layer of modules, or the dependent dependency.

**pnpm**
Pnpm has built-in support for monorepo. Just create pnpm-workspace.yaml and Npmrc configuration file also supports a variety of configurations. Compared with lerna and yarn workspace, pnpm not only solves the problem of monerpo, but also solves the problem of traditional solutions.

**Solve Phatom**
Compared with yarn, try to promote all dependencies to the node of the root directory node_modules are tiled, while pnpm only deals with explicitly declared dependencies. For dependencies declared only in a single package, they are only linked to /node of the root directory in the form of a soft chain node_modules/.pnpm directory, which avoids the recursive search mechanism of node and solves the expanded phantom dependency in monorepo.

![image.png-191.7kB][11]

**Solve Doppelgangers**
Pnpm not only solves the phantom dependency, but also solves the separation problem, because in the package management under monorepo, the dependencies of each package are linked to the node of the root directory through a soft chain node_modules/.pnpm, and tile it in it. You can see that in the following figure, there are different versions of express in the root directory and various packages, which are referenced in the form of soft chain, and various versions of express are tiled in.Pnpm.

![image.png-256kB][12]

We can find that pnpm avoids relying on node_modules The search rules of modules are directly managed in a unified way through the soft chain, which directly solves the problems that NPM and yarn have not solved, saving a lot of disk space and accelerating the installation speed.

## Conclusion

Through the above comparative analysis, we can see that pnpm is the real solution to node_modules' dependency dilemma is mainly through the combination of soft link and hard link, which ultimately achieves the advantages of saving disk space, fast installation speed, strict and efficient.


  [1]: https://iti-images.s3.amazonaws.com/events/1b3c8c73-75c6-1ac9-0176-df2ae3c61e3b.webp
  [2]: https://iti-images.s3.amazonaws.com/events/94e5c1f4-09a2-1d55-13c0-bb302297da24.webp
  [3]: http://static.zybuluo.com/RandyGong/pfjsr6qk72vlwhrrlg05mecw/image.png
  [4]: http://static.zybuluo.com/RandyGong/3m3ez8vzziwgqzvf3t6lc94a/image.png
  [5]: http://static.zybuluo.com/RandyGong/ukg6hz10jzdhme0tq95nih88/image.png
  [6]: http://static.zybuluo.com/RandyGong/ule1fkco3q32vq1oa0ux0eg4/image.png
  [7]: http://static.zybuluo.com/RandyGong/1pxcvcf0y6448a15vo17izze/image.png
  [8]: http://static.zybuluo.com/RandyGong/llz0jesfgu079m0796cwr66t/image.png
  [9]: http://static.zybuluo.com/RandyGong/suc204rhdugk1p1w9bn68zul/image.png
  [10]: http://static.zybuluo.com/RandyGong/mnls7q5j4qo4lcr9jpv0iidy/image.png
  [11]: http://static.zybuluo.com/RandyGong/0gerjo5if4rpkmi8tiswhwvb/image.png
  [12]: http://static.zybuluo.com/RandyGong/bozpgn0qxo3x69fjw77jh76y/image.png