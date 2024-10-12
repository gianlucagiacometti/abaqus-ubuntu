# Abaqus 2023/2024 on Ubuntu 24.04

## Intro

Ubuntu seems to not be officially supported by the Abaqus installation procedure. This guide shows how to install the necessary libraries and how to tweak the installation files in order to install Abaqus on Ubuntu 20.04. To successfully follow this guide you need writing privileges ('sudo').

## Install prerequisites

The standard Ubuntu release might not have one or more of the following libraries needed by Abaqus:

To install them open a terminal and execute the following command:

```bash
sudo apt update
sudo apt install csh tcsh ksh gfortran build-essential libmotif-dev
```

## Modify the Linux.sh file

Since Ubuntu is not officially supported, trying to install Abaqus will result in an error. In order to fix it, prior launching
the installation, locate all the **Linux.sh** files in the Abaqus installation folders, delete their content and past the following
in each of them:

path_to_abaqus_installation_folder/1/inst/common/init/Linux.sh

path_to_abaqus_installation_folder/3/NETVIBES_Exalead_CloudView/Linux64/1/inst/common/init/Linux.sh

path_to_abaqus_installation_folder/3/SIMULIA_FLEXnet_LicenseServer/Linux64/1/inst/common/init/Linux.sh

path_to_abaqus_installation_folder/3/Search_Doc/Linux64/1/inst/common/init/Linux.sh

path_to_abaqus_installation_folder/4/SIMULIA_EstablishedProducts/Linux64/1/inst/common/init/Linux.sh

path_to_abaqus_installation_folder/5/SIMULIA_Documentation/AllOS/1/inst/common/init/Linux.sh

path_to_abaqus_installation_folder/5/SIMULIA_EstablishedProducts_CAA_API/Linux64/1/inst/common/init/Linux.sh

path_to_abaqus_installation_folder/5/SIMULIA_Isight/Linux64/1/inst/common/init/Linux.sh

```sh
  DSY_LIBPATH_VARNAME=LD_LIBRARY_PATH

  which lsb_release
  if [[ $? -ne 0 ]] ; then
    echo "lsb_release is not found: check in the PDIR the list of installed packages for servers validation."
    exit 12
  fi

  DSY_OS_Release="CentOS" #Override release setting, old: DSY_OS_Release=`lsb_release --short --id |sed 's/ //g'`
  echo "DSY_OS_Release=\""${DSY_OS_Release}"\""
  export DSY_OS_Release=${DSY_OS_Release}
  export DSY_Skip_CheckPrereq=1 #Added to avoid prerequisite check

  if [[ -n ${DSY_Force_OS} ]]; then
    DSY_OS=${DSY_Force_OS}
    echo "DSY_Force_OS=\""${DSY_Force_OS}"\", use it for DSY_OS"
    return
  fi

  case ${DSY_OS_Release} in
      "RedHatEnterpriseServer"|"RedHatEnterpriseClient"|"RedHatEnterpriseWorkstation"|"CentOS")
          DSY_OS=linux_a64;;
      "SUSELINUX"|"SUSE")
          DSY_OS=linux_a64;;
      *)
          echo "Unknown linux release \""${DSY_OS_Release}"\""
          echo "exit 8"
          exit 8;;
  esac
```

Note the changes: 1) the release version was forced to be `"CentOS"`, and 2) checking for prerequisites was disabled.

### Modfication for Abaqus 2024
Instead of Linux.sh the script CheckPrereq.sh (*/inst/common/checkers/Linux/CheckPrereq.sh) has to be changed. 
Bevor exporting DSY_OS_Release it hat to be manipulated to trick the install routine into centos. 

```sh
DSY_OS_Release="centos"
export DSY_OS_Release
```
NOTE: Please change 2023 to 2024 in the following instructions if you have Abaqus 2024

## Run setup
The setup of Abaqus requires a `bash` instead of a `dash` shell, whereas the latter is the default in Ubuntu. To temporary change that we symlink the shell command to `bash` by:
```dash
sudo ln -sf bash /bin/sh
```
which we'll undo after the installation.

Once all the prerequisites are installed and the installation files modified, it is possible to proceed with the installation:
```bash
cd path_to_abaqus_installation_folder/1
```
either (if the server has no x11 installed)
```
sudo ./StartTUI.sh
```
or (if the server has x11 installed)
```
sudo ./StartGUI.sh
```
otherwise

this will start the installation for all the Abaqus related products. Use standard locations for everything.
```bash
/usr/SIMULIA/EstProducts/2023
```
When it asks for the license skip it "Skip licensing configuration" (otherwise it will cause an error) and proceed with the installation using the default locations for the software:
```bash
/var/DassaultSystemes/SIMULIA/Commands
/var/DassaultSystemes/SIMULIA/CAE/plugins/2023
```

Now that the installation is complete we revert the standard shell back to dash:
```bash
sudo ln -sf dash /bin/sh
```

## License Activation.
The exact way to setup a license depends on the type of license that you have. FlexNET or DSLS.

### FlexNET
To specify a FlexNET server, execute:
```sh
sudo gedit /usr/SIMULIA/EstProducts/2023/linux_a64/SMA/site/custom_v6.env
```

and then change the license server type to FLEXNET and the abaquslm_license_file to the one of your license

```sh
# Installation of Established Products 2023

# Day Month date hh:mm:ss yyyy
  plugin_central_dir="/var/DassaultSystemes/SIMULIA/CAE/plugins/2023"
  license_server_type=FLEXNET
  abaquslm_license_file="<port>@<your_domain>"
```
where `<port>` is the port used on the license server, and `<your_domain>` the domain of the license server.

### DSLS
To specify a DSLS server, execute:
```sh
sudo gedit /var/DassaultSystemes/Licenses/DSLicSrv.txt
```
and add to this file the line:
```sh
  <your_domain>:<port>
```
where `<port>` is the port used on the license server, and `<your_domain>` the domain of the license server.

## Make abaqus command available from any directory

In order to run Abaqus from any location, execute the following command: 
```sh
sudo ln /var/DassaultSystemes/SIMULIA/Commands/abq2023 /usr/bin/abaqus
```

## Create temp directory for tosca

```sh
sudo mkdir /usr/temnp
chmod 777 /usr/temp
```

## Credits

This guide is based on:

* <https://github.com/franaudo/abaqus-ubuntu>
* <https://gist.github.com/cmaurini/60fab556ba4a43bd44341a1af114dde7>
* <https://github.com/Kevin-Mattheus-Moerman/Abaqus-Installation-Instructions-for-Ubuntu>
* <https://github.com/imirzov/Install-Abaqus-2019-on-Ubuntu-18.04-LTS>
