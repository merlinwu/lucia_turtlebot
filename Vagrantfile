# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  config.vm.box = "ubuntu/trusty64"

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: "192.168.37.15"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    #vb.gui = true

    # Customize the amount of memory on the VM:
    vb.memory = "2048"
  end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  $script = <<-SHELL
    export LC_ALL="en_US.UTF-8"
    export CI_ROS_DISTRO="indigo"
    export CATKIN_WS=~/catkin_ws
    export CATKIN_WS_SRC=${CATKIN_WS}/src
    
    sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu trusty main" > /etc/apt/sources.list.d/ros-latest.list'
    wget http://packages.ros.org/ros.key -O - | sudo apt-key add -
    sudo apt-get update -qq
    sudo apt-get install -qq -y python-rosdep python-wstool ros-$CI_ROS_DISTRO-catkin git build-essential cmake
    sudo rosdep init
    rosdep update
    
    # Install the Las Vegas Reconstruction toolkit
    sudo apt-get install -qq -y libfreenect-dev libopencv-dev libflann-dev libeigen3-dev libvtk5-dev libvtk5.8-qt4 python-vtk libvtk-java libboost-all-dev freeglut3-dev libxmu-dev libusb-1.0.0-dev
    mkdir -p ~/lucia_software
    cd ~/lucia_software
    git clone https://github.com/lasvegasrc/Las-Vegas-Reconstruction.git
    mkdir -p Las-Vegas-Reconstruction/build
    cd Las-Vegas-Reconstruction/build
    cmake -DCMAKE_BUILD_TYPE=Release ..
    make
    sudo make install

    # Install the 3D Toolkit
    sudo apt-get install -qq -y libzip-dev libann-dev libsuitesparse-dev libnewmat10-dev subversion
    cd ~/lucia_software
    svn checkout svn://svn.code.sf.net/p/slam6d/code/trunk slam6d-code
    mkdir -p slam6d-code/build
    cd slam6d-code/build
    rm ../3rdparty/CMakeModules/FindCUDA.cmake
    cmake -DWITH_FBR=ON .. ; cmake -DWITH_FBR=ON ..
    make

    # Create a Catkin workspace, clone and build our ROS stacks:
    mkdir -p $CATKIN_WS_SRC
    cd $CATKIN_WS_SRC/
    ln -s /vagrant/ $CATKIN_WS_SRC/lucia_turtlebot
    wstool init -j4 . https://raw.githubusercontent.com/uos/uos_rosinstalls/master/lucia2016-indigo.rosinstall
    
    # Use rosdep to install all dependencies (including ROS itself)
    sudo apt-get install -qq -y linux-image-extra-$(uname -r)  # workaround for ros-indigo-realsense-camera
    rosdep install --from-paths ./ -i -y --rosdistro $CI_ROS_DISTRO

    source /opt/ros/$CI_ROS_DISTRO/setup.bash
    catkin_init_workspace
    cd $CATKIN_WS

    # Run catkin_make up to three times: when rosjava is installed, it sometimes fails on the first try
    catkin_make -DCMAKE_BUILD_TYPE=Release || catkin_make -DCMAKE_BUILD_TYPE=Release || catkin_make -DCMAKE_BUILD_TYPE=Release

    echo 'export LC_ALL="en_US.UTF-8"' >> ~/.bashrc
    echo 'source ~/catkin_ws/devel/setup.bash' >> ~/.bashrc
    echo 'export ROS_MASTER_URI=http://192.168.37.15:11311' >> ~/.bashrc
    echo 'export ROS_HOSTNAME=192.168.37.15' >> ~/.bashrc
  SHELL
  config.vm.provision "shell", inline: $script, privileged: false
end
