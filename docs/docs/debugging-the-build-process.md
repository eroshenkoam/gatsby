---
title: Debugging the build process
---

Gatsby's `build` and `develop` steps run as a Node.js application which you can debug using standard tools for Node.js applications.

In this guide you will learn how to debug some code using:

- [Chrome DevTools for Node](#chrome-devtools-for-node)
- [VS Code debugger](#vs-code-debugger)

As an example let's use the following code snippet in a `gatsby-node.js` file:

```js
const { createFilePath } = require("gatsby-source-filesystem")

exports.onCreateNode = args => {
  const { actions, Node } = args

  if (Node.internal.type === "MarkdownRemark") {
    const { createNodeField } = actions

    const value = createFilePath({ node, getNode })
    createNodeField({
      name: `slug`,
      node,
      value,
    })
  }
}
```

There is a bug in this code and using it will produce the error below:

```
TypeError: Cannot read property 'internal' of undefined

  - gatsby-node.js:6 Object.exports.onCreateNode.args [as onCreateNode]
    D:/dev/blog-v2/gatsby-node.js:6:12
```

## Chrome DevTools for Node

### Running Gatsby with the `inspect` flag

In your project directory instead of running `gatsby develop` run the following command:

```shell
node --inspect-brk --no-lazy node_modules/gatsby/dist/bin/gatsby develop
```

- `--inspect-brk` will enable Node's inspector agent which will allow you to connect a debugger. It will also pause execution until the debugger is connected and then wait for you to resume it.
- `--no-lazy` - this will force Node's V8 engine to disable lazy compilation and will help with using breakpoints.

### Connecting DevTools

Open `chrome://inspect` in Chrome browser and connect to a "Remote Target" by clicking the `inspect` link:

![Chrome inspect page](./images/chrome-devtools-inspect.png)

You should see Chrome DevTools start and that code execution is paused at the start of the `gatsby.js` entry file:

![Paused Chrome DevTools](./images/chrome-devtools-init.png)

### Setting up `Sources`

Right now you can't see your files in Sources. You need to add those using the "Add folder to workspace" button and pick the directory with the code you want to debug. If you want to debug code in your `gatsby-node.js` or your local plugins, pick your project directory. If you want debug the `gatsby` package you will have to pick the `gatsby` directory inside `node_modules`.

This example has problematic code in your local `gatsby-node.js` file, so let's add the directory containing it to Sources. You should have a directory with your code in the left pane:

![Files added to Sources tab](./images/chrome-devtools-files.png)

### Using DevTools

Let's go ahead and add a breakpoint just before the place that the error is thrown. To add a breakpoint navigate to `gatsby-node.js` and left click on a line number:

![Added breakpoint](./images/chrome-devtools-new-breakpoint.png)

Now you can resume code execution by clicking the "resume" icon in the DevTools debug toolbar (or press F8 on your keyboard). Gatsby will start running and pause once it reaches a breakpoint, allowing you to inspect variables:

![Breakpoint hit](./images/chrome-devtools-breakpoint-hit.png)

To inspect variables you can hover your mouse over them or go to the `Scope` section in the right-hand pane (either collapse the "Call Stack" section or scroll through it to the bottom).

In the example `Node` is `undefined` and to figure out why, let's go backwards. `Node` is extracted from `args` so let's examine that by hovering `args`:

![Examine variable](./images/chrome-devtools-examine-var.png)

We can now see the problem - `args` doesn't contain `Node` - it contains `node`. So this small typographic mistake was causing our code to fail. Adjusting our code to use a lowercase `node` fixes the problem and we did that without adding tons of `console.log` output!

### Finishing thoughts on DevTools

You can succussfully debug your code using Chrome DevTools but using it isn't really that convenient. There are a lot of steps you need to do manually every time you want to use debugger, so in the next section you'll learn how to use the built-in debugging capabilities of VS Code.

"Why did we go through all those steps only to find out that there are better options?" you might ask. That's a great question and here are couple of reasons:

- This was an introduction to Node.js debugging. Using information from this section you can setup debugging in your code editor or IDE of choice (if it supports node debugging).
- You don't _need_ a code editor or IDE to debug Node.js applications. Using Chrome DevTools is usually a safe fallback.
- Debugging isn't the only thing you can do in Chrome DevTools. Once you connect to DevTools you can use CPU or memory profilers. Check the `Profiler` and `Memory` tabs in DevTools.

## VS Code debugger

Using built in debuggers in code editors is very convenient. You will be able to skip a lot of setup needed to use Chrome DevTools. You will also be able to put breakpoints in the same view you write your code.

We won't go in depth here about how to debug in VS Code - for that you can check the [excellent VS Code documentation](https://code.visualstudio.com/docs/editor/debugging). We will however share a launch configuration needed to run and debug Gatsby:

`launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Gatsby develop",
      "type": "node",
      "request": "launch",
      "protocol": "inspector",
      "program": "${workspaceRoot}/node_modules/gatsby/dist/bin/gatsby",
      "args": ["develop"],
      "stopOnEntry": false,
      "runtimeArgs": ["--nolazy"],
      "sourceMaps": false
    },
    {
      "name": "Gatsby build",
      "type": "node",
      "request": "launch",
      "protocol": "inspector",
      "program": "${workspaceRoot}/node_modules/gatsby/dist/bin/gatsby",
      "args": ["build"],
      "stopOnEntry": false,
      "runtimeArgs": ["--nolazy"],
      "sourceMaps": false
    }
  ]
}
```

After putting a breakpoint in `gatsby-node.js` and using the `Start debugging` command from VS Code you can see the final result:

![VSCode breakpoint hit](./images/vscode-debug.png)

## Additional resources

- [Debugging - Getting Started | Node.js](https://nodejs.org/en/docs/guides/debugging-getting-started/)
- [Debugging with Node.js - Paul Irish talk at Node Summit 2017](https://www.youtube.com/watch?v=Xb_0awoShR8)