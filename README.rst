MXNet-Gluon-SyncBN
==================
created by `Hang Zhang <http://hangzh.com/>`_

A preview tutorial for MXNet Gluon Synchronized Batch Normalization (SyncBN) [1]_ . We follow the sync-onece implmentation described in the paper [2]_ . If you are not familiar with Synchronized Batch Normalization, please see this `blog <http://hangzh.com/blog/SynchronizeBN/>`_. Special thanks to `Haibin <https://github.com/eric-haibin-lin>`_ for the technical support.

Jump to:

- `How to use SyncBN`_
- `MNIST example <https://github.com/zhanghang1989/MXNet-Gluon-SyncBN/blob/master/mnist.ipynb>`_
- `Load Pre-trained Network`_

Install MXNet from Source
-------------------------

* Please follow the MXNet docs to `install dependencies <http://mxnet.incubator.apache.org/install/index.html>`_
* Clone and install `syncbn branch <https://github.com/zhanghang1989/incubator-mxnet/tree/syncbn>`_

.. code:: bash

    # clone the branch
    git clone -b syncbatchnorm --recursive https://github.com/zhanghang1989/incubator-mxnet
    # compile mxnet
    cd incubator-mxnet && make -j $(nproc) USE_OPENCV=1 USE_BLAS=openblas USE_CUDA=1 USE_CUDA_PATH=/usr/local/cuda USE_CUDNN=1
    # install python API
    cd python && python setup.py install

How to use SyncBN
-----------------

``from syncbn import BatchNorm`` and  use ModelDataParallel with the network (input and output are both a list of NDArray). Everything else looks the same as before

.. code:: python

    import mxnet as mx
    from mxnet import gluon, autograd
    from mxnet.gluon import nn
    from mxnet.gluon.nn import Block
    # import SyncBN here
    from syncbn import BatchNorm, ModelDataParallel

    # create your own Block
    class Net(Block):
        def __init__(self):
            super(Net, self).__init__()
            self.conv = nn.Conv2D(in_channels=3, channels=10,
                                  kernel_size=3, padding=1)
            self.bn = BatchNorm(in_channels=10)
            self.relu = nn.Activation('relu')

        def forward(self, x):
            x = self.conv(x)
            x = self.bn(x)
            x = self.relu(x)
            return x

    # set the contexts (suppose using 4 GPUs)
    nGPUs = 4
    ctx_list = [mx.gpu(i) for i in range(nGPUs)]
    # get the model
    model = Net()
    model.initialize()
    model = ModelDataParallel(model, ctx_list)
    # load the data
    data = mx.random.uniform(-1,1,(8, 3, 24, 24))
    x = gluon.utils.split_and_load(data, ctx_list=ctx_list)
    with autograd.record():
        y = model(x)


MNIST Example
-------------

Please visit the `python notebook <https://github.com/zhanghang1989/MXNet-Gluon-SyncBN/blob/master/mnist.ipynb>`_

Load Pre-trained Network
------------------------

**TODO**

`常见问答 <https://github.com/zhanghang1989/MXNet-Gluon-SyncBN/blob/master/ChineseQA.md>`_
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Reference
---------

.. [1] Ioffe, Sergey, and Christian Szegedy. "Batch normalization: Accelerating deep network training by reducing internal covariate shift." *ICML 2015*

.. [2] Hang Zhang, Kristin Dana, Jianping Shi, Zhongyue Zhang, Xiaogang Wang, Ambrish Tyagi, and Amit Agrawal. "Context Encoding for Semantic Segmentation." *CVPR 2018*
