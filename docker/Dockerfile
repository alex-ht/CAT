FROM kaldiasr/kaldi:gpu-latest

RUN DEBIAN_FRONTEND=noninteractive apt-get install -y cmake python3-pip flac
RUN pip3 install --upgrade pip
RUN pip3 install --no-cache-dir torch===1.2.0 torchvision===0.4.0 -f https://download.pytorch.org/whl/torch_stable.html

ADD . /opt/CAT

# Step 1. Copy src/kaldi-patch/latgen-faster.cc to kaldi src/bin, and compile.
RUN cp /opt/CAT/src/kaldi-patch/latgen-faster.cc /opt/kaldi/src/bin
WORKDIR /opt/kaldi/src/bin
RUN sed -i "s:BINFILES =:BINFILES = latgen-faster:g" Makefile
RUN make latgen-faster

#  Step 2. cd src/ctc_crf, and run the commands below:
#  make OPENFST=/path/to/your/openfst
# For pytorch version 1.0+, use python setup_1_0.py install in the ctc_crf/Makefile.
WORKDIR /opt/CAT/src/ctc_crf
RUN OPENFST=/opt/kaldi/tools/openfst make GPUCTC GPUDEN PATHWEIGHT
RUN python3  setup_1_0.py install

# Clear the APT cache
RUN apt-get clean

WORKDIR /opt/CAT
