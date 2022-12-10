# CMPE 283 - Virtualization Technologies

## Assignment III - Instrumentation via hypercall (Student ID: 016114254)


   Objective:  
    The aim of this assignment is to modify the CPUID emulation code in KVM to report back additional information when special CPUID leaf nodes are requested.
    
        CPUID leaf node %eax=0x4FFFFFFF:
            Return the number of exits for the exit number provided (on input) in %ecx
               This value should be returned in %eax 
        CPUID leaf node %eax=0x4FFFFFFE:
            Return the time spent processing the exit number provided (on input) in %ecx
                 Return the high 32 bits of the total time spent for that exit in %ebx
                 Return the low 32 bits of the total time spent for that exit in %ecx
              
   ### 1. Team contribution:
   
        I have done this assignment alone.
   
   ### 2. Procedure:
   
#### step 1: Create a virtual machine (VM) on Google Cloud Engine by following the instructions and enable the virtualization.
        Configuration used to launch VM: 
        Series: N2
        Machine Type: n2-standard-8
        CPU platform: Intel Cascade Lake
        Launch instance with the SSH Keys and Metadata added:
  <img width="1100" alt="image" src="https://user-images.githubusercontent.com/98585812/206857598-bbdf122a-2285-4ed7-b966-a6f4d10903a5.png">

  
#### step 2: Install all the necessary updates & dependencies using the below commands
      
         sudo apt-get update
         sudo apt-get upgrade
         sudo apt-get install vim gcc make linux-headers-$(uname -r)

<img width="1082" alt="image" src="https://user-images.githubusercontent.com/98585812/206857614-8390eae1-a605-4fc3-a895-c2e5fd6eb28d.png">
<img width="1102" alt="image" src="https://user-images.githubusercontent.com/98585812/206857634-60b5ca54-e932-4f66-a128-68049618fb30.png">
<img width="1103" alt="image" src="https://user-images.githubusercontent.com/98585812/206857656-f462ecb1-8e4b-46b7-b7ff-ae17101f6a78.png">


#### step 3: Download the source code from the offical linux kernel website and extract the folder
      
         sudo apt-get install wget
         wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.0.7.tar.xz
         tar xvf linux-6.0.7.tar.xz
          
<img width="1100" alt="image" src="https://user-images.githubusercontent.com/98585812/206857689-16bd4ef7-fa5b-40f5-a0e8-68484799ce79.png">
<img width="1082" alt="image" src="https://user-images.githubusercontent.com/98585812/206857721-0f6ccda7-8a32-40f7-9699-a75721253089.png">
<img width="1091" alt="image" src="https://user-images.githubusercontent.com/98585812/206857733-8b8df579-2171-462e-8fd6-2c666ff06896.png">


#### step 4: Install all the necessary packeges which are required to build the kernel

         sudo apt-get install git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison
      
<img width="1102" alt="image" src="https://user-images.githubusercontent.com/98585812/206857806-837659bc-2689-4cb1-bb6c-f3df82aa6f57.png">


#### step 5: Copying the current configuration file to a.config file to configure the kernel

         cd linux-6.0.7
         cp -v /boot/config-$(uname -r) .config
         make menuconfig
         make oldconfig
      
<img width="878" alt="image" src="https://user-images.githubusercontent.com/98585812/206857850-e7e76805-92be-41e6-8642-c590f43f9f15.png">
<img width="853" alt="image" src="https://user-images.githubusercontent.com/98585812/206857868-119c74d5-58e3-42d7-82f7-04d0d867876e.png">

#### step 6: Build the kernel using the following commands 

     If there is any error saying "No rule to make target "debian/canonical-certs.pem"" appears while building the kernel, run the following two commands   to fix it. :
           # scripts/config --disable SYSTEM_TRUSTED_KEYS
           # scripts/config --disable SYSTEM_REVOCATION_KEYS
         sudo make -j 512 modules
         sudo make -j 512
         sudo make -j 512 modules_install
         sudo make -j 512 install
      
<img width="879" alt="image" src="https://user-images.githubusercontent.com/98585812/206857908-34720a16-6d6f-4f80-863d-c35f1461e6ab.png">
<img width="854" alt="image" src="https://user-images.githubusercontent.com/98585812/206857914-2d8271a8-c6c4-4c18-bbe0-63003694ba88.png">
<img width="884" alt="image" src="https://user-images.githubusercontent.com/98585812/206857924-8eb40a8f-7fb0-4aff-882f-1ef868066335.png">
<img width="873" alt="image" src="https://user-images.githubusercontent.com/98585812/206857942-5b034134-3644-4d98-9a16-7b5003b98fc4.png">


#### step 7: Once the make is executed, the bootloader will be automatically updated and now we can reboot the system and check the version once the VM has resumed.

         sudo reboot
         uname -mrs
<img width="623" alt="image" src="https://user-images.githubusercontent.com/98585812/206857958-54c968b6-1664-4221-bf00-ce5bc70913f3.png">

