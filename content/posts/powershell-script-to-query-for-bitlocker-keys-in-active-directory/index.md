---
title: "Powershell Script to Query for Bitlocker Keys in Active Directory"
date: 2013-07-03T07:13:47-05:00
draft: false
tags: [Windows, Powershell, Bitlocker]
---

In my organization, we are using Bitlocker to encrypt Windows 7 computers. We are storing the recovery keys in Active Directory, this stores the key as an attribute of the computer object. I recently wanted to generate a report of the bitlocker status of the computer objects in AD. I found out I could do this pretty easily in Powershell, and thought I would document that here.  My inspiration for this script came from this [Technet Gallery script](http://gallery.technet.microsoft.com/4231a8a1-cc60-4e07-a098-2844353186ad)

To start, we need the Quest ActiveRoles Management Shell for for Active Directory.   This is available for free from Quest and can be downloaded from [here](http://www.quest.com/powershell/activeroles-server.aspx).  This should be downloaded and installed on the workstation that is going to be used to run the script.  With this installed, we are ready to take a look at the script.

```
# Check to make sure the path has been specified otherwise display a message and exit the script
param([string]$CsvFilePath)
if (!$CsvFilePath) {
Write-Host ""
Write-Host "Path not not specified!"
Write-Host "Please specify the path for the output as a parameter e.g. : "
Write-Host ".\Get-BitlockerComputerReport.ps1 """c:\reports\BitlockerReport.csv""""
Exit
}
```

The script will ultimately output the report to a CSV file.  I want the path and filename to the report to be specified as a command line parameter to keep the script flexible.  As a result, we start the script with checking for the existence of the parameter.  If none was provided, we will give user a helpful message and exit the script.

```
# Check to make sure the correct snapins are installed and loaded otherwise exit the script
$snaps1 = Get-PSSnapin -Registered
$snaps2 = Get-PSSnapin Quest* -ErrorAction SilentlyContinue
$questsnap = 0
foreach ($snap1 in $snaps1) {
    if ($snap1.name -eq "Quest.ActiveRoles.ADManagement") {
        write-host "Quest Snapin Registered"
        $questsnap = 1
        }
    }
if ($questsnap -eq 0) {
    Write-Host "Quest Snapin not Registered."
    Write-Host "Please install the Quest ActiveRoles Management Shell for Active Directory"
    Write-Host "Download from : http://www.quest.com/powershell/activeroles-server.aspx"
    Exit
}
 
if ($questsnap -eq 1) {
    foreach ($snap2 in $snaps2) {
        if($snap2.name -eq "Quest.ActiveRoles.ADManagement") {
            Write-Host "Quest Snapin already loaded"
            $questsnap = 2
            }
        }
    if ($questsnap -ne 2) {
        Write-Host "Loading Quest Snapin..."
        Add-PSSnapin Quest.ActiveRoles.ADManagement
        }
    }
```

This next section of code is checking for the required snapin. Our script cannot work without the Quest snapin so we should check for the snapin and load it. Lines 2-10 in the code above gets a list of registered snapins and checks to see if the one we need is among them. If it is, we display a message and set $questsnap to 1. In Lines 11-16 we deal with case where the snapin was not registered. We display a helpful message and exit the script. In lines 18-24 we know the snapin is registered but it may or may not be loaded. We check to see if it is loaded and display a message to the console if it is. Lines 25-59 take care of loading the snapin if we have determined that the snapin is registered but not loaded.

With the preliminary error checking out of the way, let’s get started with the main part of the script :

```
$SearchContainer = "DN=Computers,DC=example,DC=com"
 
write-host "Starting query..."
 
#Query Active Directory
$BitLockerEnabled = Get-QADObject -SizeLimit 0 -IncludedProperties Name,ParentContainer | Where-Object {$_.type -eq "msFVE-RecoveryInformation"} | Foreach-Object {Split-Path -Path $_.ParentContainer -Leaf} | Select-Object -Unique
$computers = Get-QADComputer -SearchRoot $SearchContainer -SizeLimit 0 -IncludedProperties Name,OperatingSystem,ModificationDate,ParentContainer,msTPM-OwnerInformation | Where-Object {$_.operatingsystem -like "Windows 7*" -or $_.operatingsystem -like "Windows Vista*"} | Sort-Object Name
 
#Create array to hold computer information
$export = @()
```

Line 1 here is setting our base search container. The search will include all sub OU’s as well. You may want to change this to suite your environment. Line 7 and 8 are where we use the cmdlets provided by the Quest snap-ins. The query in line 7 will get a collection of objects that have Bitlocker recovery information. The query in line 8 will build a collection will all Windows 7 and Vista computer objects. Make special note of the “-IncludedProperties” part of the queries. You may wish to use a different set of properties to suite your own needs.  In line 10 we are initializing an array to be used in the next section of the script :

```
foreach ($computer in $computers)
  {
    #Create custom object for each computer
    $computerobj = New-Object -TypeName psobject
 
    #Add desired properties to custom object
    $computerobj | Add-Member -MemberType NoteProperty -Name Name -Value $computer.Name
    $computerobj | Add-Member -MemberType NoteProperty -Name OperatingSystem -Value $computer.operatingsystem
    $computerobj | Add-Member -MemberType NoteProperty -Name ModificationDate -Value $computer.ModificationDate
    $computerobj | Add-Member -MemberType NoteProperty -Name ParentContainer -Value $computer.ParentContainer
 
    #Set HasBitlockerRecoveryKey to true or false, based on matching against the computer-collection with BitLocker recovery information
    if ($computer.name -match ('(' + [string]::Join(')|(', $bitlockerenabled) + ')')) {
    $computerobj | Add-Member -MemberType NoteProperty -Name HasBitlockerRecoveryKey -Value $true
    }
    else
    {
    $computerobj | Add-Member -MemberType NoteProperty -Name HasBitlockerRecoveryKey -Value $false
    }
 
    #Set HasTPM-OwnerInformation to true or false, based on the msTPM-OwnerInformation on the computer object
     if ($computer."msTPM-OwnerInformation") {
    $computerobj | Add-Member -MemberType NoteProperty -Name HasTPM-OwnerInformation -Value $true
    }
    else
    {
    $computerobj | Add-Member -MemberType NoteProperty -Name HasTPM-OwnerInformation -Value $false
    }
 
#Add the computer object to the array with computer information
$export += $computerobj
  }
```

This section of code is a For loop to loop through the entire collection of computer objects. In line 4 we create an object for the current computer and then in lines 7-10 we add the desired properties. You will note that these correspond to the properties that were listed in the “-IncludedProperties” part of the query from the previous section od code.

In lines 12-19 we determine if the record had a Bitlocker key present and then add a property to our object with a value of true or false. In lines 22-28 we do a similar operation, but with TPM Owner information.

Finally in line 31 we add the object we just build to our array. Now to finish up the script is just one more line :

```
$export | Export-Csv -Path $CsvFilePath -NoTypeInformation
```

This final line simply outputs our array to a CSV file to the path we specified as a parameter when we ran the script.  There is a bit of room for improvement here, as the folder path specified must pre-exist or the script will fail.  It would be better to check for the path and create it if it does not exist.

The full script is available on [GitHub](https://github.com/vmiller/BitlockerComputerReport)