---
title: "Auto file updater"
---
# Auto File Replicator
#### by Graham Ward
#### Programming language: Go version 1.13.8
#### Compiled/run on Ubuntu 20.04 Linux 64 Bit / Windows 10
#### Version 1.1.0


## Overview
The purpose of this program is to monitor and automatically replicate a specified local file on a remote computer. When a change or write occurs to the file in question, an event is immediately triggered. The event then copies the source file to the designated target destination. The target destination can be any computer, server, Raspberry Pi etc. that is network accessible from the local (source) computer.

Multiple instances of the program may be run concurrently in order to monitor and update different files. You'll have to create different directories for each copy of the program to accomodate the configurations (the IP address and port obviously remain the same for each config file, without incurring any SSH conflicts). Note I could have added some additional facility to monitor an entire directory (recursively) but for my requirement this wasn't necessary. I also felt that for larger projects it may be prudent to not be copying all the files that are not changing, and instead focus on only replicating the files that do change.

This program was written to help me make software development on the Raspberry Pi simpler. IE although I do regularly cross compile code, there are times when what I am developing simply will not cross compile and has to be compiled natively on the Raspberry Pi itself. For this reason I developed this application that will monitor my source file as I write code, and as soon as I save it, automatically copy it over (using SFTP over an SSH connection with full authentication/encryption). I can then turn to the Raspberry Pi and compile the code just copied over to it. The simple yet tedius task of manually copying and pasting the file over a mapped network drive (or frequent use of an application like Filezilla) is thus mitigated.

The program should have no problem replicating files of any kind across a network. It's use is not limited to text files.

The code is written in Golang and serves as a working example of how to get SSH communications up and running, along with SFTP, using Go. It utilizes a watcher that relies on an OS API to monitor for any file changes that may occur. This minimizes program overhead/resource use.
the code is tested/verified to compile and run in both Windows 10 (with SSH capability) and Ubuntu Linux. A note to windows users: As the configuration data is unmarshalled in Go, a shortcoming of the Go process is that it treats backslash '\' as an escape sequence. Attempting to use s single backslash in your configuration will result in a JSON unmarshalling fatal error. To avoid this either escape the backslash by using a double backslash (eg C:\\users\\username\\Documents) or use forward slashes '/' instead (The application will function just fine on a Windows system when using forward slashes). A third option is to upgrade to Linux...:)

In order to ensure secure communications, SSH keys need to be manually generated and copied over to the target device as well. The process is relatively simple though.
Follow the instructions below (or see the resource link for more information) to manually generate and deploy the required keys

Finally, before proceeding, ensure that you have SSH connectivity enabled on both local and remote machines, and you can successfully connect in a terminal window.

#### Linux
1. Verify whether or not the keys exist already. You should see **id_rsa** private and **id_rsa.pub** public key files listed when you execute:
   
      <span style="color:green">**ls ~/.ssh/id_***</span>

2. If the keys do not exist, create SSH **id_rsa** private and public keys.
         Note: email address is simply a comment line entered into the generated key file and has no action or effect.
               Refer to resource below for more information.
               I set no password when prompted, however you can, should you deem it necessary.

      <span style="color:green">**ssh-keygen -t rsa -b 4096 -C "your_email@domain.com"**</span>

3. Confirm keys now exist (should see **id_rsa** private and **id_rsa.pub** public key files):
   
      <span style="color:green">**ls ~/.ssh/id_***</span>

4. Copy keys over from local host to remote server or Raspberry Pi etc.:
   
      <span style="color:green">**ssh-copy-id remote_username@server_ip_address**</span>


#### Windows

1. Confirm OpenSSH Client is installed under settings -> Application -> Manage optional applications

2. Open CMD (run as adminisitrator by searching for **cmd** in start menu, then right clicking for option)

3. Generate the required ssh keys using the following command. When prompted, enter nil for filename and password (optional, but I don't use it).
    The applicable keys will be generated under the respective user as private (C:\Users\username\\.ssh\id_rsa) and public (id_rsa.pub) keys:
    When the keys have been successfully generated, a key fingerprint is displayed as well as a "randomart" image
    Note you may be asked if you wish to overwrite existing keys. This scenario falls outside the scope of these instructions.
    
    <span style="color:green">**ssh-keygen**</span>

4. Copy the SSH keys over using the following commands (replace the IP address and user-name as required)

    <span style="color:green">**scp /users/gward/.ssh/id_rsa graham@192.168.1.126:/home/user-name/.ssh**</span>
    
    <span style="color:green">**scp /users/gward/.ssh/id_rsa.pub graham@192.168.1.126:/home/user-name/.ssh**</span>

5. On the remote (assumed to be linux or Raspberry Pi) machine check the keys that currently exist in the SSH authrorized_keys by using this command:

    <span style="color:green">**more ~/.ssh/authorized_keys**</span>

6. Add the new keys using these commands (Use the previous command every time to confirm addition of keys)
    

    Note:

    1. You may need to adjust source path to suit here.
   
    2. A Windows based target or remote system is beyond the scope of these instructions


    <span style="color:green">**cat ~/.ssh/id_rsa >> ~/.ssh/authorized_keys**</span>
    
    <span style="color:green">**cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys**</span>

## Operation

1. The application is run from the command line. Either compile from source or use the ready made (Linux 64Bit) executable (replicator) or Win10_Replicator.exe if you're a windows user. To execute simply go to the installation directory and type in:
   
    Linux: &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;  **./replicator**

    Windows 10: &nbsp; **Win10_Replicator**

2. When the program is first started it will look for a configuration file. If one is not found it will auto-generate a new one with default settings.
   You will need to edit the default config (JSON) file to suit your particular requirements (source and tartget directories and files).
   The target file name is optional. If ommited the program will default to using the source file name.

3. Once that is done you are free to restart the program which will then display your current configuration. If there are any errors at any point the program will immediately abort.

4. Once it gets to the point of successful connection, try making a change to your source file and confirm it is immediately copied to the target destination. Feedback is provided in the terminal window as well, where you will see a tally of the number of updates and how many bytes were copied over.

5. As the instructions state, hit **CTRL+C** to quit once done.

## Resources:

https://linuxize.com/post/how-to-set-up-ssh-keys-on-ubuntu-20-04/

https://serverfault.com/questions/309171/possible-to-change-email-address-in-keypair

https://skarlso.github.io/2019/02/17/go-ssh-with-host-key-verification/

https://kb.iu.edu/d/aews

https://phoenixnap.com/kb/generate-ssh-key-windows-10
