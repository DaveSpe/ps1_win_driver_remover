# ps1_win_driver_remover
Base code for windows 3rd party driver removal, by ITBros.com

## Source 
[TheITBros.com](https://theitbros.com/remove-old-unused-drivers-using-powershell/)

## Usage

Removing Old and Unused Drivers from Driver Store using Powershell
FEBRUARY 9, 2017 CYRIL KARDASHEVSKYWINDOWS, POWERSHELL
Each time you install or update your device driver, Windows OS (since Vista) continues to store the old version of the driver in the system Driver Store. Thereby, if the system doesn’t work correctly with the new driver, user can roll back to an older version of the driver at any moment.

Windows Driver Store is located in the folder C:\windows\system32\DriverStore\FileRepository. However, some Windows users began to notice that over time, the folder %windir%\system32\DriverStore\FileRepository begins to occupy more and more disk space. In our example, the size of this folder is more than 11 GB. Quite a lot, especially for the SSD drive!

powershell uninstall driver

How to Remove Old and Unused Drivers from Driver Store?
To clear the contents of folder FileRepository from the outdated drivers, we prepared a small PowerShell script, that removes all duplicates drivers except the drivers with the latest date.

ADVERTISEMENT
Warning! In any case don’t delete any files or folders manually from the directory FileRepository!

Important Tips:

To run this script you need to update your PowerShell version to Windows Management Framework 1 (or at least WMF4).
Before running the script, make sure you have the current drivers for the video card, printer and other devices, that are constantly in use. In case of problems, you can always re-install properly driver.
Also it is recommended to create a restore point. To do this, run the command:
Checkpoint-Computer -Description “PointBeforeDeleteUnusedDrivers”
Verify that the restore point was created successfully using the command:

Get-ComputerRestorePoint
We will get the list of third-party drivers installed in the system by using the DISM command:

dism /online /get-drivers
Then parse the output using PowerShell, select the driver duplicates and sort them by date. From the resulting list we will exclude the most recent version for each driver.

After that, remove remaining drivers using pnputil utility. In some cases, if some driver is not removed, add the -f switch.

The script is as follows:

```powershell

$dismOut = dism /online /get-drivers

$Lines = $dismOut | select -Skip 10




$Operation = "theName"

$Drivers = @()




foreach ( $Line in $Lines ) {




    $tmp = $Line

    $txt = $($tmp.Split( ':' ))[1]




    switch ($Operation) {




        'theName' { $Name = $txt

                     $Operation = 'theFileName'

                     break

                   }




        'theFileName' { $FileName = $txt.Trim()

                         $Operation = 'theEntr'

                         break

                       }




        'theEntr' { $Entr = $txt.Trim()

                     $Operation = 'theClassName'

                     break

                   }




        'theClassName' { $ClassName = $txt.Trim()

                          $Operation = 'theVendor'

                          break

                        }




        'theVendor' { $Vendor = $txt.Trim()

                       $Operation = 'theDate'

                       break

                     }




        'theDate' { # change the date format for easy sorting

                     $tmp = $txt.split( '.' )

                     $txt = "$($tmp[2]).$($tmp[1]).$($tmp[0].Trim())"

                     $Date = $txt

                     $Operation = 'theVersion'

                     break

                   }




        'theVersion' { $Version = $txt.Trim()

                        $Operation = 'theNull'




                        $params = [ordered]@{ 'FileName' = $FileName

                                              'Vendor' = $Vendor

                                              'Date' = $Date

                                              'Name' = $Name

                                              'ClassName' = $ClassName

                                              'Version' = $Version

                                              'Entr' = $Entr

                                            }

   

                        $obj = New-Object -TypeName PSObject -Property $params

                        $Drivers += $obj




                        break

                      }




         'theNull' { $Operation = 'theName'

                      break

                     }




    }

}




Write-Host "All installed third-party  drivers"

$Drivers | sort Filename | ft




Write-Host "Different versions"




$last = ''

$NotUnique = @()




foreach ( $Dr in $($Drivers | sort Filename) ) {

   

    if ($Dr.FileName -eq $last  ) {  $NotUnique += $Dr  }

    $last = $Dr.FileName

}




$NotUnique | sort FileName | ft




Write-Host "Outdated drivers"

$list = $NotUnique | select -ExpandProperty FileName -Unique




$ToDel = @()

foreach ( $Dr in $list ) {

    Write-Host "duplicate found" -ForegroundColor Yellow

    $sel = $Drivers | where { $_.FileName -eq $Dr } | sort date -Descending | select -Skip 1

    $sel | ft




    $ToDel += $sel

}




Write-Host "Drivers to remove" -ForegroundColor Red




$ToDel | ft




# removing old drivers

foreach ( $item in $ToDel ) {

    $Name = $($item.Name).Trim()

    Write-Host "deleting $Name" -ForegroundColor Yellow

    Write-Host "pnputil.exe -d $Name" -ForegroundColor Yellow

    Invoke-Expression -Command "pnputil.exe -d $Name"

}
```

Tip. In case you need to show the list of unused drivers only, comment Invoke-Expression (add a # character before the command Invoke-Expression -Command “pnputil.exe -d $Name)

Copy the code of the script and save it to file drv_cleanup.ps1 (to the folder c:\ps). Open the elevated PowerShell console and allow to execute the unsigned scripts:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

Next run the script: `C:\PS\drv_cleanup.ps1`

powershell remove driver

The script will remove unused drivers. After its completion, restart your computer and check if everything works properly and, if necessary, reinstall the appropriate driver.
