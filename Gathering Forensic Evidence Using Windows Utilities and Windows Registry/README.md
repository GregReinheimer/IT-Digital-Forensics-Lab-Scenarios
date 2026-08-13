# Gathering Forensic Evidence Using Windows Utilities and Windows Registry

## Objective

Using Windows utilities to inspect active processes, network activity, and file system changes, then explore the Windows Registry to gather system and user-related forensic evidence. 



## Business Scenario

>Over the past week, several employees report unusual behavior on their workstations—applications freezing, unexpected pop‑ups, and slow network performance during off‑hours. At the same time, the company’s monitoring systems flag irregular login patterns from a workstation that normally shows minimal activity. The device appears to be communicating with external servers that are not part of any approved business workflow.
>Management becomes concerned that the workstation may have been compromised, potentially exposing sensitive internal data or providing a foothold for unauthorized access. To determine whether the activity is legitimate, accidental, or malicious, the IT security department is tasked with conducting a detailed investigation into the system’s behavior, recent changes, and any traces of user or application activity that could explain the anomalies.

This scenario is a typical situation for the IT security department to investigate and provide outcomes.




## Environment and Tools Used

| Component | Detail |
|---|---|
| Server | Windows 2019 (VM workstation)|
| Test Image | Windows 10 drive image |
| Test Scope | Test vm only (No production machine or real user data) |
| Live System Analysis | Task Manager, Resource Monitor, Fsutil |
| Windows Registry Analysis | Registry Editor (Regedit) |

## ***Steps Performed***

### Part 1: Gather Basic System Information

#### Step 1.1: Open Task Manager, add the command line column, and identify a core process in the properties window

   >**Note**: From the Task Manager, you can review all the application, processes, and services that are active at the           moment. You can even attempt to shut down specific processes, if necessary.
   
From the workstation desktop, **right click** anywhere on the taskbar, then select **Task Manager** from the context menu to open the Task Manager. In the Task Manager, **right click** the **Name Column header**, then **select Command line** from the context menu to add the Command line column. **Identify** one of the core Windows processes listed above, then **right click** the process and **select properties** from the context menu to display additional information about the process in the Properties window. Then close the properties window.

  (PICTURE)

   >**Note**: By adding the Command line column, you can observe the file path for each process's executable file, as well as       any command line options that were used to launch the program. For forensic investigators it can be a valuable source of  detailed information when attempting to analyze and isolate malware that has been running since the start of the system.

---
#### Step 1.2: Check network activity by viewing the performance tab, resource monitor, and the listening ports on the network tab

In the Task Manager, **click** the **Performance tab** to display information about resource usage. From the bottom of the Task Manager, **click** the **Open Resource Monitor link** to launch the Resource Monitor. In the Resource Monitor, **click** the **Network tab** to display a list of processes that are currently generating network activity. On the Network tab, **expand** the **Listening Ports header** to display a list of ports that are actively listening and the processes that are using them. Close the **Resource Monitor** and **Task Manager windows**.

  (PICTURE)

  >**Note**: The Resource Monitor is a dedicated utility for monitoring system resource usage in real time, including CPU, memory usage, disk utilization, and network use. When the Resource Monitor contains many of the same metrics as the         Performance Tab, it allows knowledgeable users to drill deeper into the underlying data.

 >**Note**: The Network tab displays network activity, TCP connections, and listening ports for any running process. As a         forensic investigator, this information can be essential to identifying spyware, botnets, and other forms of malware that   require network connectivity.

---
#### Step 1.3: Gather NTS drive information

Open the **Command Prompt** and type `fsutil fsinfo ntfsinfo c:` and **press enter** to display information about the c: drive.

    (PIC)

   >**Note**: The output of this command provides information about the c: drive, inlcuding the NTFS volume serial number,       version, number of total clusters, number of free clusters, total number of sectors, and more. In forensic investigations, this information can be used to determine if there is hidden data in a boot file or boot record of the drive. Also it can be used to determine if there are fake bad clusters, which are areas of the drive that the file system ignores due to damage, but are in fact viable and used to hide data.

---
#### Step 1.4: Review the usn journal

At the command prompt, type `fsutil usn queryjournal c:` and **press enter** to display information related to the update sequence number changes journal. Then, type `fsutil usn readjournal c:` and **press enter** to display all records in the usn journal.
>After allowing the command to run for about 10 seconds, **press ctrl+c** to terminate it.

    (PIC)

  >**Note**: This information can be useful for several reasons. First, when a program is run, typically you will be able to       see the modified prefetch files, which will give you a timestamp for execution. Second, the usn journal can provide         evidence of deleted files. Third, you can see file modifications and creation of specific particular file extensions, such as executbale, which in some contexts can be indicative of a system compromise. Many compromises occur after a piece of file-based malware is initially run, like from a macro or PDF. The usn journal can provide a timeline of           initial infection.

---
#### Step 1.5: Generate a new usn record

