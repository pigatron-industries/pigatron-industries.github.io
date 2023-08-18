---
layout: post
title: "Running a Tensorflow Lite model on a Teeny microcontroller"
---
# Running a Tensorflow Lite model on a Teeny microcontroller


## Compiling

Basic instructions for building tflite from source can be found [here](https://github.com/tensorflow/tflite-micro/blob/main/tensorflow/lite/micro/docs/new_platform_support.md) 

For Teensy these are the steps that I needed to take to compile. These intructions worked from the main branch of tflite-micro in July 2023. tflite doesn't seem to have versioned releases so this may be different in the future. 

1. Clone the tflite-micro repository in [github](https://github.com/tensorflow/tflite-micro/tree/main)

2. As in the instructions above run the python script to create a directory tree containing the sources necessary to build:

```python tensorflow/lite/micro/tools/project_generation/create_tflm_tree.py tmp/tflm-tree
```

3. The created tree can be copied into the src folder of a platformio project to build it.

4. You will need to update the default tool chain to a version which includes std::to_string by adding the following line to platformio.ini:

```platform_packages = toolchain-gccarmnoneeabi @ 1.90201.191206
```

5. You will then need to provide an updated math .a library file from [here](https://github.com/ARM-software/CMSIS_4/blob/master/CMSIS/Lib/GCC/libarm_cortexM7lfsp_math.a). 
This can be copied into /lib folder in the project and will need adding to the build flags in platofromio.ini:

```build_flags = -L ./lib
```

6. Add the following build flag to platformio.ini:

```build_flags = -D ARDUINOSTL_M_H
```

6. tflite/tensorflow/lite/micro/kernels/lstm_eval_common.cc
Remove the double cast from line 192:
```  cell_state_info.quantized_cell_clip = static_cast<int16_t>(
    std::min(std::max(cell_clip / cell_state_scale, -32768.0), 32767.0));
```

7. tflite/third_party/gemmlowp/fixedpoint/fixedpoint.h
Add a double cast on line 519+:
```        std::max(static_cast<double>(round(x * static_cast<double>(1ll << kFractionalBits))),
                 min_bound),
        max_bound)));
```

8. tflite/tensorflow/lite/kernels/cppmath.h
Declare TfLiteRound directly instead of with macro expansion:
```// DECLARE_STD_GLOBAL_SWITCH1(TfLiteRound, round);
template <class T>                                
inline T TfLiteRound(const T x) {                    
    return round(x); 
}
```

9. tflite/tensorflow/lite/micro/debug_log.cc
Import Arduino.h and change fprintf to Serial.println:
```#import <Arduino.h>
```
```Serial.println(s);
```

10. It should now be possible to compile the source without errors. I've done this and checked the resulting code into the [xen_machinelearning](https://github.com/pigatron-industries/xen_machinelearning) repository.

This repository can be used as a library in other projects by adding it as a dependency in platformio.ini:

```lib_deps = https://github.com/pigatron-industries/xen_machinelearning.git
```

and also the following build flag:

```build_flags = -D ARDUINOSTL_M_H
```


