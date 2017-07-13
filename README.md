# Introduction

This package sets up a DICOM image reconstruction server. It is able to:

- Listen to and handle incoming DICOM transfer
- Process the incoming DICOM image with customized reconsturction script
- Push the processed DICOM image back to the request peer. 
- Back up all the DICOM transfer history to a external storage device on a daily basis. 




# Requirement

- System: (tested on)
	- Ubuntu 14.04 LTS 64bit
	- Ubuntu 15.10 64bit
	- Mac OS X 10.6.8
	- Mac OS X 10.10

- Software:
	- [daemonize](http://software.clapper.org/daemonize/index.html)
	- [dcmtk 3.6.0](http://dicom.offis.de/dcmtk.php.en)

- Reconstruction script:

    This package uses program "medin" for DICOM --> QSM reconstruction.
    
    However, it can be replaced by any command-line reconstruction script whose input/output format is DICOM image, with changes in auto_QSM/src/recon_script accordingly.




# Installation

1. Unzip the package:
   
       $ tar -xvzf auto_QSM.tar.gz
   
   or get it from Github:
   
       $ git clone https://github.com/leonhart217/auto_QSM.git
   
2. cd to the root folder:
   
       $ cd auto_QSM

3. Prepare binary files with `auto_QSM/dep/bin_dep_[Your System Version].tar.gz`:
   
       $ mkdir -p bin
       $ tar -C bin -xvzf dep/bin_dep_[Your System Version].tar.gz
	
   [Note]: if your OS is differnet from those in Requirement, you may need to compile/install the required dependency binary file from source code:
    - average_binary:

        Source code: `auto_QSM/dep/src_dep.tar.gz/average_binary.c`
	- storescp_mod:
		
        Source code: `auto_QSM/dep/src_dep.tar.gz/storescp_mod.cc`
		
        Compiled with dcmtk 3.6.0
	- gdcmdump:
		
        Install [GDCM 2.4](http://gdcm.sourceforge.net/wiki/index.php/Main_Page)

4. (Optional) Add `auto_QSM/bin` to $PATH in order to debug the package from command line

    For example for Ubuntu:
    
        $ echo "export PATH=\$PATH:$(pwd)/bin" >> ~/.profile
        $ . ~/.profile

    or for Mac OS X:
    
        $ echo "export PATH=\$PATH:$(pwd)/bin" >> ~/.bash_profile 
        $ . ~/.bash_profile

5. run

        $ ./install





# Configuration

- System setting: `auto_QSM/src/setup_config`
    - disk_data_fix:
        
        This is recommended to be manually set for the Filesystem of server's system drive ${disk_data}

- User setting: `auto_QSM/etc/config_user`
	- flag_auto_backup:
		
        Automatically backup data from DICOM archive in system drive to external drive.
	    Backup will happen when available space on ${disk_data} is below 80GB, and only happen at 3am ~ 5am, at most once per day.
		
        [Note]: if this option is on, it is recommended to manually set ${mount_backup_fix} in `auto_QSM/src/setup_config`.

- Scanning list: `auto_QSM/etc/scanner_list`

    Information of new scanner should be added as a row with:
	
        [manufactor brand]	[AETitle] [IP address] [port] [relay server name, if needed]

- If any configuration changes are made, restart the server by (Make sure the server is not currently running anything important):

        $ auto_QSM/src/restart_all





# Frequent Q&A
- **Q1**: Do I need to compile the dcmtk and daemonize before installing this package?
    
    **A1**: Compilation and installation of dcmtk and daemonize are recommended. However, one can choose to use pre-compiled binaries for dcmtk which is available online [here](http://dicom.offis.de/dcmtk.php.en). 
    For pre-compiled binary for daemonize, please contact the author.

    [Note]: when using pre-compiled binaries, please make sure the path to the binaries is added to $PATH in crontab.

- **Q2**: How do I check if the package is properly functioning?

    **A2**: Simple check: in Ubuntu or Mac OS, run:

        $ ps -u ${USER} -f | grep auto_QSM

    Typical output has one running instance for each of "qsmrecon_main", "dicom_listen" and "check_log_size". It also has one or more instances for "storescp_mod". If flag_auto_backup is on, it should also has one instance of "auto_backup".

    Full check: more detailed information can be found in the log files:
	    
    `buffer/log_stdout_qsmrecon_main`: log of the recon script (E.g. check if one series is completely received; archive received DICOM; examine the DICOM header information).
	
    `buffer/log_stdout_dicom_listen`: log of DICOM transfer (E.g. check if a port has been used; check if a peer is able to be accessed).
	
    `buffer/log_QSM`: log of successfully reconstructed case(s).

- **Q3**: How do I un-install this package?

    **A3**:	
    1. Backup the data in your DICOM archive ($HOME/data by default)
	2. Run (under auto_QSM)

           $ cat crontab.bak | crontab -
	       $ src/kill_all
	3. Remove the auto_QSM folder

For further questions, please contact the author





# Contact
- Email: zl376@cornell.edu
