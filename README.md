# BasicPXE

## Commands Run in the video

### Sources:
- [iPXE](https://ipxe.org/download)
- [Ubuntu Server](https://ubuntu.com/download/server)
- [Windows PE](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/download-winpe--windows-pe?view=windows-11)
- [Windows 10 ISO](https://www.microsoft.com/en-us/software-download/windows10ISO)

## Setting up Ubuntu VM

1. Create a new Ubuntu VM.
2. Run the following commands:
    ```bash
    sudo apt update
    sudo apt install tftpd-hpa git
    nano /etc/default/tftpd-hpa
    ```
3. Modify the configuration file `/etc/default/tftpd-hpa` as follows:
    ```
    # /etc/default/tftpd-hpa
    
    TFTP_USERNAME="tftp"
    TFTP_DIRECTORY="/srv/tftp"
    TFTP_ADDRESS=":69"
    TFTP_OPTIONS="--secure"
    ```
4. Clone iPXE repository and build:
    ```bash
    git clone https://github.com/ipxe/ipxe.git
    cd ipxe
    sudo apt install gcc binutils make perl liblzma-dev mtools mkisofs syslinux -y
    make bin-x86_64-efi/ipxe.efi
    ```
5. Configure `autoexec.ipxe`:
    ```bash
    cd /srv/tftp/ipxe
    sudo nano autoexec.ipxe
    ```
    Make the file look like this:
    ```ipxe
    #!ipxe
    
    dhcp
    set menu-timeout 5000
    set submenu-timeout ${menu-timeout}
    
    isset ${menu-default} || set menu-default exit
    set boot-url tftp://
    
    ######## MAIN MENU ###################
    :start
    menu Welcome to iPXE's Boot Menu
    item
    item BootHardDisk Boot from Hard Disk
    item iPXECommandLine iPXE Shell
    choose --default exit --timeout 8000 target && goto ${target}
    ########## End ####################
    
    :iPXECommandLine
      echo Entering iPXE command line...
      shell
    
    :BootHardDisk
    exit
    ```
6. Configure DHCP server with:
    - Option 66: `192.168.1.36` (put your tftp server IP here)
    - Option 67: `ipxe/ipxe.efi`
    Restart DHCP.

## Setting up Windows PE

1. Download and get required software to create Windows PE from [Microsoft](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/download-winpe--windows-pe?view=windows-11).
2. Start the Deployment and Imaging Tools Environment as an administrator.
    ```cmd
    cd "..\Windows Preinstallation Environment\amd64"
    md C:\WinPE_amd64\mount
    Dism /Mount-Image /ImageFile:"en-us\winpe.wim" /index:1 /MountDir:"C:\WinPE_amd64\mount"
    notepad C:\WinPE_amd64\mount\Windows\System32\startnet.cmd
    ```
3. Add the following:
    ```
    powercfg /s 8c5e7fda-e8bf-4a96-9a85-a6e23a8c635c 
    net use Y: \\nas\Public\Windows10
    Y:
    setup.exe /unattend:Unattend.xml
    ```
    Save and close Notepad.
4. Copy necessary files:
    ```cmd
    Xcopy "C:\WinPE_amd64\mount\Windows\Boot\EFI\bootmgr.efi" "Media\bootmgr.efi" /Y
    Xcopy "C:\WinPE_amd64\mount\Windows\Boot\EFI\bootmgfw.efi" "Media\EFI\Boot\bootx64.efi" /Y
    Dism /Unmount-Image /MountDir:"C:\WinPE_amd64\mount" /commit
    rmdir /S /Q C:\WinPE_amd64
    copype amd64 C:\WinPE_amd64
    MakeWinPEMedia /ISO C:\WinPE_amd64 C:\WinPE_amd64\WinPE_amd64.iso
    ```

## Testing

- Test the iPXE boot.

## Additional Notes

- Make or download your Windows 10 install ISO and place it on your network share.
- [Windows 10 ISO](https://www.microsoft.com/en-us/software-download/windows10ISO)

