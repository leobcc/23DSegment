# --------------------------------------------------------------------------------------------------------------------
# The image needs to be run with: $ docker run -it --gpus all --name lbocchi-4D-Dress-Seg -v /data-slow/datasets/4d_dress:/home/lbocchi/UnA-Gen/data/4d_dress -p 8760:8888 lbocchi-4d-dress-seg
# If changes are not being included in the build use $ docker build --no-cache -t lbocchi-4d-dress-seg . 
# To select the gpu to be used: $ export CUDA_VISIBLE_DEVICES=1,2
# Check memory usage $ sudo du -d 1 -h --exclude /home/lbocchi/UnA-Gen/data --exclude /proc /
# -------------------------------------------------------------------------------------------------------------------- DEBIAN_FRONTEND=noninteractive

# Build the container using the CUDA toolkit version 11.1 image as the base image
FROM nvidia/cuda:11.3.1-devel-ubuntu20.04

ARG USER_NAME=lbocchi
ARG USER_ID=10034
ARG GROUP_NAME=hupba
ARG GROUP_ID=10000

# Fetch the keys that are missing from the CUDA base image 
RUN apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/3bf863cc.pub

# Update and install general dependencies without Python 3.8 being pulled
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    software-properties-common \
    sudo \
    build-essential \
    git \
    git-lfs \
    libglfw3 \
    wget \
    unzip \
    && rm -rf /var/lib/apt/lists/*

# Ensure you are using Python 3.10 from the start
RUN add-apt-repository ppa:deadsnakes/ppa && apt-get update && apt-get install -y \
    python3.8 \
    python3.8-dev \
    python3.8-venv \
    python3-pip \
    && rm -rf /var/lib/apt/lists/*

# Create symbolic links to ensure `python3` points to Python 3.10
RUN update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.8 1

# Manually create a symlink for pip3 to use Python 3.10's pip
RUN ln -sf /usr/bin/pip3 /usr/local/bin/pip && ln -sf /usr/bin/python3.8 /usr/local/bin/python

# Upgrade pip (this is necessary otherwise the installation of setuptools gives problems for some versions)
#RUN python3 -m pip install --upgrade pip

###
# Set up environment
# Update and install system dependencies
RUN apt-get update && apt-get install -y \
    wget \
    build-essential \
    ffmpeg \
    libjpeg-dev \
    libpng-dev \
    libtiff-dev \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies with pip
RUN pip install --upgrade pip setuptools wheel

# Install core Python libraries and dependencies
RUN pip install \
    torch==1.11.0+cu113 \
    torchvision==0.12.0+cu113 \
    torchaudio==0.11.0+cu113 \
    --extra-index-url https://download.pytorch.org/whl/cu113

# Install additional packages
RUN pip install \
    fvcore==0.1.5.post20210915 \
    iopath==0.1.9 \
    pillow==10.2.0 \
    requests==2.31.0 \
    tqdm==4.66.2 \
    termcolor==2.4.0 \
    typing_extensions==4.9.0 \
    pyyaml==5.1.2 \
    #pytorch3d==0.7.2 \
    opencv-python==4.9.0.80 \
    scikit-image==0.21.0 \
    numpy==1.22.4 \
    scipy==1.10.1 \
    matplotlib==3.7.5 \
    networkx==3.1

# Add any other required packages listed under pip
RUN pip install \
    contourpy==1.1.1 \
    lazy-loader==0.4 \
    segment-anything==1.0 \
    pycocotools==2.0.7
###

# Create user and group
RUN groupadd --gid $GROUP_ID $GROUP_NAME && \
    useradd --uid $USER_ID --gid $GROUP_ID --shell /bin/bash --create-home $USER_NAME

# Grant sudo access to the user
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers && \
    usermod -aG sudo $USER_NAME

# Downgrade numpy to version 1.23.1 to avoid ImportError: cannot import name 'bool' from 'numpy'
#RUN python3 -m pip install numpy==1.23.1

# Install pip using get-pip.py (ensure the correct pip is installed)
RUN wget https://bootstrap.pypa.io/get-pip.py && python3.8 get-pip.py

# Remove the get-pip.py script after installation
RUN rm get-pip.py

# Upgrade setuptools to avoid issues with newer package installations
RUN python3.8 -m pip install --upgrade setuptools

# Set environment variables
ENV FORCE_CUDA=1
ENV TORCH_CUDA_ARCH_LIST="8.6"

USER $USER_NAME
WORKDIR /home/$USER_NAME

# Clone the repository
RUN git clone -b main --single-branch https://github.com/eth-ait/4d-dress.git
WORKDIR /home/lbocchi/4d-dress
#RUN conda env create -f environment.yml
#RUN conda activate 4ddress
#RUN conda create -n 4ddress python==3.8
#RUN conda activate 4ddress
#RUN bash env_install.sh

RUN git clone https://github.com/Gaoyiminggithub/Graphonomy.git 4dhumanparsing/lib/Graphonomy
RUN mkdir 4dhumanparsing/checkpoints
RUN mkdir 4dhumanparsing/checkpoints/graphonomy

RUN git clone https://github.com/princeton-vl/RAFT.git 4dhumanparsing/lib/RAFT
RUN wget -P 4dhumanparsing/checkpoints/raft/ https://dl.dropboxusercontent.com/s/4j4z58wuv8o0mfz/models.zip
RUN unzip 4dhumanparsing/checkpoints/raft/models.zip -d 4dhumanparsing/checkpoints/raft/

RUN pip install git+https://github.com/facebookresearch/segment-anything.git
RUN wget -P 4dhumanparsing/checkpoints/sam/ https://dl.fbaipublicfiles.com/segment_anything/sam_vit_h_4b8939.pth

RUN git clone https://github.com/yujiali/pygco.git 4dhumanparsing/lib/pygco
RUN cd 4dhumanparsing/lib/pygco && \
    wget -N -O gco-v3.0.zip https://vision.cs.uwaterloo.ca/files/gco-v3.0.zip && \
    unzip -o gco-v3.0.zip -d ./gco_source && \
    make all && \
    cd ../../..

# cd sam2 && pip install -e . && pip install -e ".[notebooks]"
# cd checkpoints && ./download_ckpts.sh && cd ..
# pip install ipywidgets


# Install CUDA-compatible version of PyTorch
#RUN pip install torch==2.3.1 torchvision==0.18.1 torchaudio==2.3.1 --index-url https://download.pytorch.org/whl/cu121

#RUN pip install -r requirements.txt

#RUN mkdir supp_repos
#RUN wget -O depth_anything.zip https://github.com/LiheYoung/Depth-Anything/archive/main.zip
#RUN unzip depth_anything.zip -d supp_repos
#RUN mv supp_repos/Depth-Anything-main supp_repos/Depth_Anything_main
#RUN rm depth_anything.zip

#RUN pip install -r /home/$USER_NAME/UnA-Gen/supp_repos/Depth_Anything_main/requirements.txt

# Remains to fix the copy of the smpl folder and smpl_model checkpoints, 
# as well as the midification in the depth_anything dpt file for a path set manually. 
# Also, the antialias option that needs to be removed cause it's missing from a librayry.
# (one option is to include everything in the repo as an adapted modification)