## make sure in the first run, run build using first half of the Docker file, stop at "RUN echo 3 | plumed patch -p". find what the number is, and then run full file
##you can also check this page, https://github.com/plumed/plumed2/tree/master/patches, find the index of right version

FROM nvidia/cuda:12.1.0-cudnn8-devel-ubuntu22.04

RUN apt-get update && apt-get install -y git python3-pip cmake libopenmpi-dev jq && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists/*
  
RUN git clone https://github.com/plumed/plumed2.git 

WORKDIR /plumed2
RUN ./configure && make -j10 && make install

ADD https://github.com/gromacs/gromacs/archive/refs/tags/v2024.3.tar.gz /

RUN mkdir /gromacs && tar xfz /v2024.3.tar.gz -C /gromacs --strip-components=1

WORKDIR /gromacs

ENV LD_LIBRARY_PATH=/usr/local/lib CUDACXX=/usr/local/cuda/bin/nvcc

############ in the first pass, stop here to valide the setting for plumed. In current version, it is 3
RUN echo 3 | plumed patch -p

RUN pip install -r python_packaging/gmxapi/requirements.txt
RUN mkdir build
WORKDIR /gromacs/build
RUN cmake .. -DGMX_BUILD_OWN_FFTW=ON -DREGRESSIONTEST_DOWNLOAD=ON -DGMX_GPU=CUDA && \
   make -j 20 && \
   make install 

WORKDIR /

RUN rm -fr gromacs && rm -fr /plumed2

ADD entrypoint.sh /usr/local/gromacs/bin

ENTRYPOINT ["/usr/local/gromacs/bin/entrypoint.sh"]
