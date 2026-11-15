# code-oss

[![Build VSCode Web](https://github.com/YieldRay/code-oss/actions/workflows/build.yml/badge.svg)](https://github.com/YieldRay/code-oss/actions/workflows/build.yml)
[![GitHub Release](https://img.shields.io/github/v/release/YieldRay/code-oss)](https://github.com/YieldRay/code-oss/releases)
[![NPM](https://img.shields.io/npm/v/code-oss)](https://www.npmjs.com/package/code-oss)

This project produces a [VSCode OSS](https://github.com/microsoft/vscode) web build **without mangling or minification**.

To get the official build, please use [this link](https://update.code.visualstudio.com/api/update/web-standalone/stable/latest), see also [@vscode/test-web](https://github.com/microsoft/vscode-test-web/blob/main/src/server/download.ts).

For alternative build methods, see [@github1s/vscode-web](https://github.com/conwnet/github1s/tree/master/vscode-web), [Felx-B/vscode-web](https://github.com/Felx-B/vscode-web) and [progrium/vscode-web](https://github.com/progrium/vscode-web).

## Usage

Minimal Setup.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=no"
    />
    <link rel="stylesheet" href="https://unpkg.com/code-oss@latest/out/vs/workbench/workbench.web.main.internal.css" />
  </head>
  <body></body>
  <script type="module">
    const CDN_PREFIX = "https://unpkg.com/code-oss@latest";
    globalThis._VSCODE_FILE_ROOT = `${CDN_PREFIX}/out/`;
    await import(`${CDN_PREFIX}/out/nls.messages.js`);
    // You can implement `workbench.js` yourself; this project provides a convenient implementation for quick setup
    const { default: init } = await import(`${CDN_PREFIX}/workbench.js`);
    init();
  </script>
</html>
```

Use the convenient `workbench.js` for manual setup.  
Most of the time, you'll want to set up a custom built-in extension.  
For example, [this project](https://github.com/YieldRay/vscode-web-extension-memfs) sets up a memfs so all files can be stored in the browser memory.

```js
const CDN_PREFIX = "https://unpkg.com/code-oss@latest";
globalThis._VSCODE_FILE_ROOT = `${CDN_PREFIX}/out/`;
await import(`${CDN_PREFIX}/out/nls.messages.js`);
const { default: init } = await import(`${CDN_PREFIX}/workbench.js`);
const url = new URL(location.href);

init(document.body, {
  /** https://github.com/Microsoft/vscode/blob/main/product.json */
  productConfiguration: {
    nameShort: "Code - OSS",
    nameLong: "Code - OSS",
    applicationName: "code-oss",
    dataFolderName: ".vscode-oss",
    extensionsGallery: {
      serviceUrl: "https://open-vsx.org/vscode/gallery",
      itemUrl: "https://open-vsx.org/vscode/item",
      resourceUrlTemplate: "https://openvsxorg.blob.core.windows.net/resources/{publisher}/{name}/{version}/{path}",
    },
    extensionEnabledApiProposals: {},
    /** this config is required for `webWorkerExtensionHostIframe.html` to work */
    webEndpointUrlTemplate: CDN_PREFIX,
    /** this is also required */
    quality: "stable",
  },
  /**
   * optional:
   * this sets the default workspace folder URI
   */
  folderUri: {
    scheme: "my-file-scheme",
    authority: url.host,
    query: url.search,
    path: url.pathname,
  },
  /**
   * optional:
   * check out this project to see how to set up built-in extensions
   * https://github.com/YieldRay/vscode-web-extension-memfs
   */
  additionalBuiltinExtensions: [
    {
      scheme: url.protocol.replace(":", ""),
      authority: url.host,
      path: url.pathname + "path-to-extension",
    },
  ],
});
```
