<!--- Licensed to the Apache Software Foundation (ASF) under one -->
<!--- or more contributor license agreements.  See the NOTICE file -->
<!--- distributed with this work for additional information -->
<!--- regarding copyright ownership.  The ASF licenses this file -->
<!--- to you under the Apache License, Version 2.0 (the -->
<!--- "License"); you may not use this file except in compliance -->
<!--- with the License.  You may obtain a copy of the License at -->

<!---   http://www.apache.org/licenses/LICENSE-2.0 -->

<!--- Unless required by applicable law or agreed to in writing, -->
<!--- software distributed under the License is distributed on an -->
<!--- "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY -->
<!--- KIND, either express or implied.  See the License for the -->
<!--- specific language governing permissions and limitations -->
<!--- under the License. -->

# TVM WebAssembly Runtime

This folder contains TVM WebAssembly Runtime.

## Installation

The LLVM main branch support webassembly as a target, we can directly
build TVM with LLVM mainline to generate wasm modules.
Note that, however, we still need emscripten to compile the runtime and provide system library support.

Note that so far we requires everything to be in the source and setup PYTHONPATH(instead of use setup.py install).

### Setup Emscripten

We use emscripten to compile our runtime wasm library as well as a WASI variant that we can deploy
to the browser environment.

Follow [Emscripten](https://emscripten.org/) to download emsdk and install emcc on your local environment.

### Build TVM Wasm Runtime

After the emcc is setup correctly. We can build tvm's wasm runtime by typing `make` in the web folder.

```bash
make
```

This command will create the follow files:
- `dist/wasm/libtvm_runtime.bc` bitcode library `tvm.contrib.emcc` will link into.
- `dist/wasm/tvmjs_runtime.wasm` a standalone wasm runtime for testing purposes.
- `dist/wasm/tvmjs_runtime.wasi.js` a WASI compatible library generated by emscripten that can be fed into runtime.


### Build TVM Wasm JS Frontend

Type the following command in the web folder.

```bash
npm run bundle
```

This command will create the tvmjs library that we can use to interface with the wasm runtime.


## Use TVM to Generate Wasm Library and Run it

Check code snippet in

- [tests/python/prepare_test_libs.py](https://github.com/apache/incubator-tvm/tree/master/web/tests/pythob/prepare_test_libs.py)
  shows how to create a wasm library that links with tvm runtime.
  - Note that all wasm libraries have to created using the `--system-lib` option
  - emcc.create_wasm will automatically link the runtime library `dist/wasm/libtvm_runtime.bc`
- [tests/web/test_module_load.js](https://github.com/apache/incubator-tvm/tree/master/web/tests/node/test_module_load.js) demonstrate
  how to run the generated library through tvmjs API.


## Run Wasm Remotely through WebSocket RPC.

We can now use js side to start an RPC server and connect to it from python side,
making the testing flow easier.

The following is an example to reproduce this.
- run `python -m tvm.exec.rpc_proxy --example-rpc=1` to start proxy.
- Start the WebSocket RPC
  - Browswer version:  open https://localhost:8888, click connect to proxy
  - NodeJS version: `npm run rpc`
- run `python tests/node/websock_rpc_test.py` to run the rpc client.


## WebGPU Experiments

Web gpu is still experimental, so apis can change.
Right now we use the SPIRV to generate shaders that can be accepted by Chrome and Firefox.

- Obtain a browser that support webgpu.
  - So far only Chrome Canary on MacOS works
  - Firefox should be close pending the support of Fence.
- Download vulkan SDK (1.1 or higher) that supports SPIRV 1.3
- Start the WebSocket RPC
- run `python tests/node/webgpu_rpc_test.py`