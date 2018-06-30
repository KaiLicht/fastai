FROM continuumio/miniconda3

LABEL maintainer="Kai Lichtenberg <kai@sentin.ai>"

RUN apt-get update && apt-get install -y --allow-downgrades --no-install-recommends \
    python-qt4 \
    libgl1-mesa-glx \
    make \
    libpng-dev &&\
    rm -rf /var/lib/apt/lists/* &&\
    mkdir /fastai

WORKDIR /fastai

RUN git clone https://github.com/kailicht/fastai.git . &&\
    /opt/conda/bin/conda env update --file /fastai/docs/environment-doc.yml &&\
    /opt/conda/bin/conda clean -ya &&\
    rm -rf /fastai

ENV PATH /opt/conda/envs/fastai-doc/bin:$PATH

CMD ["sh","-c","cd /fastai/docs/ && make clean && make html"]