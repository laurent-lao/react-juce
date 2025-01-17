# Debugger Support

(Because console.log() isn't always enough)

Note, there are known issues debugging on Windows. See [Known Issues](#known-issues)

JS debugging support for React-JUCE/Duktape is available via [Visual Studio Code](https://code.visualstudio.com/) using the [Duktape Debugger](https://marketplace.visualstudio.com/items?itemName=HaroldBrenes.duk-debug) extension.
Currently this is the only debug client known to support the use of source maps with Duktape.
If you do become aware of other Duktape debug clients that may work well with React-JUCE, please open an issue with details for us to investigate.

React-JUCE provides an implementation of the Duktape debugging interface via `EcmascriptEngine::debuggerAttach` and `EcmascriptEngine::debuggerDetach`.

If you are using the `ReactApplicationRoot` class to host your React app/ui then you can enjoy a debugging experience similar to that of React Native.

In debug builds with `JUCE_DEBUG` defined, ensure your editor has focus and use the `CTRL-d/CMD-d` command to trigger a call to `EcmascriptEngine::debuggerAttach`.
This command blocks the UI and awaits connection from a debug client. You can then attach your debugger, continue execution and set breakpoints etc.

To configure the Duktape debugger extension for use with React-JUCE, you will need to do the following:

- Download/Add the Duktape debugger extension.

- Ensure you have compiled React-JUCE and your application/plugin in debug mode (with `JUCE_DEBUG` defined).

- Add a launch configuration for debugging your UI.

An example launch configuration for use with the `GainPlugin` example project is provided below.
Note that `localRoot` points to the root `workspaceFolder`. In this case we assume `workspaceFolder` to be `examples/GainPlugin/jsui`.
The `outDir` entry points to the build directory containing the compiled JS bundle you wish to debug.
This `outDir` directory should also contain the bundle's associated source map.

```
{
  "version": "0.2.0",
  "configurations": [
      {
          "name": "JSUI Debug Attach",
          "type": "duk",
          "request": "attach",
          "address": "localhost",
          "port": 9091,
          "localRoot": "${workspaceFolder}",
          "sourceMaps": true,
          "outDir": "${workspaceFolder}/build/js",
          "stopOnEntry": false,
          "debugLog": false
      }
   ]
}
```

To ensure debugging works reliably, you need to ensure your source map has been generated by webpack using [devtool](https://webpack.js.org/configuration/devtool/).
The following option should be added to the `output` key in your webpack config:

`devtoolModuleFilenameTemplate: 'webpack:///[absolute-resource-path]'`

You also need to ensure that your bundle is not minified, this can be achieved by building your JS bundle with `webpack --mode=development`.
In the case of the `GainPlugin` example this can be achieved by running `npm run start` in `GainPlugin/jsui`.

The `webpack.config.js` template provided by React-JUCE enables all of this for you. See: [Starting a new Project](Integrating_Your_Project.md) for a project
template setup which includes debugger support. Project template files are available under `react-juce/packages/react-juce/template`.

Example `webpack.config.js`:

```
module.exports = {
  entry: './src/index.js',
  // This is the section you care about for debugging.
  output: {
    path: __dirname + '/build/js',
    filename: 'main.js',
    sourceMapFilename: "[file].map",
    devtoolModuleFilenameTemplate: 'webpack:///[absolute-resource-path]'
  },
  devtool: "source-map",
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: ['babel-loader']
      },
      {
        test: /\.svg$/,
        exclude: /node_modules/,
        use: ['svg-inline-loader']
      },
      {
        test: /\.(png|jpeg|jpg|gif)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: true,
              esModule: false
            }
          }
        ]
      }
    ]
  },
}
```

To test debugging support using the `GainPlugin` example do the following:

- Open the `GainPlugin/jsui` directory in Visual Studio Code.

- Add the `launch.json` configuration from above to your `.vscode` directory under `GainPlugin/jsui`.

- Run `npm run start` in the `jsui` directory

- Start the standalone plugin app in your IDE (i.e. Visual Studio/Xcode)

- Give the `GainPlugin`'s UI/PluginEditor focus and hit `CTRL-d` (Win) / `CMD-d` (Mac)

- Start the `JSUI Debug Attach` launch task from the VSCode debugger menu.

- Set some breakpoints, profit!

## Known Issues

- Debugging support is currently experimental and issues may occur from time to time whilst we refine the experience.

- At present there are some issues debugging duktape under Windows. The vscode-duktape-debug extension requires a patch to resolve source map entries, without this patch setting breakpoints in source files fails. To obtain a version of the extension with this patch you can fetch https://github.com/JoshMarler/vscode-duktape-debug/tree/topic/fix-windows-debug-blueprint. To use this version of the extension run `npm install` in the `vscode-duktape-debug` root followed by `npm run compile`. You can then copy the contents of `vscode-duktape-debug/dist` to your vscode extensions directory which is usually under `.vscode` in your home folder. Copy the contents of `vscode-duktape-debug/dist` to `.vscode/extensions/haroldbrenes.duk-debug-0.5.6`. There are issues attaching the debugger with breakpoints already set in vscode on Windows. You will need to attach the debugger and then set breakpoints once attached.

## Acknowledgements

Thanks to [harold-b](https://github.com/harold-b/vscode-duktape-debug) for his excellent extension.