On the desktop, **right click** on empty space and **select New → Text Document** to create an empty text file on the desktop. In the filename field, type `yourname` where your name is your own name, and **press enter** to name the new file. At the command prompt, type `cd desktop` and **press enter** to change the current directory to the desktop. Then type `fsutil usn readjournal c: csv > usn.log` and **press enter** to save the contents of the usn journal to the workstation desktop as a .csv file. **Select** the **usn.log file** to open it in Notepad. In the notepad window, **scroll down** to the bottom of the file and **locate** the record referencing the **yourname.txt file**. **Highlight** the **File ID value** for the yourname.txt file, then **press ctrl+c** to copy it to the clipboard.
At the command prompt, type `fsutil file queryFileNameById c:\ 0xFile ID`, where File ID is the value you copied from the previous command output (paste), then **press enter** to retrieve the file path associated with the file ID.
>The file ID should be the long hexadecimal number directly to the right of the word "Archive".

  (PICTURE)

  (PICTURE)

  
---
### Part 2: Explore the Registry
---

  >**Note**: The Windows Registry is one of the key components of the Windows OS. This database contains Windows settings, application settings, device driver info, and user passwords. When an application is installed in a Windows environment, some part of the software is stored in the Registry. Registry analysis is often a critical component of conducting forensics investigations on Windows systems.

---
#### Step 2.1: Open the registry editor to locate the current version key

From the taskbar, **click** the **start icon** and type `regedit` and **press enter** to search for and launch the Registry Editor. Navigate to the following key: `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\WindowsNT\CurrentVersion`

>This key contains multiple data points that could be very useful to an investigator who has obtained a drive image  as evidence, but doesn't know the exact version of Windows that was installed on it. If the drive image was taken from a recent compromised system, this information could help corroborate evidence of an exploit.

---
#### Step 2.2: Review the installation details and convert the timestamp to human readable format

In the right pane, **select** the **InstallDate value** to open the Edit DWORD Value Box (32 bit). In the dialog box, **right click** the **Value data field**, then **select copy** from the context menu to copy the hex value to the clipboard. Then close the box. **Select Chrome icon**, to open a new Chrome browser window. In the navigation bar, type `epochconverter.com/hex` and **press enter** to navigate to the hexadecimal converter. In the hexadecimal converter, **delete** the existing timestamp value, then **right click** the empty hex field and **select paste** from the context menu to paste the hex value into the converter. **Click** to perform the conversion. Close the browser.

  >**Note**: Value data in the Windows Registry is commonly stored in the hexadecimal notation.

  (PICTURE)

  ---
#### Step 2.3 : Collect the required registry keys for the network interfaces and the windows login function.
Note: Although these keys show the general idea of what they are, they may not be exact based on the version of the OS.

In the Registry Editor, navigate to the following keys:

  -  `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces\{39007...}`

>This key contains information about the default network interfaces for the workstation, including IP address and default gateway.

  -  `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\WindowsNT\CurrentVersion\Winlogin`

>Winlogon is a common target for several threats that could modify its function and memory usage. Signs of increased memory usage for this process might indicate that it has been compromised.

  (PICTURE) (PICTURE)
  
---
#### Step 2.4: Collect the required registry keys related to the current user account: Shell Bags and RecentDocs

In the Registry Editor, naviaget to the following keys:

  -  `HKEY_CURRENT_USER\Software\Microsoft\Windows\Shell\Bags`
    
>The information in Shell Bags can be vital to providing non-repudiation of claims that a defendant was not aware of a particular folder on their computer.
    
  -  `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs`
    
>Within this key you can see hexadecimal records for the last 10 files that the current user accessed through the File Explorer.

    (PICTURE)(PICTURE)

---

## Skills Learned

- **Live system tools reveal real-time evidence**: Windows utilities provide immediate insight into what a system is doing at any given moment. By examining active processes, network connections, and listening ports, you can quickly identify unusual or suspicious behavior. This real-time data is essential for detecting potential threats while the system is still running.
- **NTFS and usn journal provide historical file activity**: This information allowed me to reconstruct evidence when files were created, modified, deleted or completely gone. This historical view is critical for building timelines and understanding how an incident unfolded.
- **Registry keys store critical system and user artifacts**: The registry acts as a central repository for system config, user activity, and application behavior. Keys like Winlogon, ShellBags, and RecentDocs reveal how the system is set up, how users interact with it, and how attackers may establish persistence. 
  

---
## Troubleshooting Logic

| Symptoms | Causes | Resolution |
|---|---|---|
| applications fail to start, login issues, broken user files | malware tampering, improper software removal, registry corruption | compare keys against known good baselines, restore from registry backup, reinstall affected apps |
| files appearing/disappearing without user action, suspicious extensions | malware execution, automated scripts, unathorized activity| review usn journal entries for timestamps and file IDs, use fsutil to locate file paths |
| system lag, high CPU or memory usage in Task Manager | background services stuck, malware processing, runaway processes| check command line for suspicious execution, disable services |


