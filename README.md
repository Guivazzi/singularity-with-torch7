# Passo a passo de como criar um container com Torch7, Lua, Luarocks e pacotes necessarios, CUDA9.0 e LibCuDNN5.x
  No terminal, criar um arquivo com o nome de “Singularity”

$ sudo vim Singularity

# Dentro do arquivo colar as seguintes linhas do quadro abaixo:

-----------------------------------------------------------------------------------------------------------------------------------
Bootstrap: docker
From: ubuntu:16.04

%post

    #Pacotes necessarios

    apt-get -y update
    apt-get -y upgrade
    apt -y install vim
    apt -y install sudo
    apt -y install git
    apt -y install lua5.2
    apt-get -y install libhdf5-serial-dev
    apt-get -y install --no-install-recommends libhdf5-serial-dev liblmdb-dev
    apt -y install wget

    #Instalação e compilação do Torch
    export TORCH_ROOT=/usr/local/torch/torch
    git clone https://github.com/torch/distro.git $TORCH_ROOT --recursive
    cd $TORCH_ROOT
    export TORCH_NVCC_FLAGS="-D__CUDA_NO_HALF_OPERATORS__"
    bash install-deps
    yes | bash ./install.sh


    #Instalação do CUDA

    wget https://developer.nvidia.com/compute/cuda/9.0/Prod/local_installers/cuda-repo-ubuntu1604-9-0-local_9.0.176-1_amd64-deb
    dpkg --install cuda-repo-ubuntu1604-9-0-local_9.0.176-1_amd64-deb
    apt-key add /var/cuda-repo-9-0-local/7fa2af80.pub
    apt-get update
    export TORCH_NVCC_FLAGS="-D__CUDA_NO_HALF_OPERATORS__"
    apt-get install -y cuda

    #Pacotes luarocks

    luarocks install "https://raw.github.com/deepmind/torch-hdf5/master/hdf5-0-0.rockspec"
    luarocks install "https://raw.github.com/Neopallium/lua-pb/master/lua-pb-scm-0.rockspec"
    luarocks install lightningmdb 0.9.18.1-1 LMDB_INCDIR=/usr/include LMDB_LIBDIR=/usr/lib/x86_64-linux-gnu
    luarocks install "https://raw.githubusercontent.com/ngimel/nccl.torch/master/nccl-scm-1.rockspec"
    luarocks install nngraph
    luarocks install optim
    luarocks install nn
    luarocks install camera
    luarocks install qtlua
    luarocks install hdf5
    luarocks install cudnn
    luarocks install cunn
    luarocks install cutorch


    #Instalação Conda

    wget https://repo.continuum.io/archive/Anaconda3-5.1.0-Linux-x86_64.sh
    bash Anaconda3-5.1.0-Linux-x86_64.sh -f -b -p /usr/anaconda
    export PATH="/usr/anaconda/bin:$PATH"

    conda update -y conda
    conda install -c anaconda jupyter

    #Adicionais

    apt -y install mirage
    apt -y install firefox

%environment
    export PATH=$PATH:/usr/local/torch/torch/install/bin
    export TORCH_ROOT=/usr/local/torch/torch/
    export PATH="$HOME/usr/local/torch/torch/install/bin:$PATH"
    export PATH="/usr/anaconda/bin:$PATH"
    export TORCH_NVCC_FLAGS="-D__CUDA_NO_HALF_OPERATORS__"
    export LC_ALL=C
    export PATH=/usr/games:$PATH




-----------------------------------------------------------------------------------------------------------------------------------

#Salve o arquivo.
#Crie o container com o seguinte comando:

$ sudo singularity build --writable torch.img Singularity

#Após a conclusão do container (vai demorar cerca de 1h20 min para concluir), entre no mesmo com o seguinte comando;

$ singularity shell torch.img

 #Para validar o funcionamento básico do mesmo, digite “th” e valide se o Torch está funcionando.
 #Por último, é necessário baixar e  instalar o LibcuDNN 5.1/ CUDA 7.5, e para isso iremos no [Link] e baixar o mesmo para “Ubuntu 14.04” 

#Baixar a versão Runtime, e Developer
 #A única pasta compartilhada entre o seu computador e o container é o /root/, logo, precisamos ter privilégios root para mover os dois arquivos .deb baixados para a pasta /root/.


$ cd Downloads/
$ sudo cp libcudnn5_5.0.5-1+cuda7.5_amd64.deb /root/
$ sudo cp libcudnn5-dev_5.0.5-1+cuda7.5_amd64.deb /root/

 #Após copiar as libcudnn para a pasta root, entraremos novamente no container criado, mas para conseguir alterar entraremos com sudo e permissões de escrita.

$ sudo singularity shell --writable torch.img

 #Ao dar o comando “ls” após entrar no container, já iremos ver os pacotes copiados para a pasta /root.
 para instalar segue os comandos:

$ dpkg -i libcudnn5_5.0.5-1+cuda7.5_amd64.deb
$ dpkg -i libcudnn5-dev_5.0.5-1+cuda7.5_amd64.deb

 #Validar se a Lib  se encontra como padrão no container com o comando:

$ update-alternatives --config libcudnn
