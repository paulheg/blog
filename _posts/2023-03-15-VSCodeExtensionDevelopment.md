---
layout: post
title: "What I learned writing a VS Code Webview Extension"
---

# TL;DR

I created an extension to compare images in VS Code:

[**Image Diff**](https://marketplace.visualstudio.com/items?itemName=paulheg.image-diff)

# How it all started

This semester I took part in the computer graphics course at my university.
Part of the course was a mandatory exercise certificate, or how we like to say in German: *Übungsschein*.
To pass the Übungsschein you had to solve various programming tasks ranging from implementing ray tracing
to rasterization with OpenGL.
All tasks had one thing in common: to check if your implementation was correct,
there always were pictures with the expected result along with the tasks.
You could then compare these images with the ones produced by your code.

At first I was comparing them side by side.
In some tasks this wasn't possible.
For example Gaussian blur, it for sure looked blurry, but was it calculated correctly?
There was simply no way telling the difference only with your eyes.

Then I switched to using an online tool: [Diffchecker](https://www.diffchecker.com/) recommended by a friend of mine.

But after a while it was getting on my nerves, to always upload the images to this site.

> There should be a VS Code extension for this.

Sadly there was none that matched my requirements.

What it should do:

- Calculate the difference between to images and display it
- Easy file selection, if there are images with the same name in a different folder in the workspace, show them them first when selecting.
- Zooming, to inspect even small changes

So it was decided, I had to create my own extension for this.


# Every beginning is hard

I had no experience with writing VS Code extensions until then,
other than watching [Ben Awad creating stories for VS Code](https://www.youtube.com/watch?v=ApR-kNXxLUs) and reporting bugs on GitHub of extensions I was using.

I started reading the [default documentation](https://code.visualstudio.com/api/get-started/your-first-extension) and generated my first extension using `yo code`.

I already knew it was possible to display images, since there is a [default image viewer](https://github.com/microsoft/vscode/tree/main/extensions/media-preview) already included.
After looking at the source code of *Diffchecker* I could just make use of the CSS property: ` mix-blend-mode: difference;`.
To do this I had to use [WebViews](https://code.visualstudio.com/api/extension-guides/webview) and [CustomEditors](https://code.visualstudio.com/api/extension-guides/custom-editors).

I did not want to overwrite the default behavior of displaying images so I decided to make my `CustomEditor` optional, only opening it when the user explicitly says so.

My `CustomDocument` is basically just the base image `Uri` the user is trying to compare, and a second image `Uri` to compare with.

# Communication to the `WebView` and back
It is important to understand that the webview part of the extension is separate from the extension itself and has to communicate trough `message` events.

Use this to send messages to the webview:
```typescript
webviewPanel.webview.postMessage({
    command: command,
    arguments: args,
});
```

Register this in your webview to receive messages:
```typescript
window.addEventListener('message', 
(event: MessageEvent<{ command: string, arguments: any}>) => {

});
```

To send messages from the webview to the extension you need a [`VsCodeApiWrapper`](https://github.com/paulheg/image-diff/blob/main/webview-ui/src/utilities/vscode.ts).


The basic lifecycle of my extension looks like this:

1. The user opens the custom editor with the base image already selected in the extension. The webview is now loading.
2. After the webview is fully loaded a custom `WEBVIEW_READY` message is sent to the extension. *(The webview does not know about the base image jet).*
3. The extension now knows that all event handlers are in place and can now send the first and if already there the second image `Uri`.
4. The webview will receive the messages and will set the images accordingly.

# Commands and Editors
A command can be triggered trough different ways:
- Pressing `Ctrl+Shift+P` and typing the command
- Using menu buttons, that are bound to the command

From this context it is not directly clear in which `CustomEditor` the command should be executed.


To solve this problem I assumed that the user is pressing the registered button of the current tab.
Therefore I did the following:
```typescript
const documentUri = (
    vscode.window.tabGroups.activeTabGroup.activeTab?.input as any
).uri as vscode.Uri;

const editor = ImageDiffEditorProvider.editors.get(documentUri.fsPath);
```
The `activeTab` has an `input` property that contains the base `Uri` of the currently opened file.
To get the editor, I created a `Map` from said `Uri` to my `CustomEditor`.
This `Map` is filled when the editor is resolved in the `resolveCustomEditor` function. 

# Create the webview

At first I tried to work with bare `html`, `ts` and `css` files.
This lasted until I wanted to try the [vscode-webview-ui-toolkit](https://github.com/microsoft/vscode-webview-ui-toolkit).
This had me switch to `React` as I wanted to learn it anyway.
As a basis I used the [vscode-webview-ui-toolkit-samples](https://github.com/microsoft/vscode-webview-ui-toolkit-samples/tree/main/frameworks/hello-world-react-cra) repository as well as this very good [Cheatsheet](https://github.com/typescript-cheatsheets/react).

Unfortunately does `React` not bundle your `.js` files into one per default. To make this work, use [build-react-no-split.js](https://github.com/microsoft/vscode-webview-ui-toolkit-samples/blob/main/frameworks/hello-world-react-cra/webview-ui/scripts/build-react-no-split.js) and set it up in your webview-`package.json`: 

```json
...
"scripts": {
    "build": "node ./scripts/build-react-no-split.js",
    ...
}
...
```

While creating the webview I had problems with the lifecycle of `React` and `undefined` `this` when calling class methods out of `html` events, as described [in this article](https://www.voitanos.io/blog/deal-with-undefined-this-react-event-handler-performant-way/).
On top of that I faced the usual struggle with `css`.


## Packaging and Publishing
To learn how to publish my working extension I read [this article](https://code.visualstudio.com/api/working-with-extensions/publishing-extension).
You should definitely use `esbuild` or another solution to package and minify your extension. This way you don't have to include the `node_modules` folder with all the libraries.

If you are not sure if you ignored all unnecessary files, you can just unpack the `.vsix` and have a look inside.
With this method I removed every unnecessary file to make a small and compact package.

# Fin
I hope this writeup contained some useful information for you.

Thank's for reading.