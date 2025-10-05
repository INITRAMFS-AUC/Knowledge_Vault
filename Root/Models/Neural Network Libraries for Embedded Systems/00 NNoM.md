---
References:
  - https://github.com/majianjia/nnom?tab=readme-ov-file
  - https://majianjia.github.io/nnom/guide_5_min_to_nnom/
tags:
  - csce/machine_learning/models/libraries
---
## The structure of NNoM
- NNoM is a high-level inference Neural Network library specifically for microcontrollers.
![[Pasted image 20251003212416.png]]
- NNoM uses a layer-based structure. The most benefit is the model structure can seem directly from the codes.
- It also makes the model conversion from other layer-based libs (Keras, TensorLayer, Caffe) to NNoM model very straight forward. 
- When use `generate_model(model, x_test, name='weights.h')` to generate NNoM model, it simply read the configuration out and rewrite it to C codes.
![[Pasted image 20251003212955.png]]
- NNoM uses a compiler to manage the layer structure and other resources. After compiling, all layers inside the model will be put into a shortcut list per the running order. 
- Besides that, arguments will be filled in and the memory will be allocated to each layer (Memory are reused in between layers). Therefore, no memory allocation performed in the runtime, performance is the same as running backend function directly.
- The NNoM is more on managing the higher-level structure, context argument and memory. The actual arithmetics are done by the backend functions.
- Currently, NNoM supports a pure C backend and CMSIS-NN backend. The CMSIS-NN is a highly optimized low-level NN core for ARM-Cortex-M microcontroller. Please check the [optimization guide](https://majianjia.github.io/nnom/Porting_and_Optimisation_Guide/) for utilization.
## Why is NNoM different from others?

NNoM is a higher-level inference framework. The most obvious feature is the human understandable interface.
- It is also a layer-based framework, instead of operator-based. A layer might contain a few operators.
- It natively supports complex model structure. High-efficiency network always benefited from complex structure.
- It provides layer-to-layer analysis to help developer optimize their models.
## Available Operations

> *Notes: NNoM now supports both HWC and CHW formats. Some operation might not support both format currently. Please check the tables for the current status. *

**Core Layers**

|Layers|Struct API|Layer API|Comments|
|---|---|---|---|
|Convolution|conv2d_s()|Conv2D()|Support 1/2D, support dilations (New!)|
|ConvTransposed|conv2d_trans_s()|Conv2DTrans()|Under Dev. (New!)|
|Depthwise Conv|dwconv2d_s()|DW_Conv2D()|Support 1/2D|
|Fully-connected|dense_s()|Dense()||
|Lambda|lambda_s()|Lambda()|single input / single output anonymous operation|
|Batch Normalization|N/A|N/A|This layer is merged to the last Conv by the script|
|Flatten|flatten_s()|Flatten()||
|SoftMax|softmax_s()|SoftMax()|Softmax only has layer API|
|Activation|N/A|Activation()|A layer instance for activation|
|Input/Output|input_s()/output_s()|Input()/Output()||
|Up Sampling|upsample_s()|UpSample()||
|Zero Padding|zeropadding_s()|ZeroPadding()||
|Cropping|cropping_s()|Cropping()||

**RNN Layers**

|Layers|Status|Layer API|Comments|
|---|---|---|---|
|Recurrent NN|Under Dev.|RNN()|Under Developpment|
|Simple RNN|Under Dev.|SimpleCell()|Under Developpment|
|Gated Recurrent Network (GRU)|Under Dev.|GRUCell()|Under Developpment|

**Activations**

Activation can be used by itself as layer, or can be attached to the previous layer as ["actail"](https://majianjia.github.io/nnom/docs/A_Temporary_Guide_to_NNoM.md#addictionlly-activation-apis) to reduce memory cost.

There is no Struct API for activation currently, since activation are not usually used as a layer.

| Actrivation       | Struct API | Layer API | Activation API | Comments                            |
| ----------------- | ---------- | --------- | -------------- | ----------------------------------- |
| ReLU              | N/A        | ReLU()    | act_relu()     |                                     |
| Leaky ReLU (New!) | N/A        | ReLU()    | act_relu()     |                                     |
| Adv ReLU          | N/A        | N/A       | act_adv_relu() | advance ReLU, Slope, max, threshold |
| TanH              | N/A        | TanH()    | act_tanh()     |                                     |
| Sigmoid           | N/A        | Sigmoid() | act_sigmoid()  |                                     |

**Pooling Layers**

|Pooling|Struct API|Layer API|Comments|
|---|---|---|---|
|Max Pooling|maxpool_s()|MaxPool()||
|Average Pooling|avgpool_s()|AvgPool()||
|Sum Pooling|sumpool_s()|SumPool()||
|Global Max Pooling|global_maxpool_s()|GlobalMaxPool()||
|Global Average Pooling|global_avgpool_s()|GlobalAvgPool()||
|Global Sum Pooling|global_sumpool_s()|GlobalSumPool()|A better alternative to Global average pooling in MCU before Softmax|

**Matrix Operations Layers**

|Matrix|Struct API|Layer API|Comments|
|---|---|---|---|
|Concatenate|concat_s()|Concat()|Concatenate through any axis|
|Multiple|mult_s()|Mult()||
|Addition|add_s()|Add()||
|Substraction|sub_s()|Sub()||
## Dependencies

NNoM now use the local pure C backend implementation by default. Thus, there is no special dependency needed.
## Quantisation

NNoM currently only support 8 bit weights and 8 bit activations. The model will be quantised through model conversion `generate_model(model, x_test, name='weights.h')`.

The input data (activations) will need to be quantised then feed to the model.
## Performance

Performances vary from chip to chip. Efficiencies are more constant.

We can use _Multiply–accumulate operation (MAC) per Hz (MACops/Hz)_ to evaluate the efficiency. It simply means how many MAC can be done in one cycle.

Currently, NNoM only count MAC operations on Convolution layers and Dense layers since other layers (pooling, padding) are much lesser.
## Known Issues
### The Converter do not support implicitly defined activations

The script currently does not support implicit act:

```
x = Dense(32, activation="relu")(x)
```

Use the explicit activation instead.

```
x = Dense(32)(x)
x = Relu()(x)
```
## Evaluations

Evaluation is equally important to building the model.

In NNoM, we provide a few different methods to evaluate the model. The details are list in [Evaluation Methods](https://majianjia.github.io/nnom/api_nnom_utils/). If your system support print through a console (such as serial port), the evaluation can be printed on the console.

Firstly, the model structure is printed during compiling in `model_compile()`, which is normally called in `nnom_model_create()`.

Secondly, the runtime performance is printed by `model_stat()`.

Thirdly, there is a set of `prediction_*()` APIs to validate a set of testing data and print out Top-K accuracy, confusion matrix and other info.

### An NNoM model

This is what a typical model looks like in the `weights.h` or `model.h` or whatever you name it. These codes are generated by the script. In user's `main()`, call `nnom_model_create()` will create and compile the model.

```C
/* nnom model */
static int8_t nnom_input_data[784];
static int8_t nnom_output_data[10];
static nnom_model_t* nnom_model_create(void)
{
    static nnom_model_t model;
    nnom_layer_t* layer[20];

    new_model(&model);

    layer[0] = Input(shape(28, 28, 1), nnom_input_data);
    layer[1] = model.hook(Conv2D(12, kernel(3, 3), stride(1, 1), PADDING_SAME, &conv2d_1_w, &conv2d_1_b), layer[0]);
    layer[2] = model.active(act_relu(), layer[1]);
    layer[3] = model.hook(MaxPool(kernel(2, 2), stride(2, 2), PADDING_SAME), layer[2]);
    layer[4] = model.hook(Cropping(border(1,2,3,4)), layer[3]);
    layer[5] = model.hook(Conv2D(24, kernel(3, 3), stride(1, 1), PADDING_SAME, &conv2d_2_w, &conv2d_2_b), layer[4]);
    layer[6] = model.active(act_relu(), layer[5]);
    layer[7] = model.hook(MaxPool(kernel(4, 4), stride(4, 4), PADDING_SAME), layer[6]);
    layer[8] = model.hook(ZeroPadding(border(1,2,3,4)), layer[7]);
    layer[9] = model.hook(Conv2D(24, kernel(3, 3), stride(1, 1), PADDING_SAME, &conv2d_3_w, &conv2d_3_b), layer[8]);
    layer[10] = model.active(act_relu(), layer[9]);
    layer[11] = model.hook(UpSample(kernel(2, 2)), layer[10]);
    layer[12] = model.hook(Conv2D(48, kernel(3, 3), stride(1, 1), PADDING_SAME, &conv2d_4_w, &conv2d_4_b), layer[11]);
    layer[13] = model.active(act_relu(), layer[12]);
    layer[14] = model.hook(MaxPool(kernel(2, 2), stride(2, 2), PADDING_SAME), layer[13]);
    layer[15] = model.hook(Dense(64, &dense_1_w, &dense_1_b), layer[14]);
    layer[16] = model.active(act_relu(), layer[15]);
    layer[17] = model.hook(Dense(10, &dense_2_w, &dense_2_b), layer[16]);
    layer[18] = model.hook(Softmax(), layer[17]);
    layer[19] = model.hook(Output(shape(10,1,1), nnom_output_data), layer[18]);
    model_compile(&model, layer[0], layer[19]);
    return &model;
}
```

### Model info, memory

This is an example printed by `model_compile()`, which is normally called by `nnom_model_create()`.

```
Start compiling model...
Layer(#)         Activation    output shape    ops(MAC)   mem(in, out, buf)      mem blk lifetime
-------------------------------------------------------------------------------------------------
#1   Input      -          - (  28,  28,   1)          (   784,   784,     0)    1 - - -  - - - - 
#2   Conv2D     - ReLU     - (  28,  28,  12)      84k (   784,  9408,    36)    1 1 1 -  - - - - 
#3   MaxPool    -          - (  14,  14,  12)          (  9408,  2352,     0)    1 1 1 -  - - - - 
#4   Cropping   -          - (  11,   7,  12)          (  2352,   924,     0)    1 1 - -  - - - - 
#5   Conv2D     - ReLU     - (  11,   7,  24)     199k (   924,  1848,   432)    1 1 1 -  - - - - 
#6   MaxPool    -          - (   3,   2,  24)          (  1848,   144,     0)    1 1 1 -  - - - - 
#7   ZeroPad    -          - (   6,   9,  24)          (   144,  1296,     0)    1 1 - -  - - - - 
#8   Conv2D     - ReLU     - (   6,   9,  24)     279k (  1296,  1296,   864)    1 1 1 -  - - - - 
#9   UpSample   -          - (  12,  18,  24)          (  1296,  5184,     0)    1 - 1 -  - - - - 
#10  Conv2D     - ReLU     - (  12,  18,  48)    2.23M (  5184, 10368,   864)    1 1 1 -  - - - - 
#11  MaxPool    -          - (   6,   9,  48)          ( 10368,  2592,     0)    1 1 1 -  - - - - 
#12  Dense      - ReLU     - (  64,   1,   1)     165k (  2592,    64,  5184)    1 1 1 -  - - - - 
#13  Dense      -          - (  10,   1,   1)      640 (    64,    10,   128)    1 1 1 -  - - - - 
#14  Softmax    -          - (  10,   1,   1)          (    10,    10,     0)    1 1 - -  - - - - 
#15  Output     -          - (  10,   1,   1)          (    10,    10,     0)    1 - - -  - - - - 
-------------------------------------------------------------------------------------------------
Memory cost by each block:
 blk_0:5184  blk_1:2592  blk_2:10368  blk_3:0  blk_4:0  blk_5:0  blk_6:0  blk_7:0  
 Total memory cost by network buffers: 18144 bytes
Compling done in 179 ms

```

It shows the run order, Layer names, activations, the output shape of the layer, the operation counts, the buffer size, and the memory block assignments.

Later, it prints the maximum memory cost for each memory block. Since the memory block is shared between layers, the model only uses 3 memory blocks, altogether gives a sum memory cost by `18144 Bytes`.

### Runtime statistices

This is an example printed by `model_stat()`.

> This method requires a microsecond timestamp porting, check [porting guide](https://majianjia.github.io/nnom/Porting_and_Optimisation_Guide/)

```
Print running stat..
Layer(#)        -   Time(us)     ops(MACs)   ops/us 
--------------------------------------------------------
#1        Input -        11                  
#2       Conv2D -      5848          84k     14.47
#3      MaxPool -       698                  
#4     Cropping -        16                  
#5       Conv2D -      3367         199k     59.27
#6      MaxPool -       346                  
#7      ZeroPad -        36                  
#8       Conv2D -      4400         279k     63.62
#9     UpSample -       116                  
#10      Conv2D -     33563        2.23M     66.72
#11     MaxPool -      2137                  
#12       Dense -      2881         165k     57.58
#13       Dense -        16          640     40.00
#14     Softmax -         3                  
#15      Output -         1                  

Summary:
Total ops (MAC): 2970208(2.97M)
Prediction time :53439us
Efficiency 55.58 ops/us
NNOM: Total Mem: 20236

```

Calling this method will print out the time cost for each layer, and the efficiency in (MACops/us) of this layer.

This is very important when designing your ad-hoc model.

## Others

### Memeory management in NNoM

As mention, NNoM will allocate memory to the layer during the compiling phase. Memory block is a minimum unit for a layer to apply. For example, convolution layers normally apply one block for input data, one block for output data and one block for the intermediate data buffer.

```
Layer(#)         Activation    output shape    ops(MAC)   mem(in, out, buf)      mem blk lifetime
-------------------------------------------------------------------------------------------------
#2   Conv2D     - ReLU     - (  28,  28,  12)      84k (   784,  9408,    36)    1 1 1 -  - - - - 
```

The example shows input buffer size `784`, output buffer size `9408`, intermediate buffer size `36`. The following `mem blk lifetime` means how long does the memory block last. All three block last only one step, they will be freed after the layer. In NNoM, the output memory will be pass directly to the next layer(s) as input buffer, so there is no memory copy cost and memory allocation in between layers.

--- 
## Example 
### Neural Network with Keras

Lets say if we want to classify the MNIST hand writing dataset. This is what you normally do with Keras.
```
model = Sequential()
model.add(Dense(32, input_dim=784))
model.add(Activation('relu'))
model.add(Dense(10))
```
Each operation in Keras are defined by "Layer", same as we did in NNoM. The terms are different from Tensorflow.
### Deploy using NNoM

After the `model` is trained, the weights and parameters are already functional. We can now convert it to C language files then put it in your MCU project.

> The result of this step is a single `weights.h` file, which contains everything you need.

To convert the model, NNoM has provided an simple API `generate_model()`[API](https://majianjia.github.io/nnom/api_nnom_utils/) to automatically do the job. Simply pass the `model` and the test dataset to it. It will do all the magics for you.

```python
generate_model(model, x_test, name='weights.h')
```

When the conversion is finished, you will find a new `weights.h` under your working folder. Simply copy the file to your MCU project, and call `model = nnom_model_create();` inside you `main()`.

Below is what you should do in practice.

```c
#include "nnom.h"
#include "weights.h"

int main(void)
{
    nnom_model_t *model;

    model = nnom_model_create();
    model_run(model);
}
```

Then, your model is now running on you MCU. If you have supported `printf` on your MCU, you should see the compiling info on your consoles.

Compiling logging similar to this:

```
Start compiling model...
Layer(#)         Activation    output shape    ops(MAC)   mem(in, out, buf)      mem blk lifetime
-------------------------------------------------------------------------------------------------
#1   Input      -          - (  28,  28,   1)          (   784,   784,     0)    1 - - -  - - - - 
#2   Conv2D     - ReLU     - (  28,  28,  12)      84k (   784,  9408,    36)    1 1 3 -  - - - - 
#3   MaxPool    -          - (  14,  14,  12)          (  9408,  2352,     0)    1 2 3 -  - - - - 
#4   UpSample   -          - (  28,  28,  12)          (  2352,  9408,     0)    1 2 2 -  - - - - 
#5   Conv2D     -          - (  14,  14,  12)     254k (  2352,  2352,   432)    1 1 2 1  1 - - - 
#6   Conv2D     -          - (  28,  28,  12)    1.01M (  9408,  9408,   432)    1 1 2 1  1 - - - 
#7   Add        -          - (  28,  28,  12)          (  9408,  9408,     0)    1 1 1 1  1 - - - 
#8   MaxPool    -          - (  14,  14,  12)          (  9408,  2352,     0)    1 1 1 2  1 - - - 
#9   Conv2D     -          - (  14,  14,  12)     254k (  2352,  2352,   432)    1 1 1 2  1 - - - 
#10  AvgPool    -          - (   7,   7,  12)          (  2352,   588,   168)    1 1 1 1  1 1 - - 
#11  AvgPool    -          - (  14,  14,  12)          (  9408,  2352,   336)    1 1 1 1  1 1 - - 
#12  Add        -          - (  14,  14,  12)          (  2352,  2352,     0)    1 1 - 1  1 1 - - 
#13  MaxPool    -          - (   7,   7,  12)          (  2352,   588,     0)    1 1 1 2  - 1 - - 
#14  UpSample   -          - (  14,  14,  12)          (   588,  2352,     0)    1 1 - 2  - 1 - - 
#15  Add        -          - (  14,  14,  12)          (  2352,  2352,     0)    1 1 1 1  - 1 - - 
#16  MaxPool    -          - (   7,   7,  12)          (  2352,   588,     0)    1 1 1 1  - 1 - - 
#17  Conv2D     -          - (   7,   7,  12)      63k (   588,   588,   432)    1 1 1 1  - 1 - - 
#18  Add        -          - (   7,   7,  12)          (   588,   588,     0)    1 1 1 -  - 1 - - 
#19  Concat     -          - (   7,   7,  24)          (  1176,  1176,     0)    1 1 1 -  - - - - 
#20  Dense      - ReLU     - (  96,   1,   1)     112k (  1176,    96,  2352)    1 1 1 -  - - - - 
#21  Dense      -          - (  10,   1,   1)      960 (    96,    10,   192)    1 1 1 -  - - - - 
#22  Softmax    -          - (  10,   1,   1)          (    10,    10,     0)    1 - 1 -  - - - - 
#23  Output     -          - (  10,   1,   1)          (    10,    10,     0)    1 - - -  - - - - 
-------------------------------------------------------------------------------------------------
Memory cost by each block:
 blk_0:9408  blk_1:9408  blk_2:9408  blk_3:9408  blk_4:2352  blk_5:588  blk_6:0  blk_7:0  
 Total memory cost by network buffers: 40572 bytes
Compling done in 76 ms

```

You can now use the model to predict your data.

- Firstly, filling the input buffer `nnom_input_buffer[]` with your own data(image, signals) which is defined in `weights.h`.
- Secondly, call `model_run(model);` to do your prediction.
- Thirdly, read your result from `nnom_output_buffer[]`. The maximum number is the results.

Now, please do check NNoM examples for more fancy methods.

Next [[01 ExecuTorch]]