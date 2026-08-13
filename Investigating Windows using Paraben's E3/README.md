# Investigating Windows using Paraben's E3

## Objective

Using several utilities to perform live analysis on a Windows Server 2019 system, by first exploring evidence-rich areas of the Windows operating system - the Windows Registry. Then investigating the Windows drive image and conducting forensic analysis by using Paraben's E3 to explore: NTFS file system, Windows Registry on Windows 10 drive image, and Windows artifacts (link files and browser history.

## Business Scenario

A company's Information Technology department received an alert indicating that an employee's workstation has accessed several restricted company folders outside of the employee's normal working hours. The folders contain proprietary financial reports, internal business documents, and customer related information. Security logs showed several unusual file-access events occurring late at night shortly before a large amount of data was copied from an employee's workstation. Management was concerned that the employee's computer may have been used to access or copy proprietary information without authorization. The company also discovered a USB storage device had recently been connected to the workstation.

Since the activity could represent a violation of the company's acceptable-use and information security policies, the company's IT security manager authorized a digitial forensic investigation.



## Environment and Tools Used

| Component | Detail |
|---|---|
| Server | Windows 2019 (VM workstation)|
| Test Image | Windows 10 drive image |
| Test Scope | Test vm only (No production machine or real user data) |
| Live System Analysis | Task Manager, Resource Monitor, Fsutil |
| Windows Registry Analysis | Registry Editor (Regedit) |
| Forensic Image Analysis | Paraben's E3 |


## ***Steps Performed***

## Phase A: Gather Basic System Information

### Part 1: Using the Task Manager

#### Step 1.1: Open Task Manager

From the workstation desktop, **right click** anywhere on the taskbar, then select **Task Manager** from the context menu to open the Task Manager.

  (PICTURE)

  Note: From the Task Manager, you can review all the application, processes, and services that are active at the moment.     You can even attempt to shut down specific processes, if necessary.
  
#### Step 1.2: Select command line column

In the Task Manager, **right click** the **Name Column header**, then **select Command line** from the context menu to add the Command line column.

  (PICUTRE)

   Note: By adding the Command line column, you can observe the file path for each process's executable file, as well as       any command line options that were used to launch the program. For forensic investigators it can be a valuable source of    detailed information when attempting to analyze and isolate malware that has been running since the start of the system.

#### Step 1.3: Select properties Window

In the Task Manager, **identify** one of the core Windows processes listed above, then **right click** the process and **select properties** from the context menu to display additional information about the process in the Properties window. Then close the properties window.

  (PICTURE)

#### Step 1.4: Performance tab

In the Task Manager, **click** the **Performance tab** to display information about resource usage.

  (PICTURE)


#### Step 1.5: Select Resource Monitor

From the bottom of the Task Manager, **click** the **Open Resource Monitor link** to launch the Resource Monitor.

  (PICTURE)

  Note: The Resource Monitor is a dedicated utility for monitoring system resource usage in real time, including CPU,         memory usage, disk utilization, and network use. When the Resource Monitor contains many of the same metrics as the         Performance Tab, it allows knowledgeable users to drill deeper into the underlying data.

#### Step 1.6: Select Network tab

In the Resource Monitor, **click** the **Network tab** to display a list of processes that are currently generating network activity.

  (PICUTRE)

 Note: The Network tab displays network activity, TCP connections, and listening ports for any running process. As a         forensic investigator, this information can be essential to identifying spyware, botnets, and other forms of malware that   require network connectivity.

#### Step 1.7: Select Listening Ports header

On the Network tab, **expand** the **Listening Ports header** to display a list of ports that are actively listening and the processes that are using them. Close the **Resource Monitor** and **Task Manager windows**.

    (PIC)

### Part 2: Use the command prompt to locate information about the c: drive and the workstation's usn journal

#### Step 2.1: Command Prompt - fsinfo

Open the **Command Prompt** and type `fsutil fsinfo ntfsinfo c:` and **press enter** to display information about the c: drive.

    (PIC)

   Note: The output of this command provides information about the c: drive, inlcuding the NTFS volume serial number,       version, number of total clusters, number of free clusters, total number of sectors, and more. In forensic investigations, this information can be used to determine if there is hidden data in a boot file or boot record of the drive. Also it can be used to determine if there are fake bad clusters, which are areas of the drive that the file system ignores due to damage, but are in fact viable and used to hide data.

#### Step 2.2: Command Prompt - usn

