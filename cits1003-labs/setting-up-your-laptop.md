# Lab 1: Setting up your laptop

We will set up various software that will be used in the labs, the principle one being Docker Desktop. However, it is a good idea to create a folder specifically for organising the different week's labs.&#x20;

## Windows users only: Installing Windows Subsystem for Linux (WSL)

This is a necessary step for the unit and also for running Docker Desktop. There are instructions for this on the web e.g. here: [https://andrewlock.net/installing-docker-desktop-for-windows/](https://andrewlock.net/installing-docker-desktop-for-windows/)

To get started, you need to launch a command prompt in Administrator mode. Search for cmd and then right click on the command prompt and select run as Administrator.

![Running Command Prompt as Administrator](<../.gitbook/assets/Screen Shot 2021-06-30 at 10.12.47 am.png>)

Then enter the following commands:

> dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart&#x20;
>
> dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

Now type powershell to get a powershell prompt and continue with the commands:

> Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform -NoRestart
>
> wsl --set-default-version 2

From the Windows Store, search for and install: Ubuntu 20.04

After this, restart windows.&#x20;

To test this out, type wsl in the search bar and run the command prompt.&#x20;

{% hint style="info" %}
There are a number of things that can go wrong doing this installation.

You can't find cmd.exe or PowerShell:

1. PowerShell can be installed from the Microsoft Store.
2. You may be running in Windows S mode that will prevent you running and installing apps - to deactivate this feature: To **turn off Windows** 10 **S Mode**, click the Start button then go to Settings > Update & Security > Activation. Select Go to the Store and click **Get** under the Switch **out of S Mode**

You encounter problems installing and running WSL 2

1. Follow the instructions here [https://docs.microsoft.com/en-us/windows/wsl/install-win10](https://docs.microsoft.com/en-us/windows/wsl/install-win10) the kernel update is the key thing

If Docker Desktop fails to start

1. Clean up data and try again
{% endhint %}

## Running Windows 10 on Azure

If you are unable to get your laptop/PC working, another option is to run Windows 10 on a Virtual Machine on Azure. To do this, you will need a student account created on https://portal.azure.com/.

You can create a VM using Windows 10 Pro 21 H1 and pick a Standard\_D2s\_V3 machine. Use all of the default settings but select Australia as the region to run it in (if you are located internationally, pick a region close to you).

Once created, you can connect to the machine via remote desktop and then configure the machine as above.&#x20;

{% hint style="info" %}
Although you have credit when creating a student account, be careful with the machine and stop it running by using the console when you are not using it - that way you will not be charged for the time you are not using it.&#x20;
{% endhint %}

## Apple Mac M1 (Apple Silicon) Users: Enable Rosetta

Apple's computers are increasingly using the new M1 chip that uses a different instruction set than the Intel-based Macs. Apple allows programs built for the Intel chip to run by using an emulator called Rosetta 2. If you have not already installed it, then:

1. Open a Terminal window
2. Type (paste) the command `/usr/sbin/softwareupdate --install-rosetta --agree-to-license`

Once this is done, you can proceed with installing and running Docker Desktop

Whilst all of the Docker images in the labs can be run on the Apple M1, there may be warnings given about the platform (you can avoid this warning by passing the argument --platform linux/amd64). I have created specific versions of Docker Images for the Apple M1 and where these are available, you can run those in preference. Look at each lab for details but generally, they will have a '-x' on the end of the name.

## Installing and running Docker Desktop

We will be using a technology called Docker Desktop to run different environments on your laptop. Unfortunately, this environment will not be available on the lab machines and so we will try and provide an alternative for people who want to use the lab machines.&#x20;

You can get a more comprehensive overview of what Docker is from here [https://docs.docker.com/get-started/overview/](https://docs.docker.com/get-started/overview/). To summarise though, Docker allows you to "package and run an application in a loosely isolated environment called a container". Containers are a way of virtualizing an environment by using the native operating system's functionality to isolate application environments.

The process for installing Docker Desktop is straightforward and involves using the installer for the particular laptop you have:

{% embed url="https://www.docker.com/get-started" %}

To test the environment, we will run a simple container that allows you to access a bash terminal. This allows you to enter commands that get executed within the container. You can only do what the container will let you do as it is a constrained environment.&#x20;

To start with, make sure that your Docker Desktop application is running. Once it is, open a terminal window, PowerShell or Command prompt and run the following commands

{% tabs %}
{% tab title="Windows" %}
```bash
PS C:\Users\david\Documents\cits1003> docker pull cybernemosyne/cits1003:bash
bash: Pulling from cybernemosyne/cits1003
345e3491a907: Pull complete
57671312ef6f: Pull complete
5e9250ddb7d0: Pull complete
5fac5b10dc75: Pull complete
164545966a0f: Pull complete
346af9298e18: Pull complete
4beda6fc08a5: Pull complete
Digest: sha256:691832551b79cd93e5592b54d234b89d23b253336934677e311e64eefc8b958b
Status: Downloaded newer image for cybernemosyne/cits1003:bash
docker.io/cybernemosyne/cits1003:powershell
PS C:\Users\david\Documents\cits1003> docker run -it cybernemosyne/cits1003:bash
root@45fe3a838ef0:/# whoami
root
root@45fe3a838ef0:/# 

```
{% endtab %}

{% tab title="Mac OSX" %}
```bash
MyComputer:~$ docker pull cybernemosyne/cits1003:bash
Digest: sha256:691832551b79cd93e5592b54d234b89d23b253336934677e311e64eefc8b958b
Status: Image is up to date for cybernemosyne/cits1003:bash
cybernemosyne/cits1003:bash

0x4447734D4250:~$ docker run -it cybernemosyne/cits1003:bash
root@45fe3a838ef0:/# whoami
root
root@45fe3a838ef0:/# 
```
{% endtab %}

{% tab title="Apple Silicon" %}
```
MyComputer:~$ docker pull cybernemosyne/cits1003:bash-x
Digest: sha256:691832551b79cd93e5592b54d234b89d23b253336934677e311e64eefc8b958b
Status: Image is up to date for cybernemosyne/cits1003:bash-x
cybernemosyne/cits1003:bash

0x4447734D4250:~$ docker run -it cybernemosyne/cits1003:bash-x
root@45fe3a838ef0:/# whoami
root
root@45fe3a838ef0:/# 
```
{% endtab %}
{% endtabs %}

The _**docker pull**_ command downloads the docker image to your machine. The image contains all of the files and configurations needed to run the container. You run a container using the _**docker run**_ command as shown above.&#x20;

In the case of the bash container, to stop it, you simply type _**exit**_. Other containers can be stopped using the _**docker stop**_ command from another terminal. To do this, you need to provide the Container ID which you can do as follows:

```bash
0x4447734D4250:~$ docker ps
CONTAINER ID   IMAGE                         COMMAND       CREATED         STATUS         PORTS     NAMES
45fe3a838ef0   cybernemosyne/cits1003:bash   "/bin/bash"   3 minutes ago   Up 3 minutes             hungry_hodgkin
0x4447734D4250:~$ docker stop 45fe3a838ef0
45fe3a838ef0
```

Once you have finished with a container, you can remove the image that was downloaded using the Docker Desktop GUI. Remember that anything you have done in the container will be lost when you remove the container.&#x20;

We will be using containers in the various labs and so you will learn more about using Docker and how containers work generally as we proceed.&#x20;

## Question 1. Find your first flag

Go back to the bash docker container. There is a file called flag.txt hidden somewhere. Can you find it?&#x20;

{% tabs %}
{% tab title="" %}
Click on Hint tab to reveal solution
{% endtab %}

{% tab title="Hint" %}
{% hint style="info" %}
To find the file, we will first go to the home directory of the user root by using the&#x20;

> cd /root

command. This will change the current directory to /root

Once there, we can list the contents of that directory by using the **ls** command

> ls -al&#x20;

There will be a file called flag.txt in the directory. We can view the contents of the file by using the **cat** command:

> cat flag.txt
{% endhint %}
{% endtab %}
{% endtabs %}

**Flag: Submit the flag that you got when you executed the cat flag.txt command**

## Final Thoughts

There is other software that you will need to install as we progress through the labs but we will deal with those as they arise.
