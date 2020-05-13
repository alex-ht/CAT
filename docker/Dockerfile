FROM pytorch/pytorch:1.5-cuda10.1-cudnn7-devel

# Get latest CMake
RUN DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -y apt-transport-https ca-certificates gnupg software-properties-common wget
RUN wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | gpg --dearmor - | tee /etc/apt/trusted.gpg.d/kitware.gpg >/dev/null
RUN apt-add-repository 'deb https://apt.kitware.com/ubuntu/ bionic main' && apt-get update

# Install Kaldi
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y cmake flac build-essential git subversion automake autoconf unzip sox gfortran libtool python2.7
RUN ln -sf /bin/bash /bin/sh

RUN conda install -y mkl-include && \
    git clone --depth 1 https://github.com/kaldi-asr/kaldi.git /opt/kaldi && \
    cd /opt/kaldi && \
    cd /opt/kaldi/tools && \
    make -j $(nproc) && \
    cd /opt/kaldi/src && \
    ./configure --shared --use-cuda --mkl-root=/opt/conda --mkl-libdir=/opt/conda/lib && \
    make depend -j $(nproc) && \
    make -j $(nproc)

ADD . /opt/CAT

# Step 1. Copy src/kaldi-patch/latgen-faster.cc to kaldi src/bin, and compile.
RUN cp /opt/CAT/src/kaldi-patch/latgen-faster.cc /opt/kaldi/src/bin
WORKDIR /opt/kaldi/src/bin
RUN sed -i "s:BINFILES =:BINFILES = latgen-faster:g" Makefile
RUN make latgen-faster

# Step 2. cd src/ctc_crf, and run the commands below:
# make OPENFST=/path/to/your/openfst
# For pytorch version 1.0+, use python setup_1_0.py install in the ctc_crf/Makefile.
WORKDIR /opt/CAT/src/ctc_crf
RUN OPENFST=/opt/kaldi/tools/openfst make GPUCTC GPUDEN PATHWEIGHT
RUN python setup_1_0.py install

# Clear the APT cache
RUN apt-get clean

WORKDIR /opt/CAT