At the command prompt, type `fsutil usn queryjournal c:` and **press enter** to display information related to the update sequence number changes journal.

    (PIC)

  Note: The update sequence number (usn) change journal provides visibility into filesystem activity going back several days or even weeks. As files, directories, and other NTFS objects are added, deleted, and modified, NTFS enters records into the usn change journal, one for each volume on the computer. Each record indicates the type of change that occurred, such as file creation or deletion, and the filename that was changed.
  
#### Step 2.3: Inspect the contents

At the command prompt, type `fsutil usn readjournal c:` and **press enter** to display all records in the usn journal.
>After allowing the command to run for about 10 seconds, **press ctrl+c** to terminate it.

Note: This information can be useful for several reasons. First, when a program is run, typically you will be able to       see the modified prefetch files, which will give you a timestamp for execution. Second, the usn journal can provide         evidence of deleted files. Third, you can see file modifications and creation of specific particular file extensions,       such as executbale, which in some contexts can be indicative of a system compromise. Many compromises occur after a         piece of file-based malware is initially run, like from a macro or PDF. The usn journal can provide a timeline of           initial infection.
   
### Part 3: Identify the file path for yourname.txt file

#### Step 3.1: Generate a new record for the usn journal

On the desktop, **right click** on empty space and **select New → Text Document** to create an empty text file on the desktop. In the filename field, type `yourname` where your name is your own name, and **press enter** to name the new file. 

#### Step 3.2: Export the usn journal

At the command prompt, type `cd desktop` and **press enter** to change the current directory to the desktop. Then type `fsutil usn readjournal c: csv > usn.log` and **press enter** to save the contents of the usn journal to the workstation desktop as a .csv file.

#### Step 3.3: Use notepad to open file and identify the file path  

On the desktop, **select** the **usn.log file** to open it in Notepad. In the notepad window, **scroll down** to the bottom of the file and **locate** the record referencing the **yourname.txt file**. **Highlight** the **File ID value** for the yourname.txt file, then **press ctrl+c** to copy it to the clipboard.
>The file ID should be the long hexadecimal number directly to the right of the word "Archive".

#### Step 3.4: Retrieve the file path

At the command prompt, type `fsutil file queryFileNameById c:\ 0xFile ID`, where File ID is the value you copied from the previous command output (paste), then **press enter** to retrieve the file path associated with the file ID.

  (PICTURE)

## Phase 2: Explore the Registry

### Part 1: Getting Acquainted with Registry Editor

  Note: The Windows Registry is one of the key components of the Windows OS. This database contains Windows settings, application settings, device driver info, and user passwords. When an application is installed in a Windows environment, some part of the software is stored in the Registry. Registry analysis is often a critical component of conducting forensics investigations on Windows systems.

#### Step 1.1: Open the registry editor to locate the current version key

From the taskbar, **click** the **start icon** and type `regedit` and **press enter** to search for and launch the Registry Editor. Navigate to the following key: **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\WindowsNT\CurrentVersion**

  Note: The HKEY_LOCAL_MACHINE registry hive contains the majority of the configuration info for the software installed on the system, as well as for the Windows OS itself. It also contains valuable information about currently detected hardware and device drivers.
>This key contains multiple data points that could be very useful to an investigator who has obtained a drive image  as evidence, but doesn't know the exact version of Windows that was installed on it. If the drive image was taken from a recent compromised system, this information could help corroborate evidence of an exploit.

#### Step 1.2: Locate the InstallDate value 

In the right pane, **select** the **InstallDate value** to open the Edit DWORD Value Box (32 bit). In the dialog box

#### Step 1.3: Copy the value data and paste into epochconverter website field

In the dialog box, **right click** the **Value data field**, then **select copy** from the context menu to copy the hex value to the clipboard. Then close the box. **Select Chrome icon**, to open a new Chrome browser window. In the navigation bar, type `epochconverter.com/hex` and **press enter** to navigate to the hexadecimal converter. In the hexadecimal converter, **delete** the existing timestamp value, then **right click** the empty hex field and **select paste** from the context menu to paste the hex value into the converter. **Click** to perform the conversion. Close the browser.

  Note: Value data in the Windows Registry is commonly stored in the hexadecimal notation.

#### Step 1.4: [Step Name]

Describe the task performed.

#### Step 1.2: [Step Name]

Describe the task performed.

#### Step 1.3: [Step Name]

Describe the task performed.



## Skills Learned

- Skill or competency developed
- Skill or competency developed
- Skill or competency developed
- Skill or competency developed
- Skill or competency developed


## Troubleshooting Logic/Reference

| Symptoms | Causes | Resolution |
|---|---|---|
| Operating System | |
| Hardware | |
| Software | |
| Networking | |
| Security Tools | |
| Other | |
