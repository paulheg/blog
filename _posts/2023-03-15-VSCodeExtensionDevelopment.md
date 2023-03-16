---
layout: post
title: "What I learned writing a VS Code Webview extension"
---

# TL;DR

I created an extension to compare images in VS Code:

[**Image Diff**](https://marketplace.visualstudio.com/items?itemName=paulheg.image-diff)

# How it all started

This semester I took part in the computer graphics course at my university.
Part of the course was a mandatory exercise certificate, or as we like to say in German: *Übungsschein*.
To pass the Übungsschein you had to solve various programming tasks ranging from implementing ray tracing
to rasterization with OpenGL.
All tasks had one thing in common: to check if your implementation was correct,
there always were pictures with the expected result along with the tasks.
You could then compare these images with the ones produced by your code.

At first I compared them side by side.
For some tasks this wasn't possible.
For example, Gaussian blur, it certainly looked blurry, but was it calculated correctly?
There was just no way to tell the difference with your eyes alone.

Then I switched to using an online tool: [Diffchecker](https://www.diffchecker.com/) recommended by a friend of mine.

But after a while I got tired of uploading the pictures to this site.

> There should be a VS code extension for this.

Unfortunately, there was none that met my requirements.

What it should do:

- Calculate and display the difference between two images.
- Easy file selection, if there are images with the same name in another folder in the workspace, show them first when selecting.
- Zoom, to inspect even small changes

So it was decided, I had to create my own extension for this.

What follows is a rundown of all the things and I learned and all the obstacles I encountered.
If you are interested in developing your own extension you might find this an interesting read.

# Every beginning is hard

Until then, I had no experience writing VS Code extensions other than [watching Ben Awad create stories for VS Code](https://www.youtube.com/watch?v=ApR-kNXxLUs) and reporting bugs on GitHub for extensions I was using.

I started reading the [default documentation](https://code.visualstudio.com/api/get-started/your-first-extension) and generated my first extension using `yo code`.

I already knew it was possible to display images, since there is a [default image viewer](https://github.com/microsoft/vscode/tree/main/extensions/media-preview) included with VS Code.
After looking at the source code of *Diffchecker* I could just make use of the CSS property: ` mix-blend-mode: difference;`.
To do this I had to use [WebViews](https://code.visualstudio.com/api/extension-guides/webview) and [CustomEditors](https://code.visualstudio.com/api/extension-guides/custom-editors).

I did not want to override the default behavior of displaying images so I decided to make my `CustomEditor` optional, opening it only when the user explicitly says so.

My `CustomDocument` is basically just the base image `Uri` the user wants to compare, and a second image `Uri` to compare with.

# Communication to the `WebView` and back
It is important to understand that the webview part of the extension is separate from the extension itself and has to communicate through `message` events.

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


The basic life cycle of my extension is as follows:

1. The user opens the custom editor with the base image already selected in the extension. The webview is now loading.
2. After the webview is fully loaded a custom `WEBVIEW_READY` message is sent to the extension. *(The webview does not know about the base image jet).*
3. The extension now knows that all event handlers are in place and can now send the first and if already there the second image `Uri`.
4. The webview receives the messages and sets the images accordingly.

# Commands and Editors
A command can be triggered in several ways:
- Pressing `Ctrl+Shift+P` and typing the command
- Using menu buttons, that are bound to the command

From this context it is not immediately clear in which `CustomEditor` the command should be executed.


To solve this problem I assumed that the user presses the registered button of the current tab.
So I did the following:
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

Unfortunately `React` does not bundle your `.js` files into one file per default. To make this work, use the [build-react-no-split.js](https://github.com/microsoft/vscode-webview-ui-toolkit-samples/blob/main/frameworks/hello-world-react-cra/webview-ui/scripts/build-react-no-split.js) script and set it up in your webview-`package.json`: 

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

To check if your extension works before publishing it, 
you can just install the package yourself running the `Install from VSIX` command.

# Fin
I hope this writeup contained some useful information for you.

Thank's for reading.