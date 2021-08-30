# Enhanced Ensemble model

## Usage
### General
The general purpose of this mod is to allow ensemble model to read 
from output tensor data from submodel and use the data in the tensor
to decide which submodel will be the next one.

To enable this function, **dynamic_enabled** must be set in config.pbtxt.

### control_output_map
**control_output_map** is the key element in dynamic ensemble mode, it should
always be an TYPE_STRING tensor with shape [ 1 ]. The string inside its tensor
data indicates the next step in ensemble_scheduling map.

The key of control_output_map maps to submodel's tensor just like ordinary 
output_map. The value is the tensor being controlled.\

### none
**none** is a special value for control tensor, if none is detected, the controlled
tensor will be thrown without going to any next input.

### example
```
{
    model_name: "selection"
    model_version: -1
    input_map {
        key: "TENSOR_INPUT"
        value: "preprocessed_image"
    }
    control_output_map {
        key: "SHORTCUT_CONTROL" # SHORTCUT_CONTROL is an control tensor
        value: "SHORTCUT_OUTPUT" # the value in SHORTCUT_CONTROL is where the next input SHORTCUT_OUTPUT will go to
    }
    output_map {
        key: "SHORTCUT_OUTPUT" # this output is controlled by SHORTCUT_CONTROL
        value: "branch_a @@ branch_b @@ none " # the value will not be used since SHORTCUT_OUTPUT is already controlled
    }
    output_map {
        key: "NORMAL_OUTPUT" # a normal output
        value: "next_input" # this output will always be sent to next_input
    } 
}
```

It can be noticed that the value field of an contolled output will not be used. It recommended to
fill it with possible branches it may go to for debug-friendly.

## Build
These options are added into build.py:
1. no-backend-build: skips the process to build backend from source
2. no-agent-build: skips the process to build model agent from source
3. custom-repo: customized git url for given repo name

These optimizations have been done:
1. [ONNX] Missing ORT path no longer throws exception when building. 
The program just skips the backend instead.

### Example build command
```
python3 build.py 
        --target-platform=jetpack 
        --cmake-dir=`pwd`/build 
        --build-dir=`pwd`/builddir 
        --no-container-build 
        --build-type=Debug -j`nproc` 
        --enable-logging 
        --enable-stats 
        --enable-tracing 
        --enable-metrics  
        --enable-gpu 
        --endpoint=http 
        --endpoint=grpc 
        --repo-tag=common:r21.06-mod 
        --repo-tag=core:r21.06 
        --repo-tag=backend:r21.06 
        --repo-tag=thirdparty:r21.06 
        --backend=ensemble  
        --backend=onnxruntime:r21.06 
        --backend=python:r21.06 
        --repoagent=checksum:r21.06 
        --custom-repo=common:https://github.com/myrepo 
        --no-backend-build 
        --no-agent-build
```



