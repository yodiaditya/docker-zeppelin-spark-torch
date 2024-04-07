FROM apache/zeppelin:0.10.1

ENV ZEPPELIN_HOME=/opt/zeppelin
ENV ZEPPELIN_NOTEBOOK_DIR /notebooks
ENV ZEPPELIN_INTERPRETER_DEP_MVNREPO https://repo1.maven.org/maven2/
ENV ZEPPELIN_ADDR 0.0.0.0
ENV SPARK_HOME /opt/bitnami/spark
ENV TZ=UTC \
    DEBIAN_FRONTEND=noninteractive

USER root

ENV PIP_CACHE_DIR=/var/cache/buildkit/pip
RUN mkdir -p $PIP_CACHE_DIR
RUN rm -f /etc/apt/apt.conf.d/docker-clean
RUN --mount=type=cache,target=/var/cache/apt \
        	apt-get update && \
	        apt-get install -yqq --no-install-recommends \
            libnss-wrapper gcc git wget vim build-essential libxml2 nvtop 

# Install libnss-wrapper
# RUN apt-get install -y libnss-wrapper gcc git wget vim build-essential libxml2

RUN mkdir -p /opt/bitnami/common/lib/
RUN ln -s /usr/lib/libnss_wrapper.so /opt/bitnami/common/lib/libnss_wrapper.so
ENV LD_PRELOAD=/usr/lib/libnss_wrapper.so

# Create the notebooks directory with proper permissions
RUN mkdir -p ${ZEPPELIN_NOTEBOOK_DIR}
RUN chown -R 1000:1000 ${ZEPPELIN_NOTEBOOK_DIR}
RUN chmod -R 777 ${ZEPPELIN_NOTEBOOK_DIR}

# Make sure to copy pip-cache in zeppelin folder
# This will use zeppelin folder as parent directory (Dockerfile context . )
COPY ./pip-cache ${ZEPPELIN_HOME}/pip-cache
RUN chown -R 1000:1000 ${ZEPPELIN_HOME}/pip-cache

# Copy required files
COPY ./conf/zeppelin-env.sh ${ZEPPELIN_HOME}/conf/zeppelin-env.sh
COPY ./requirements.txt ${ZEPPELIN_HOME}/requirements.txt
COPY ./tfx_requirements.txt ${ZEPPELIN_HOME}/tfx_requirements.txt

# Setup conda
ENV PATH /opt/conda/envs/python_3_with_R/bin:$PATH
COPY ./bashrc ${ZEPPELIN_HOME}/.bashrc

################# Install CUDA #################
# COMMENT THIS LINE UNTIL END INSTALLATION IF YOU DON'T NEED TO USE CUDA 

# ENSURE TO COPY THE cuda_11.8.0_520.61.05_linux.run and cudnn INTO THE zeppelin/cuda FOLDER

# CUDA Installation
COPY ./cuda ${ZEPPELIN_HOME}/cuda
RUN chmod a+x cuda/cuda_11.8.0_520.61.05_linux.run
RUN cuda/cuda_11.8.0_520.61.05_linux.run --silent --toolkit

# copy CUDNN following files into the cuda toolkit directory.
RUN cp -P cuda/cudnn/include/cudnn.h /usr/local/cuda/include
RUN cp -P cuda/cudnn/lib/libcudnn* /usr/local/cuda/lib64/
RUN chmod a+r /usr/local/cuda-11.8/lib64/libcudnn*

# Ensure cuda is added to the PATH, since ~./bashrc is not loaded
ENV PATH /usr/local/cuda/bin:${PATH}
ENV LD_LIBRARY_PATH /usr/local/cuda/lib:/usr/local/cuda/lib64

# Export the path
RUN echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ${ZEPPELIN_HOME}/.bashrc
RUN echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ${ZEPPELIN_HOME}/.bashrc
RUN ldconfig

################# End Install CUDA #################

RUN chown 1000:1000 ${ZEPPELIN_HOME}/.bashrc
USER 1000

# Install the required packages. You can modify the requirements.txt file to include the packages you need.
# At first run, you can run the following command to install the packages
# the pip-cache can be copied from Docker to the zeppelin folder with name `pip-cache`
RUN pip install --cache-dir pip-cache -r requirements.txt

################# Install TFX #################
# COMMENT THIS LINE UNTIL END INSTALLATION IF YOU DON'T NEED TO USE TFX 

# Install TFX
RUN pip install --cache-dir pip-cache -r tfx_requirements.txt
RUN pip uninstall shapely -y
################# END TFX Installation #################
