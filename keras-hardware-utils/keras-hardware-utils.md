# 多 GPU 下的 Keras

* 在安装 tensorflow 时使用

  ```text
   pip3 install tensorflow-gpu
   `or`
   python3.6 -m pip install tensorflow-gpu
  ```

  这样在 Keras 中用 tensorflow backend 时就会使用自动使用 GPU。

* 在不做任何设置时， keras 中的 tensorflow 会占用所有 GPU 的内存，但是只会使用一个 GPU 来进行计算。运行

  ```text
     nvidia-smi
  ```

  结果如下图所示  
  !\[捕获\]  
  其中 GPU-Util 是 GPU 计算负荷，而 GPU Memory Usage 则是内存用量。

* 使用 multi\_gpu\_model 可以设定多个 GPU 同时计算

  首先构造出一个普通的 model，一般在 CPU 中构造

  ```python
    with tf.device("/cpu:0"):
          model = Sequential()
          model.add(Dense(NB_CLASSES, input_shape=(RESHAPED,)))
          model.add(Activation("softmax"))
          model.summary()
  ```

  然后将 model 转化为多 GPU 模型并训练

```python
    from keras.utils import multi_gpu_model

    p_model = multi_gpu_model(model, gpus=4)
    p_model.compile(
        loss="categorical_crossentropy", optimizer=OPTIMIZER, metrics=["accuracy"]
    )
    history = p_model.fit(
        X_train,
        Y_train,
        batch_size=BATCH_SIZE,
        epochs=NB_EPOCH,
        verbose=VERBOSE,
        validation_split=VALIDATION_SPLIT,
    )
    score = p_model.evaluate(X_test, Y_test, verbose=VERBOSE)
```

**注意一般较小模型使用多 GPU 处理有可能会使得消耗时间增加**

* 对 tensorflow 的 session 进行设置以控制 GPU 内存占用量

```python
import os
import tensorflow as tf
import keras.backend.tensorflow_backend as KTF


def get_session(gpu_fraction=0.3):
    num_threads = os.environ.get("OMP_NUM_THREADS")
    gpu_options = tf.GPUOptions(per_process_gpu_memory_fraction=gpu_fraction)
    if num_threads:
        return tf.Session(
            config=tf.ConfigProto(
                gpu_options=gpu_options, intra_op_parallelism_threads=num_threads
            )
        )
    else:
        return tf.Session(config=tf.ConfigProto(gpu_options=gpu_options))

KTF.set_session(get_session())
```

* 参考 [keras 多 GPU 训练指南](https://yq.aliyun.com/articles/230182)，[tensorflow-gpu 安装](https://www.tensorflow.org/install/install_linux?hl=zh-cn#InstallingNativePip)

  \[捕获\]: images/-.PNG "捕获" { width:auto; max-width:90% }

