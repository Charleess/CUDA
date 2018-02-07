# How to instal CUDA on Ubuntu 16.04 LTS

This tutorial details all the steps for a clean install of CUDA 9.1 on a computer running Ubuntu 16.04. The installation and configuration process is *very* tedious, and might take a couple hours.

## Prerequisites

1. Check that the card you want to use is listed on NVIDIA's website as CUDA Capable
1. Check the current version of your kernel and downgrade if necessary
1. Follow the preinstallation checks from NVIDIA

### Verifying and Downgrading Kernel

Check the current version of your kernel with:

```bash
$ uname -r
4.13.0-32-generic
```

CUDA needs v4.4 to work. Fetch the 4.4 kernel directly on ubuntu's website. These are the commands for a `amd64` architecture (the most common). Simply go to `/mainline/` on the links below to see the one suited for you.

```bash
$ cd /tmp/
$ wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.4.115/linux-headers-4.4.115-0404115_4.4.115-0404115.201802031230_all.deb
$ wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.4.115/linux-headers-4.4.115-0404115-generic_4.4.115-0404115.201802031230_amd64.deb
$ wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.4.115/linux-image-4.4.115-0404115-generic_4.4.115-0404115.201802031230_amd64.deb
```

Execute the newly downloaded packages and rebot your computer

```bash
$ sudo dpkg -i linux-headers-4.4.115-*.deb linux-image-4.4.114-*.deb
$ sudo reboot
```

At boot, on the bootloader, select `Advanced options for Ubuntu`. You should see a list of available kernels, simply select the 4.4 version. Check that the version has been correctly installed:

```bash
$ uname -r
4.4.115-0404115-generic
```

### Pre-Installations checks

Check that gcc is installed. If you see an error message, install it with `apt-get`

```bash
$ gcc --version
gcc (Ubuntu ...)
```

Install the development packages if needed (If you previously downgraded your kernel, this should do nothing):

```bash
$ sudo apt-get install linux-headers-$(uname -r)
```

## Installation

Let's get started, we are going to use the "Package Manager Installation", as stated in the *not-so-clear* documentation from NVIDIA.

1. Download the CUDA toolkit from [NVIDIA](https://developers.nvidia.com/cuda-downloads). Choose the `.deb` download for ubuntu 16.04 x86_64. 
1. Add the key to your `apt`
1. Update the repositories
1. Install the toolkit.

```bash
$ sudo dpkg -i cuda-repo-ubuntu1604-9-1-local_9.1.85-1_amd64.deb
$ sudo apt-key add /var/cuda-repo-9-1-local/7fa2af80.pub
$ sudo apt-get install cuda
```

The last step should take a few minutes, and use about 2Gb of disk. **REBOOT** after the installation.

I was unable to login after the install, to fix it, log into your 4.13 kernel, reboot, and log back in the 4.4.

## Post-Installation

### Add CUDA to your $PATH.

Add the following line to your `.bashrc` (Access it by typing `vim ~/.bashrc`)

```bash
export PATH=/usr/local/cuda-9.1/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-9.1/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

Close the terminal, reopen it and check the two variables.

```bash
$ echo $PATH
$ echo $LD_LIBRARY_PATH
```

You should see CUDA in both cases.

### Add a crontab to lauch the CUDA Daemon on boot

```bash
$ crontab -e
```

Choose your text editor (3 for VIM) and add a new line at the end of the created file with: `@reboot /usr/bin/nvidia-persistenced --verbose`. This will run at startup.

### Check that the CUDA compiler is installed

```bash
$ nvcc --version
nvcc: NVIDIA (R) Cuda compiler driver
Copyright ...
```

### Install samples

This one was a bit tricky. First, make the examples script executable with `chmod`

```bash
$ cd /usr/local.cuda-9.1/bin
$ sudo chmod a+x cuda-install-samples-9.1.sh
$ cd ~
$ cuda-install-samples-9.1.sh /home/<USERNAME>
Copying ...
Finished compying samples
```

### Verify driver version and activate GPU

```bash
$ cat /proc/driver/nvidia/version
NVRM version: NVIDIA UNIX X86_64 Kernel Module 387.26
GCC version: gcc version 5.4.0
```

Version 387 seems to be the latest at this time.

```bash
$ nvidia-smi
```

If you have a message saying this command was not found, you might be running a laptop with integrated graphics AND a dedicated card. To switch to the GPU, use:

```bash
$ sudo prime-select nvidia
```

You should now be able to see you GPU stats with `nvidia-smi`. Check that your driver is `r387`.

> Note that continuous usage of the GPU will drain your battery much faster.

### Compile an example

**REBOOT** before this operation to make sure everything is OK.

```bash
$ cd ~/NVIDIA_CUDA-9.1_Samples
$ make
```

This will take a while before the examples are compiled.

### Actually test the install

This is the moment of truth. Make sure the examples compiled successfully, and run a test.

```bash
$ cd ~/NVIDIA_CUDA-9.1_Samples/bin/x86_64/linux/release
$ ./deviceQuery
```

If everything is OK, you should see a screen with the specs of your GPU. If you have a `35` error, make sure the GPU driver is `r387` in both `$ nvidia-smi`, and `$ nvcc --version`. If not, go back on the steps where we check these things.

You can also run a bandwidth test:

```bash
$ cd ~/NVIDIA_CUDA-9.1_Samples/bin/x86_64/linux/release
$ ./bandwidthTest
```

Test should `PASS`.

## Install pyTorch

Let's try the install. Install miniconda3 from their website. Create a new environment, add pytorch and jupyter, and test.

```bash
$ conda create -n DeepGPU
$ source activate DeepGPU
$ conda install pytorch jupyter torchvision cuda90 -c pytorch
```

Create a new notebook and try:

```python
import torch
torch.cuda.is_available()
```

Output should be `True`

ALLELUIA !

## Basic VIM commands if needed

* To insert text, go to insert mode by pressing `i`
* To go back to command mode press `esc`
* To save and quit, type `:wq` (Add a `!` at the end if an override is necessary)
