FROM pytorch/pytorch:0.4.1-cuda9-cudnn7-devel

# Install Kaldi
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
    cmake wget flac build-essential git subversion automake \
    autoconf unzip sox gfortran libtool python2.7 && \
    ln -sf /bin/bash /bin/sh

RUN pip install mkl-include h5py && \
    git clone --depth 1 https://github.com/kaldi-asr/kaldi.git /opt/kaldi && \
    cd /opt/kaldi/tools && \
    make -j $(nproc) && \
    cd /opt/kaldi/src && \
    ./configure --shared --use-cuda --mkl-root=/opt/conda --mkl-libdir=/opt/conda/lib && \
    make depend -j $(nproc) && \
    make -j $(nproc)

COPY . /opt/CAT

# Step 1. Copy src/kaldi-patch/latgen-faster.cc to kaldi src/bin, and compile.
COPY src/kaldi-patch/latgen-faster.cc /opt/kaldi/src/bin
RUN cd /opt/kaldi/src/bin && \
    sed -i "s:BINFILES =:BINFILES = latgen-faster:g" Makefile && \
    make latgen-faster

# Step 2. cd src/ctc_crf, and run the commands below:
# make OPENFST=/path/to/your/openfst
# For pytorch version 1.0+, use python setup_1_0.py install in the ctc_crf/Makefile.
RUN cd /opt/CAT/src/ctc_crf && \
    OPENFST=/opt/kaldi/tools/openfst make && \
    cd /opt/CAT/src/batchnorm_src && \
    PYTORCH_LIB_DIR=/opt/conda/lib/python3.6/site-packages/torch/lib PYTHON=python3 make

# Clear the APT cache
RUN apt-get clean

WORKDIR /opt/CAT
