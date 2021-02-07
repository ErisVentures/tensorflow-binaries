# Build Tensorflow from Source

## Guide

```bash
# Install go + bazelisk
brew install golang
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> ~/.zshrc
source ~/.zshrc
go get github.com/bazelbuild/bazelisk
ln -s $HOME/go/bin/bazelisk $HOME/go/bin/bazel

# Get tensorflow
git clone https://github.com/tensorflow/tensorflow.git
git checkout r2.3
cd tensorflow

# Setup a virtual environment
python3 -m venv env
source env/bin/activate
pip install --upgrade pip
pip install pip numpy wheel

# To build a native binary with maximum optimizations.
./configure # Use all defaults
bazel build --config=monolithic --config=opt //tensorflow/tools/lib_package:libtensorflow
cp -R ./bazel-bin/tensorflow/ ../march-native

# To build again but with narrower optimizations (no AVX).
# See https://gcc.gnu.org/onlinedocs/gcc/x86-Options.html for full list of CPU arch targets.
bazel clean
./configure # When asked for opt flags don't use `-march=native`, use `-march=core2`.
bazel build --config=monolithic --config=opt //tensorflow/tools/lib_package:libtensorflow
cp -R ./bazel-bin/tensorflow/ ../march-core2

# Inspect build flags
less .tf_configure.bazelrc
less .bazelrc

# To use with @tensorflow/tfjs-node
cp march-core2/tools/lib_package/libtensorflow.tar.gz path/to/repo/node_modules/@tensorflow/tfjs-node/deps/
cd path/
to/repo/node_modules/@tensorflow/tfjs-node/deps/
tar -xzf libtensorflow.tar.gz
```

## Resources

- https://www.tensorflow.org/install/source
- https://github.com/tensorflow/tensorflow/blob/master/tensorflow/tools/lib_package/README.md
