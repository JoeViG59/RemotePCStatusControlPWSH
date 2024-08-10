################################
###
### Author : Joe Vidal
### Date : 8/10/2024
###
### Show live status of computers in an active directory environment.
### This was used to get information from a specific remote site that would require rebooting and checking for network connectivity.
### Site would consistently ask for remote shutdowns as said computers were unreachable.
### Only able to remote reboot if they had network connectivity hence the live status funcion.
### Launch VNC option for specific computer for further troubleshooting.
###
###
###
### Note: Variables changed from my original script to maintain privacy
###
##############################


# Get list of computers with a specific naming pattern
$computers = Get-ADComputer -Filter 'Name -like "ComputerNamePattern*"' -Properties IPv4Address | select Name, IPv4Address 

# Function to display live status
function ShowOnlineStatus {
    Write-Host " " -NoNewline
    $greenbar= Write-Host "[|||||]" -BackgroundColor Green -ForegroundColor Black
}

# Function to display offline status in a form of a redbar
function ShowOfflineStatus {
    Write-Host " " -NoNewline
    $redbar = Write-Host "[|||||]" -BackgroundColor Red -ForegroundColor Black
}

# Function to continuously update the status of computers. Alias for start-sleep is sleep.
function LiveStatusUpdate {
    while ($true) {
        Clear-Host
        Write-Host "Current Computer Status" -ForegroundColor Magenta `n`n
        DisplayStatus
        Start-Sleep -Seconds 2
    }
}

# Function to display the status of computers
function DisplayStatus {
    foreach ($computer in $computers) {
        $pingResult = Test-Connection -ComputerName $computer.IPv4Address -Count 1 -Quiet
        if ($pingResult) {
            Write-Host "$($computer.Name) is online" -NoNewline
            ShowOnlineStatus
            Write-Host "$($computer.IPv4Address)`n"
        } else {
            Write-Host "$($computer.Name) is offline" -NoNewline
            ShowOfflineStatus
            Write-Host "$($computer.IPv4Address)`n"
        }
    }
}

# Function to reboot a selected computer
function RebootComputer {
    DisplayStatus
    Write-Host "Which computer?" -BackgroundColor Black -ForegroundColor Yellow -NoNewline
    $computerNumber = Read-Host " "
    $selectedComputer = $computers | Where-Object { $_.Name -eq "ComputerNamePattern$computerNumber" }

    if ($selectedComputer) {
        Write-Host "Rebooting Computer $computerNumber..." -BackgroundColor Black -ForegroundColor Red
        Restart-Computer -ComputerName $selectedComputer.Name -Force 
        Start-Sleep -Seconds 6
        Write-Host "Rebooted Computer $computerNumber..." -BackgroundColor Black -ForegroundColor Red
        Start-Sleep -Seconds 2
        Clear-Host
        MainMenu
    } else {
        Write-Host "Invalid Computer number."
        Clear-Host
        MainMenu
    }
}

# Function to launch a remote session for a selected computer. Assuming the executable names
# shared the same name as the actual computer to match VNC's file path.
function LaunchRemoteSession {
    DisplayStatus
    Write-Host "Which computer?" -BackgroundColor Black -ForegroundColor Yellow -NoNewline
    $computerNumber = Read-Host " "
    $selectedComputer = $computers | Where-Object { $_.Name -eq "ComputerNamePattern$computerNumber" }

    if ($selectedComputer) {
        Write-Host "Launching remote session for Computer $computerNumber..." -BackgroundColor Black -ForegroundColor Red
       
        ##VNC.exe path specific to computers name.

        Start-Process "Path\To\Remote\ComputerNamePattern$computerNumber.exe"
        Write-Host "Launched." -BackgroundColor Black -ForegroundColor Red
        Start-Sleep -Seconds 2
        Clear-Host
        MainMenu
    } else {
        Write-Host "Invalid Computer number."
        Clear-Host
        MainMenu
    }
}

# Main menu function
function MainMenu {
    DisplayStatus
    Write-Host "[1] Live Status" -ForegroundColor Yellow
    Write-Host "[2] Reboot Computers" -ForegroundColor Yellow
    Write-Host "[3] Launch Remote Session" -ForegroundColor Yellow
    Write-Host "[4] Refresh Menu" -ForegroundColor Yellow

    $selectOption = Read-Host "What do you want?"

    if ($selectOption -eq 1) {
        Clear-Host
        LiveStatusUpdate
    } elseif ($selectOption -eq 2) {
        Clear-Host
        RebootComputer
    } elseif ($selectOption -eq 3) {
        Clear-Host
        LaunchRemoteSession
    } elseif ($selectOption -eq 4) {
        Clear-Host
        MainMenu
    } else {
        Write-Host "Invalid option."
        Start-Sleep -Seconds 2
        Clear-Host
        MainMenu
    }
}

Write-Host `n"Current Computer Status"`n`n
MainMenu
