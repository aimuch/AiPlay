This is a multilayer perceptron(MLP) based Neural Collaborative Filter Recommender example showing how to generate weights for MovieLens dataset for TensorRT that TensorRT can accelerate.
This sample requires Tensorflow <= 1.7.0 to be installed.
This MLP base NCF was trained via the following method:

Building the sample:
To build the sample:
cd <TensorRT Install>/samples
make -j12

To run the sample:

1. Running Inference without MPS:

cd <TensorRT Install>/bin
./sample_movielens_mps           (default batch=32 i.e. num of users)
./sample_movielens_mps -b <N>    (batch=N i.e. num of users)
./sample_movielens_mps --verbose (prints inputs, groundtruth values, expected vs predicted probabilities)

2. Running Inference with MPS:
    a. MPS 101 (https://docs.nvidia.com/deploy/mps/index.html)
        Server:
            export CUDA_VISIBLE_DEVICES=0
            export CUDA_MPS_PIPE_DIRECTORY=/tmp/nvidia-mps # Select a location that's accessible to the given $UID
            export CUDA_MPS_LOG_DIRECTORY=/tmp/nvidia-log # Select a location that's accessible to the given $UID
            nvidia-cuda-mps-control -d # Start the daemon.
        Client:
            Set the following variables in the client process's environment. Note that CUDA_VISIBLE_DEVICES should not be set in the client's environment.
            export CUDA_MPS_PIPE_DIRECTORY=/tmp/nvidia-mps # Set to the same location as the MPS control daemon
            export CUDA_MPS_LOG_DIRECTORY=/tmp/nvidia-log # Set to the same location as the MPS control daemon
        Log Files
            $CUDA_MPS_LOG_DIRECTORY/control.log
            $CUDA_MPS_LOG_DIRECTORY/server.log

    b. Run the sample from as client
        cd <TensorRT Install>/bin
        ./sample_movielens_mps  (default batch=32 i.e. num of users, Number of processes=1)
        ./sample_movielens_mps -b <bSize> -p <nbProc> (bSize=Batch size i.e. num of users, nbProc=Number of processes)
        ./sample_movielens_mps --verbose (prints inputs, groundtruth values, expected vs predicted probabilities)

    c. Shutting Down MPS
        echo quit | nvidia-cuda-mps-control

3. Help/Usage
    ./sample_movielens_mps -h
    Usage:
        ./sample_movielens_mps [-h]
        -h        Display help information. All single dash optoins enable perf mode.
        -b        Number of Users i.e. BatchSize (default BatchSize=32).
        -p        Number of Processes (default threads=1).
        --useDLA  Enables use of DLA engine for layers that support DLA.
        --verbose Enable verbose perf mode.

Training model from scratch:
Step 1:
    git clone https://github.com/hexiangnan/neural_collaborative_filtering.git
    cd neural_collaborative_filtering
    git checkout 0cd2681598507f1cc26d110083327069963f4433

Step 2:
    Apply the patch file, `sampleMovieLensTraining.patch` to save dump the frozen protobuf file with command `patch -l -p1 < <TensorRT Install>/samples/sampleMovieLens/sampleMovieLensTraining.patch`
    Train the MLP based NCF with the command `python MLP.py --dataset ml-1m --epochs 20 --batch_size 256 --layers [64,32,16,8] --reg_layers [0.01,0.01,0.01,0.01] --num_neg 4 --lr 0.001 --learner adam --verbose 1 --out 1'
    WARNING: Using 0s for reg_layers will cause undefined behavior when training the network.
    This step will dump two files:
        1. movielens_ratings.txt
        2. sampleMovieLens.pb

Step 3:
    Convert the Frozen .pb file to .uff format using
    python3 convert_to_uff.py sampleMovieLens.pb -t
    Note: convert_to_uff.py utility will get installed here: /usr/local/bin/convert-to-uff.
        This utility gets installed with UFF .whl file installation shipped with TensorRT.
        For installation instructions, see:
        https://docs.nvidia.com/deeplearning/sdk/tensorrt-api/#python and click on the 'TensoRT Python API' link

Step 4:
    Copy sampleMovieLens.uff file to <TensorRT Install>/data/movielens
    Copy movielens_ratings.txt file to <TensorRT Install>/data/movielens

Step 5:
    Follow instruction above to build and run the sample