#### step 8: Overwrite the kernel code by downloading the vmx.c and cpuid.c from your repository by making the repo public

         vmx.c: /linux-6.0.7/arch/x86/kvm/vmx/vmx.c
         cd /linux-6.0.7/arch/x86/kvm/vmx/
         rm vmx.c
         wget https://raw.githubusercontent.com/souperala/CMPE283-Assignment2/main/vmx.c
         cpuid.c: /linux-6.0.7/arch/x86/kvm/cpuid.c
         cd /linux-6.0.7/arch/x86/kvm/
         rm cpuid.c
         wget https://raw.githubusercontent.com/souperala/CMPE283-Assignment2/main/cpuid.c
      
<img width="733" alt="image" src="https://user-images.githubusercontent.com/98585812/206857984-3620982d-0fed-4dc8-b1b5-3cbac1b94026.png">
<img width="745" alt="image" src="https://user-images.githubusercontent.com/98585812/206857994-3cbc2a49-59c6-4178-8fb2-012f0c9213e0.png">
    
#### step 9: Build the kernel and install the modules with the updated kernel code

         sudo make -j 512 modules && sudo make -j 512 modules_install
         
<img width="742" alt="image" src="https://user-images.githubusercontent.com/98585812/206858027-fd53ee65-7b3b-438e-98bd-cc44d42ee796.png">

#### step 10: Reload the kernel modules following the below commands

         sudo rmmod kvm_intel
         sudo rmmod kvm
         sudo modprobe kvm
         sudo modprobe kvm_intel

<img width="745" alt="image" src="https://user-images.githubusercontent.com/98585812/206858040-83e4266a-1d8f-4a30-adb5-a5e488e10f9b.png">

#### step 11: Create a nested VM in the current GCP VM by following the below steps

- Download the Ubuntu cloud image(QEMU compatible image) in GCP VM from the below ubuntu cloud images site
    
         wget https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img
      
<img width="831" alt="image" src="https://user-images.githubusercontent.com/98585812/206858068-c0399a45-2da0-4a07-b108-6aea64913358.png">
    
- Install the necessary QEMU packages
    
         sudo apt update && sudo apt install qemu-kvm -y
      
<img width="838" alt="image" src="https://user-images.githubusercontent.com/98585812/206858087-0f787d3b-c04c-42b3-a0c0-78d569623888.png">
    
- Go to the folder where the.img file was downloaded. There is no default password for the virtual machine included with this Ubuntu cloud image.
So, use these instructions to modify the password and log in to the virtual machine:

       sudo apt-get install cloud-image-utils
       vi user-data
           #cloud-config
           password: newpass #new password here
           chpasswd: { expire: False }
           ssh_pwauth: True
       cloud-localds user-data.img user-data
       sudo qemu-system-x86_64 -enable-kvm -hda bionic-server-cloudimg-amd64.img -drive "file=user-data.img,format=raw" -m 512 -curses -nographic(Give the username and password to connect to this nested VM)
       
<img width="840" alt="image" src="https://user-images.githubusercontent.com/98585812/206858185-d46c8a77-d2e5-4b6c-bd65-10a08a92bd9b.png">

- New VM has been launched, here is a snapshot of the nested VM

<img width="692" alt="image" src="https://user-images.githubusercontent.com/98585812/206858218-1506e135-dd7d-4004-9426-fe441b5726f2.png">

#### step 12: To check the VM functions correctly, install the cpuid utility by following the below commands
        
          sudo apt-get update
          sudo apt-get install cpuid  
           
<img width="691" alt="image" src="https://user-images.githubusercontent.com/98585812/206858238-c013ac60-37cd-488e-927c-ebdc3df1a9f9.png">
<img width="680" alt="image" src="https://user-images.githubusercontent.com/98585812/206858256-483a4c05-d8b0-4a92-bcd2-c081d0e71f92.png">

#### step 13: open the two terminals(GCP VM "T1", and nested VM terminal "T2") and test the cpuid functionality using the below commands

        Testing the CPUID functionality for '%eax=0x4fffffff'      
        T2: sudo cpuid -l 0x4fffffff
<img width="692" alt="image" src="https://user-images.githubusercontent.com/98585812/206858283-854be588-c7f7-42ff-9e3c-c821402ce401.png">
        T1: sudo dmesg
<img width="693" alt="image" src="https://user-images.githubusercontent.com/98585812/206858301-1ef8d59b-fa94-4791-88e4-53cb7a3cb178.png">

        Testing the CPUID functionality for '%eax=0x4ffffffe'
        T2: sudo cpuid -l 0x4ffffffe
<img width="693" alt="image" src="https://user-images.githubusercontent.com/98585812/206858317-a7931bce-b668-4551-ac2b-403e53c811a4.png">
        T1: sudo dmesg
<img width="668" alt="image" src="https://user-images.githubusercontent.com/98585812/206858323-9025f454-5c92-42de-8be2-cf2ee80729b3.png">

    







      


            
