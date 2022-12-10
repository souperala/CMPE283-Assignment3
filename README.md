# CMPE 283 - Virtualization Technologies

## Assignment II - Instrumentation via hypercall (Student ID: 016114254)


   Objective:  
    The aim of this assignment is to modify the CPUID emulation code in KVM to report back additional information when special CPUID leaf nodes are requested.
    
        CPUID leaf node %eax=0x4FFFFFFC:
            Return the total number of exits (all types) in %eax
        CPUID leaf node %eax=0x4FFFFFFD:
            Return the high 32 bits of the total time spent processing all exits in %ebx
            Return the low 32 bits of the total time spent processing all exits in %ecx
              %ebx and %ecx return values are measured in processor cycles, across all vCPUs
              
   ### 1. Team contribution:
   
        I have done this assignment alone.
   
   ### 2. Procedure:
   
#### step 1: Create a virtual machine (VM) on Google Cloud Engine by following the instructions and enable the virtualization.
  Configuration used to launch VM: 
  Series: N2
  Machine Type: n2-standard-8
  CPU platform: Intel Cascade Lake
  Launch instance with the SSH Keys and Metadata added:
  <img width="1503" alt="image" src="https://user-images.githubusercontent.com/98585812/205783799-fbd3077e-882a-41c6-bd11-c42346321dad.png">
  
#### step 2: Install all the necessary updates & dependencies using the below commands
      
      sudo apt-get update
      sudo apt-get upgrade
      sudo apt-get install vim gcc make linux-headers-$(uname -r)
      
<img width="787" alt="image" src="https://user-images.githubusercontent.com/98585812/205785980-6a7b11d5-f098-4ee9-a76b-05b5803a4bbd.png">
<img width="784" alt="image" src="https://user-images.githubusercontent.com/98585812/205786103-e86ff8d3-1e40-42b8-b754-e26a6ed019cd.png">
<img width="755" alt="image" src="https://user-images.githubusercontent.com/98585812/205786157-1fd22387-ef65-4826-b841-24e866301c78.png">

#### step 3: Download the source code from the offical linux kernel website and extract the folder
<img width="975" alt="image" src="https://user-images.githubusercontent.com/98585812/205786489-e5e01129-9bbb-4cfe-9f77-6a687373cf00.png">

      tar xvf linux-6.0.7.tar.xz

#### step 4: Install all the necessary packeges which are required to build the kernel

      sudo apt-get install git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison
<img width="979" alt="image" src="https://user-images.githubusercontent.com/98585812/205787596-ff45fb01-ae58-4648-a6c7-3b1ceea67408.png">

#### step 5: Copying the current configuration file to a.config file to configure the kernel

      cd linux-6.0.7
      cp -v /boot/config-$(uname -r) .config
      make menuconfig
      make oldconfig
      
<img width="966" alt="image" src="https://user-images.githubusercontent.com/98585812/205788433-f25a0d20-bdd7-4efd-86d1-56ea54f10f9b.png">
      
#### step 6: Build the kernel using the following commands 

     If there is any error saying "No rule to make target "debian/canonical-certs.pem"" appears while building the kernel, run the following two commands   to fix it. :
        # scripts/config --disable SYSTEM_TRUSTED_KEYS
        # scripts/config --disable SYSTEM_REVOCATION_KEYS
      sudo make -j 512 modules
      sudo make -j 512
      sudo make -j 512 modules_install
      sudo make -j 512 install
      
<img width="957" alt="image" src="https://user-images.githubusercontent.com/98585812/205788515-f11da20a-18ed-4858-9598-c9d57f7c668f.png">
<img width="942" alt="image" src="https://user-images.githubusercontent.com/98585812/205788547-780668f3-22eb-4620-968e-f053994a31c9.png">
<img width="980" alt="image" src="https://user-images.githubusercontent.com/98585812/205788602-a3c243d8-0f15-4fe9-8805-b36cb5814f75.png">
<img width="967" alt="image" src="https://user-images.githubusercontent.com/98585812/205788636-9550e901-3de0-49fe-883b-22787d0d0c6a.png">

#### step 7: Once the make is executed, the bootloader will be automatically updated and now we can reboot the system and check the version once the VM has resumed.

      sudo reboot
      uname -mrs
<img width="956" alt="image" src="https://user-images.githubusercontent.com/98585812/205790539-a5ed7d31-30c5-4338-be4a-af0a0481c488.png">
<img width="817" alt="image" src="https://user-images.githubusercontent.com/98585812/205790983-15509392-d814-48f1-a6ac-8c10752b6aaf.png">

