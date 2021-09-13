# PyStrategies: Deep Learning Framework for HFT Trading Strategy Development
高频交易策略发展的深度学习框架

PyStrategies is a collection of Python tools for implementing, testing, and optimizing algorithmic trading strategies. This framework is built to be used with depth-of-book US equities data. Tools are included for deep learning, parameter optimization, and high-fidelity single-stock backtesting.

PyStrategies是一组Python工具，用于实现、测试和优化算法交易策略。该框架是为配合美国股票账本深度数据而构建的。包括用于深度学习、参数优化和高保真单股回溯测试的工具。

This project relies on another one of my libraries: [PyLimitBook](https://github.com/danielktaylor/PyLimitBook)

The bottom of this README includes important setup instructions.

## Toolset

### Writing Strategies

Strategies are implemented by extending the [BaseStrategy](backtest/PyBridge/basestrategy.py) base class and adding logic for event callbacks. Refer to the [Sample Strategy](strategy/pyStrategy.py) for an (unprofitable) example.

策略是通过扩展BaseStrategy基类，和添加event callbacks逻辑来实现的。参考Sample Strategy中的(unprofitable)示例。

### Backtesting a Strategy

A single-stock backtester is included that accurately simulates the limit order book.

包括一个单股single-stock backtester，准确模拟limit order book。

To run a backtest:

        ./bin/backtest.sh quotes/XOM_BATS_2010-06-23.csv strategy/pyStrategy.py

### Graphing Results

A sample [Jupyter](http://jupyter.org/) notebook is included to analyze signals generated from your strategy.

一个样本Jupyter笔记本包括分析信号产生的策略。

To generate data and interactively graph it:

生成数据和交互式图表:

        # 生成book快照——确保在脚本中适当设置了CREATE_ON_NBBO_CHANGE_ONLY变量
        python PyLimitBook/create_graphing_data.py ./quotes/XOM_BATS_2010-06-23.csv ./analyze/data/book.csv

        # 移动由backtest生成的信号
        mv ./strategy/signals_log.csv ./analyze/data/signals.csv

        jupyter notebook --notebook-dir=./analyze

### Strategy Parameter Optimization 策略参数优化

[Optunity](https://github.com/claesenm/optunity) is used for multi-threaded parameter optimization. Parameter ranges must be defined in the [parameters.csv](optimize/parameters.csv) file.

opportunity用于多线程参数优化。参数范围必须在parameters.csv文件中定义。

To run the parameter optimizer for 1000 iterations:

        # 在optimize/parameters.csv中指定参数范围
        python optimize/optimize.py quotes/XOM_BATS_2010-06-23.csv strategy/pyStrategy.py 1000

### Machine Learning

Wrapper scripts are included to generate features and do deep learning using [Theano](https://github.com/Theano/Theano) and [Keras](https://keras.io/):

包括 Wrapper scripts 生成features 和使用Theano和Keras进行深度学习:

        # 生成 features
        python machine_learning/generate_features.py quotes/XOM_BATS_2010-06-23.csv

        # 生成 标签来预测
        python machine_learning/generate_labels.py quotes/XOM_BATS_2010-06-23.csv

        # Train the model
        python machine_learning/train_nn.py quotes/XOM_BATS_2010-06-23.csv

        # Test the model
        python machine_learning/test_nn.py quotes/XOM_BATS_2010-06-24.csv

## Setup Instructions 设置说明

### Install Python libraries 安装Python库

1. Setup virtualenv: ``virtualenv --no-site-packages venv``
2. Activate virtualenv: ``source venv/bin/activate``
3. Install libraries: ``pip install -r requirements.txt``

### Install jpy for bridging Python and Java code 安装用于桥接Python和Java代码的jpy

1. ``git clone https://github.com/bcdev/jpy.git``
2. ``cd jpy``
3. ``export JDK_HOME=`/usr/libexec/java_home` ``
4. ``export JAVA_HOME=$JDK_HOME``
5. ``python setup.py --maven build``
6. Copy lib/jpy-0.8.jar to the Java project's [lib directory](backtest/TradingFramework4j/lib)
7. Copy properties file from build/lib.*/jpyconfig.properties into the Java project's [strategy directory](strategy/jpyconfig.properties)

### Install CuDNN for Speeding up Deep Learning 安装加速深度学习的CuDNN

If your GPU supports CUDA, the machine learning with be much, much faster if you install CuDNN from NVIDIA.

如果你的GPU支持CUDA，那么如果你安装了NVIDIA的CuDNN，机器学习就会快得多。

1. Download CuDNN: https://developer.nvidia.com/rdp/cudnn-download
  * The following configuration works on Mac OS X:
    * nvcc version: release 7.5, V7.5.26
    * cuda driver: 7.5.27
    * clang version: clang-703.0.29 (XCODE 7.3.0)
    * osx version: 10.11.4
2. Install libraries:
  *  cd to where you want the libraries to live
  *  Linux: ``export LD_LIBRARY_PATH=`pwd`:$LD_LIBRARY_PATH``
  *  Mac: ``export DYLD_LIBRARY_PATH=`pwd`:$DYLD_LIBRARY_PATH``
       * Copied *.h files to CUDA_ROOT/include and *.so* files to CUDA_ROOT/lib64
           * By default, CUDA_ROOT is /usr/local/cuda on Linux and /Developer/NVIDIA/CUDA-* on mac
       * Add the following to the end of ~/.profile:

             export PATH="/Developer/NVIDIA/CUDA-7.5/bin:$PATH"
             export DYLD_LIBRARY_PATH="/Developer/NVIDIA/CUDA-7.5/lib:$DYLD_LIBRARY_PATH"

  *  Add the install path to your build and link process by adding -I`installpath` to your compile line and -L`installpath` -lcudnn to your link line.
3. Create a ~/.theanorc file: (adjust [cnmem](http://deeplearning.net/software/theano/library/config.html#config.config.lib.cnmem) as necessary)

        [global]
        device=gpu
        floatX=float32
        allow_gc=False
        warn_float64=warn

        [lib]
        cnmem=0.50

        [nvcc]
        fastmath=True
