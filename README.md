swb_repo_cli is command line tool for manipulating SWG repositories. It can build new repositories, extract files (including chain extract, history etc) and list files. Most of the functionality from the SIE repository tool is here - and faster. Most people will probably want to continue using SIE for ease - but this tool is handy if you want to automate something.

## For help run:

./swb_repo_cli -h

## Example usage:

### Create a new .tre from directory:

./swb_repo_cli -b C:/dirtopackage -o D:/output.tre

###  List all files in repository:

./swb_repo_cli -l /

###  List all files in camera/

./swb_repo_cli -l camera/

###  or only listing names:

./swb_repo_cli -l camera/ --names-only

###  or with history and recursive:

./swb_repo_cli -l camera/ --recursive --history

###  or just get history on a single file:

./swb_repo_cli -l camera/freechasecamera.iff --history

### Extraction works just like listing - only you need to specify an output directory.

### Bonus feature - object script generation for SWGEmu

If you want this, you'll know how to use it:

./swb_repo_cli --gen-server-objs object/tangible -o ./output_dir

## Scripting example
 
Thanks to **Sudo** on the MTG discord for this example!

This shell script for WSL packs your tre, kills your client if it's running, copies over the newly packed tre file to the server and client, and restarts the server.

```shell
#!/bin/bash

# Start Core3 after patching?
startCore3="y"

# SWB repo CLI path
toolPath="/mnt/c/Users/Zac/Documents/swb_repo_cli"
toolExe="swb_repo_cli.exe"

# Core3 path
core3Path="$HOME/workspace/Core3/MMOCoreORB/bin"

# Server tre path
treName="patch_custom_001.tre"
trePath="$HOME/workspace/tre/"

# Windows paths
inPath="C:/Users/Zac/Desktop/tre/patch_custom_001"
outPath="C:/Users/Zac/Desktop/tre/$treName"

# Game path
swgPath="C:/SRDev"
swgExe="SWGEmu.exe"

# Linux paths converted from Windows paths
linInPath="$(wslpath -a "$inPath")"
linOutPath="$(wslpath -a "$outPath")"
linSwgPath="$(wslpath -a "$swgPath")"

# SWG Process ID
swgPID=$(/mnt/c/Windows/System32/tasklist.exe | grep $swgExe | awk '{print $2}')

function getSWG() {
  if [[ $swgPID == "" ]]; then
    return 1
  else
    return 0
  fi
}

function killSWG() {
  /mnt/c/Windows/System32/taskkill.exe /IM $swgExe /F
}

function startCore3() {
  cd $core3Path;./core3
}

if getSWG; then
  killSWG
  sleep 1
fi

cd "$toolPath/"
./$toolExe -o $outPath -b $inPath

if [ -f $linOutPath ]; then
  rsync -avI "$linOutPath" "$linSwgPath/$treName"
  rsync -avI "$linOutPath" "$trePath/$treName"
  if [[ $startCore3 == "y" ]]; then
    startCore3
  fi
else
  echo "An error occurred building your tre file as it does not exist!"
fi
```