#### step 8: Overwrite the kernel code by downloading the vmx.c and cpuid.c from your repository by making the repo public

      vmx.c: /linux-6.0.7/arch/x86/kvm/vmx/vmx.c
      cd /linux-6.0.7/arch/x86/kvm/vmx/
      rm vmx.c
      wget https://raw.githubusercontent.com/souperala/CMPE283-Assignment2/main/vmx.c
      cpuid.c: /linux-6.0.7/arch/x86/kvm/cpuid.c
      cd /linux-6.0.7/arch/x86/kvm/
      rm cpuid.c
      wget https://raw.githubusercontent.com/souperala/CMPE283-Assignment2/main/cpuid.c
      
<img width="1203" alt="image" src="https://user-images.githubusercontent.com/98585812/205812630-a6972082-1d32-41cd-9458-696c0336f964.png">
<img width="1176" alt="image" src="https://user-images.githubusercontent.com/98585812/205812669-3cb18310-1562-497c-8a04-6683672dc67f.png">

    
#### step 9: Build the kernel and install the modules with the updated kernel code

      sudo make -j 512 modules && sudo make -j 512 modules_install
<img width="863" alt="image" src="https://user-images.githubusercontent.com/98585812/205794620-ea09038a-4963-401a-965b-1c320a17d424.png">

#### step 10: Reload the kernel modules following the below commands

      sudo rmmod kvm_intel
      sudo rmmod kvm
      sudo modprobe kvm
      sudo modprobe kvm_intel

<img width="972" alt="image" src="https://user-images.githubusercontent.com/98585812/205794873-5caf1ed2-728d-4098-9ba3-3a84631f83a9.png">

#### step 11: Create a nested VM in the current GCP VM by following the below steps

- Download the Ubuntu cloud image(QEMU compatible image) in GCP VM from the below ubuntu cloud images site
    
      wget https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img
    
- Install the necessary QEMU packages
    
      sudo apt update && sudo apt install qemu-kvm -y
    
- Go to the folder where the.img file was downloaded. There is no default password for the virtual machine included with this Ubuntu cloud image.
So, use these instructions to modify the password and log in to the virtual machine:

       sudo apt-get install cloud-image-utils
       vi user-data
           #cloud-config
           password: newpass #new password here
           chpasswd: { expire: False }
           ssh_pwauth: True
       cloud-localds user-data.img user-data (Give the username and password to connect to this nested VM)

<img width="1006" alt="image" src="https://user-images.githubusercontent.com/98585812/205796526-07c40e37-1d4f-4cd9-bf17-429247785c19.png">
<img width="969" alt="image" src="https://user-images.githubusercontent.com/98585812/205796670-1d0bc369-6204-437e-80d7-0699522f785a.png">
<img width="975" alt="image" src="https://user-images.githubusercontent.com/98585812/205796728-9ca9e735-89b6-4203-84f1-4b33ae050205.png">

- New VM has been launched, here is a snapshot of the nested VM
<img width="744" alt="image" src="https://user-images.githubusercontent.com/98585812/205811400-f7a658e5-ebc2-46f3-8bd1-afe6f7850a6c.png">

#### step 12: To check the VM functions correctly, install the cpuid utility by following the below commands
        
        sudo apt-get update
        sudo apt-get install cpuid  
<img width="958" alt="image" src="https://user-images.githubusercontent.com/98585812/205799963-39261c6d-47f6-43d2-8573-61dd3aa13b64.png">

#### step 13: open the two terminals(GCP VM "T1", and nested VM terminal "T2") and test the cpuid functionality using the below commands

        Testing the CPUID functionality for '%eax=0x4ffffffc'      
        T2: sudo cpuid -l 0x4ffffffc
<img width="756" alt="image" src="https://user-images.githubusercontent.com/98585812/205811522-37044237-df26-4c76-9ef8-91e164d7c18a.png">
        T1: sudo dmesg
<img width="770" alt="image" src="https://user-images.githubusercontent.com/98585812/205811575-d1d8de84-06a7-4023-9694-72229b0c1e67.png">

        Testing the CPUID functionality for '%eax=0x4ffffffd'
        T2: sudo cpuid -l 0x4ffffffd
<img width="730" alt="image" src="https://user-images.githubusercontent.com/98585812/205811627-32facdb8-116c-48e1-af73-8a4976e0892d.png">
        T1: sudo dmesg
<img width="781" alt="image" src="https://user-images.githubusercontent.com/98585812/205811677-7a421277-a5a6-44f6-9192-751f5709e162.png">

    







      


            
