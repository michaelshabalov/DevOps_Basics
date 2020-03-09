# Install Git On Windows

### Step 1:
To download the latest version of Git, click on the link below:  
[Download Git for Windows](https://git-scm.com/download/win/)  
Great! Your file is being downloaded.  

### Step 2:
After your download is complete, Run the .exe file in your system.  
![Windows Run Git - Install Git](https://d1jnx9ba8s6j9r.cloudfront.net/blog/wp-content/uploads/2016/11/Windows-Run-Git-Install-Git-Edureka.png)  

### Step 3:
After you have pressed the **Run** button and agreed to the license, you will find a window prompt to select components to be installed.  
![Windows Select Components - Install Git](https://d1jnx9ba8s6j9r.cloudfront.net/blog/wp-content/uploads/2016/11/Windows-Select-Components-Install-Git-Edureka.png)  
After you have made selection of your desired components, click on **Next>**.  

### Step 4:
The next prompt window will let you choose the adjustment of your path environment. This is where you decide how do you want to use Git.  
![Windows Adjusting Path Environment - Install Git](https://d1jnx9ba8s6j9r.cloudfront.net/blog/wp-content/uploads/2016/11/Windows-Adjusting-Path-Environment-Install-Git-Edureka.png)  
You can select any of the three options according to your needs. But for beginners, I recommend using **Use Git From Git Bash Only**

### Step 5:
The next step is to choose features for your Git. You get three options and you can choose any of them, all of them or none of them as per your needs. Let me tell you what these features are:  
![Windows Configuring Extra Features - Install Git](https://d1jnx9ba8s6j9r.cloudfront.net/blog/wp-content/uploads/2016/11/Windows-Configuring-Extra-Features-Install-Git-Edureka.png)  

The first is the option to **Enable file system caching**.  
Caching is enabled through Cache manager, which operates continuously while Windows is running. File data in the system file cache is written to the disk at intervals determined by the operating system, and the memory previously used by that file data is freed.  

The second option is to enable **Git Credential Manager**.  
The **Git Credential Manager** for Windows (GCM) is a credential helper for Git. It securely stores your credentials in the Windows CM so that you only need to enter them once for each remote repository you access. All future Git commands will reuse the existing credentials.  

The third option is to **Enable symbolic links**.  
Symbolic links or symlinks are nothing but advanced shortcuts. You can create symbolic links for each individual file or folder, and these will appear like they are stored in the folder with symbolic link.  

### Step 6:
Choose your terminal.  
![Windows Configuring Terminal Emulator - Install Git](https://d1jnx9ba8s6j9r.cloudfront.net/blog/wp-content/uploads/2016/11/Windows-Configuring-Terminal-Emulator-Install-Git-Edureka.png)  

You can choose one from the options.  

The default terminal of MYSYS2 which is a collection of GNU utilities like bash, make, gawk and grep to allow building of applications and programs which depend on traditionally UNIX tools to be present.  

Or you can choose the window’s default console window (cmd.exe).

### Step 7:
Now you have got all you need. Select Launch Git Bash and click on Finish.  
![Windows Finishing Git Installation - Install Git](https://d1jnx9ba8s6j9r.cloudfront.net/blog/wp-content/uploads/2016/11/Windows-Finishing-Git-Installation-Install-Git-Edureka.png)  

This will launch Git Bash on your screen which looks like the snapshot below:  
![Git Bash Terminal - Install Git](https://d1jnx9ba8s6j9r.cloudfront.net/blog/wp-content/uploads/2016/11/Git-Bash-Terminal-Install-Git-Edureka-1.png)  

### Step 8:
Let us proceed with configuring Git with your username and email.  
In order to do that, type the following commands in your Git Bash:  

```bash
git config --global user.name "<your name>"
git config --global user.email "<your email>"
```

It is important to configure your Git because any commits that you make are associated with your configuration details.  
If you want to view all your configuration details, use the command below:  

```bash
git config --list
```

![Windows Git Configuration List - Install Git](https://d1jnx9ba8s6j9r.cloudfront.net/blog/wp-content/uploads/2016/11/Windows-Git-Configuration-List-Install-Git-Edureka-2.png)
