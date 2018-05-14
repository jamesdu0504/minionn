# Requirements
```bash
sudo apt install libssl-dev libgmp-dev libglib2.0-dev
pip install pybind11 cppimport
```

# Installation


## ABY
Make sure you initialized gits submodules recursively:
```bash
git submodule update --init --recursive
```
Then, run make all in the lib folder:
```bash
cd lib
make all
```

## SEAL
Download SEAL (MiniONN is developed for Seal v2.3.0-4):
https://www.microsoft.com/en-us/research/project/simple-encrypted-arithmetic-library/
Extract SEAL into the libs/SEAL directory and build it with the -fPIC flag:
```bash
cd SEAL
export CXX="g++ -fPIC"
./configure
make
```

## Veryfying the installation
There are three test files in the lib subdirectory: test.py, test_mpc_server.py and test_mpc_client.py.

Run
```bash
python3 test.py
```
and verify that all tests passed. If you encountered errors during the initial compilation of the C++ modules, there are probably some files missing for SEAL or ABY.

Next, run the test_mpc_server.py and then the test_mpc_client.py in a second terminal.

# Usage
There are three models given in the models folder. All three take the S.tensor as input:
Server: python3 server.py -i models/S.onnx
Client: python3 client.py -i models/S.tensor -o models.out.txt

You can test and verify the correctness of the three models with the given scripts in the models folder.
```bash
cd models
python3 check_s.py
```

You can additionally build your own models with the scripts in the tools folder. Those also have their own test scripts included.

# MiniONN input
The MiniONN client requires only the model input to be given as a ONNX Tensor. The example models and tool model creators already create a TensorProto file automatically. However, for your own models you will need to export your data into a TensorProto and store that as a file. An example of how to do this process is in tools/csv_to_tensor.py .
 
# MiniONN inaccuracy
MiniONN introduces some inaccuracy into the calculated result. This can best be seen when running the above usage example and taking a look at the Difference between the expected and given result of the check_s.py file.
In our tests, this error did not change any predicted result. However, keep this error in mind whenever you experiment.

In future, this slight error might be resolved by changing the randomness of V as V gets downshifted after every matrix multiplication.

This downshift is also important for the scaling that is set in common/config.py . Here, the fractional_base is used to shift the input (weight and client input) up with the fractional base, and to scale the result down again after every matrix multiplication. If your input does not need to be scaled up (into integer range), then you can set the fractional_downscale to 1 for your models. However, you might then need to set the downscale factor in the same configuration file to prevent overflows (especially overflows over the cryptographic modulo used by MiniONN).

# Important note
The MiniONN code has a small drawback that has to be taken into account when working with it:
Currently, the server implementation uses a static dictionary in the minionn_helper file. This means that a server needs to be restarted after every run. This can easily be fixed by changing to a class that keeps all tensors of a specific instance and using this from the operation_helper.