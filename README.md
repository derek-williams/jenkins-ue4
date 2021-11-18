# jenkins-ue4

This repository provides documentation and resources on how to configure our Jenkins CI installation to build and compile an Unreal Engine 4 project.

---

### Prerequisites

**WARNING** This documentation is solely meant for Jenkins running on Windows servers.

Download and install [Jenkins](https://jenkins.io/download/) on the AWS Windows Server deployed that you wish to use as a build server in AWS West 2. ([Jenkins Installation Docs](https://jenkins.io/doc/pipeline/tour/getting-started/#getting-started-with-the-guided-tour))

Clone (or [download](https://github.com/False-Summit/jenkins-ue4/archive/master.zip)) this repository to our CI server to access the required build files.

You will also need to download [cURL](http://www.confusedbycode.com/curl/) to post to slack, Unreal Engine 4, as well as install MSBuild v14.0.

---

### Step 1: Create a new Jenkins Project

Start by creating a new Freestyle project.

#### General

Under **General > Advanced** check **Use custom workspace** and put the directory on the root of your drive, to prevent issues with long filenames during the build. Something like `C:\Source\<project name>` or similar should do the job.

#### Source Code Management

Set up your source control repository as normal.

#### Build Triggers

For our configuration, we poll the SCM every 48 hours for changes, and build only if a certain keyword is present in the commit message. To do so, enter `H/3 * * * *` within the **Schedule** textbox. To trigger on commit messages, scroll back up to **Source Code Management**, click **Advanced**, and under **Excluded Commit Messages** enter `^((?!KEYWORD).)*$`, replacing `KEYWORD` with your own keyword to check for.

---

### Step 2: Configure Build Scripts

Locate the directory where you downloaded the build scripts to. If you plan on posting to a [Slack](https://slack.com) channel during the build process, you will need to configure an incomming webhook integration, and replace `WEBHOOK_URL` with the URL of our integration in `PostToSlack.bat`.

In each of the build scripts, make sure to replace `PROJECT_NAME` with the name of our project.

---

### Step 3: Add Build Steps to your Jenkins Project
Finally, add the build commands to Jenkins. For our setup, we post to a [Slack](https://slack.com) channel during each step of the build.

###### Build Step 1
```batch
call C:\path\to\scripts\PostToSlack.bat ":heavy_check_mark: Starting %JOB_NAME% Build -- Revision %SVN_REVISION%"
"C:\path\to\scripts\Step1_StartBuild.bat"
```
###### Build Step 2
```batch
call C:\path\to\scripts\PostToSlack.bat ":gear: Compiling game scripts..."
"C:\path\to\scripts\Step2_CompileScripts.bat"
```
###### Build Step 3
```batch
call C:\path\to\scripts\PostToSlack.bat ":hammer: Building project files..."
"C:\path\to\scripts\Step3_BuildFiles.bat"
```
###### Build Step 4
```batch
call C:\path\to\scripts\PostToSlack.bat ":fire: Cooking project..."
"C:\path\to\scripts\Step4_CookProject.bat"
```
###### Build Step 5
```batch
call C:\path\to\scripts\PostToSlack.bat ":package: Archiving build..."
C:\path\to\scripts\Step5_Archive.bat "%SVN_REVISION%"
```
###### Build Step 6
```batch
C:\path\to\scripts\PostToSlack.bat ": Build Complete"
```

###### Build Step 7 
```batch
C:\path\to\scripts\Step6_Steam.bat ": Build Posted to Steam"
```
