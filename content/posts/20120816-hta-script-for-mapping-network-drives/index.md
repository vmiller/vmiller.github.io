---
title: "HTA Script for Mapping Network Drives"
date: 2012-08-16T07:11:44-05:00
draft: false
tags: [Windows]
---

I had a need for a user friendly Windows script to map network drives using credentials supplied by the user.  The script I endend up with is an HTA script that allows the user to enter their credentials and map a predefined set of network drives.  There is also a button to disconnect the mapped drives.

The script is here : https://github.com/vmiller/ConnectDrives

```
<!-- HTA script to allow machines that are not joined to a domain to access
     Windows file shares with domain credentials. It will atomatically prepend the
     domain to the username and then map several drives. If a drive is already
     mapped, it is disconnected and then mapped for the current user.
      
     Version 1.0.2
     Written by Vaughn Miller 7/20/2012
      
     Currently setup to map the following drives :
     M: = \\gonzo.ad.messiah.edu\dept
     O: = \\gonzo.ad.messiah.edu\users
     W: = \\mcweb\messiahweb
     ---------------------------------------------------------------------------------->
 
 
<HTML>
<HEAD>
<TITLE>Connect Network Drives</title>
<HTA:APPLICATION
ICON="EIPos.ico"
     ApplicationName="MapDrives.HTA"
     SingleInstance="Yes"
     WindowsState="Normal"
     Scroll="No"
     Navigable="Yes"
     MaximizeButton="No"
     SysMenu="Yes"
     Caption="Yes"
></HEAD>
 
<SCRIPT LANGUAGE="VBScript">
 
' *** Define Drive Mappings ***
dim arrDrives(2,2)
intMaxdrives = 2
 
arrDrives(0,0) = "M:"
arrDrives(0,1) = "\\gonzo.ad.messiah.edu\dept"
arrDrives(0,2) = "Dept"
 
arrDrives(1,0) = "O:"
arrDrives(1,1) = "\\gonzo.ad.messiah.edu\users"
arrDrives(1,2) = "Users"
 
arrDrives(2,0) = "W:"
arrDrives(2,1) = "\\mcweb\messiahweb"
arrDrives(2,2) = "messiahweb"
' *** End Drive Map Definitions ***
 
strDOMAIN = "messiah\" 'Domain to prepend to the username
 
 
Sub Window_Onload
  '# Size Window
  sHorizontal = 440
  sVertical = 175
  Window.resizeTo sHorizontal, sVertical
  '# Get Monitor Details
  Set objWMIService = GetObject _
    ("winmgmts:root\cimv2")
  intHorizontal = sHorizontal *2
  intVertical = sVertical *2
  Set colItems = objWMIService.ExecQuery( _
    "Select ScreenWidth, ScreenHeight from" _
    & " Win32_DesktopMonitor", , 48)
  For Each objItem In colItems
    sWidth= objItem.ScreenWidth
    sHeight = objItem.ScreenHeight
    If sWidth > sHorizontal _
      then intHorizontal = sWidth
    If sHeight > sVertical _
      then intVertical = sHeight
  Next
  Set objWMIService = Nothing
  '# Center window on the screen
  intLeft = (intHorizontal - sHorizontal) /2
  intTop = (intVertical - sVertical) /2
  Window.moveTo intLeft, intTop
  '# default window content
  window.location.href="#Top"
End Sub
 
 
Sub RunScript
   on Error Resume Next
 
   minUSRnamelength = 2
   minPASSwrdlength = 3
 
   strUsr = UsrnameArea.Value
   strPas = PasswordArea.Value
 
   Set objNetwork = CreateObject("WScript.Network")
   Set oShell = CreateObject("Shell.Application")
 
   If Len(strUsr) >= minUSRnamelength then
      strUsr = strDOMAIN & UCase(strUsr) '<--- adds the domain before the username
 
      if Len(strPas) >= minPASSwrdlength Then
         Call ClearDrives ' Delete existing mappings if they exist
          
         '***** Begin Drive mapping *****
         For n = 0 To intMaxDrives 'Loop through our array of drives
            Err.Clear
            objNetwork.MapNetworkDrive arrDrives(n,0), arrDrives(n,1), False, strUsr, strPas
            If Err.Number = 0 Then
               oShell.NameSpace(arrDrives(n,0)).Self.Name = arrDrives(n,2)
            End If
         Next
         '***** End Drive Mapping *****
           
         ELSE
            Msgbox chr(34) & strPas & """ is an incorrect password !"
            Exit Sub
         End If
   ELSE
      Msgbox chr(34) & strUsr & """ is an incorrect Username !"
      Exit Sub
   End If
    ' Clean up the objects before exiting
   Set oShell = Nothing
   Set objNetwork = Nothing
   Self.Close()
End Sub
 
 
Sub ClearDrives ' Sub Routine to remove the drives if they are already mapped
  On Error Resume Next
  Set objNetwork = CreateObject("WScript.Network")
 
  '***** Begin section to delete drive mappings ***
  Set AllDrives = objNetwork.EnumNetworkDrives
  For n = 0 To intMaxDrives 'Loop through our array of drives
     For i = 0 To AllDrives.Count - 1 Step 2
        If AllDrives.Item(i) = arrDrives(n,0) Then AlreadyConnected = True
     Next
     If AlreadyConnected = True then
        objNetwork.RemoveNetworkDrive arrDrives(n,0), True, True
     End If
  Next
  '***** End section to delete drive mappings
End Sub
 
 
Sub DisconnectDrives ' Calls ClearDrives subroutine and then closes the window
Call ClearDrives
    Set oShell = Nothing
    Set objNetwork = Nothing
Self.close()
End Sub
 
 
Sub CancelScript
   Set oShell = Nothing
   Set objNetwork = Nothing
   Self.Close()
End Sub
 
</SCRIPT>
 
 
<BODY STYLE="font:14 pt arial; color:white; filter:progid:DXImageTransform.Microsoft.Gradient(GradientType=1, StartColorStr='#000000', EndColorStr='#0000FF')">
<a name="Top"></a><CENTER>
  <table border="0" cellpadding="0" cellspacing="0"><font size="2" color="black" face="Arial">
    <tr>
      <td height="30">
        <p align="right">Your Username</p>
      </td>
      <td height="30">&nbsp;&nbsp; <input type="text" name="UsrnameArea" size="30"></td></tr>
    <tr>
      <td height="30">
        <p align="right">Password</p>
      </td>
      <td height="30">&nbsp;&nbsp; <input type="password" name="PasswordArea" size="30"></td></tr>
  </table><BR>
<HR color="#0000FF">
 <Input id=runbutton class="button" type="button" value=" Map Drives " name="run_button" onClick="RunScript">
    &nbsp;
 <Input id=runbutton class="button" type="button" value=" Disconnect Drives " name="dis_button" onClick="DisconnectDrives">
    &nbsp;
 <Input id=runbutton class="button" type="button" value="Cancel" name="cancel_button" onClick="CancelScript">
</CENTER>
</BODY>
 
</HTML>
```

The drive definitions are coded in an array so that the mapping and disconnecting subroutines can use a loop. To modify this for your own use, you need to modify strDomain (line 50) and the drive definitions (lines 33-48).

I think a nice revision of this script would be to design it to read a configuration file for the domain info and drive definitions. This was one motivation for implementing the array/loop structure.