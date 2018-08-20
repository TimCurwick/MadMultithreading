# MadMultithreading

Multithreading functions

Download and install the MadMultithreading module from the PowerShell Gallery using PowerShell:

```powershell
Install-Module -Name MadMultithreading
```

## New-MadThread

Start a given PowerShell script in a new thread.

Takes a Scriptblock, a RunspacePool and a hash table of Parameters. Returns a custom object with pointers to the associated PowerShell, Handler, and RunspacePool objects.

If variables with names matching the thread parameters exist in the calling script, switch parameter `-UseEmbeddedParameters` can be used instead of explicitly creating and using a hash table of parameters. When this switch is used, the function parses the parameter block of the provided scriptblock and dynamically creates a parameter hash table, using the parameter names as keys and any defined variables with names matching the parameters as values.

```powershell
$Threads = 6
$RunspacePool = [runspacefactory]::CreateRunspacePool( 1, $Threads )
$RunspacePool.Open()

$Threads = 1..$Threads | ForEach-Object {
    New-MadThread `
        -Scriptblock  $ThreadScript `
        -RunspacePool $RunspacePool `
        -Parameters @{
            InputQueue  = $InputQueue
            OutputQueue = $OutputQueue } }

#  or

$Threads = 1..$Threads | ForEach-Object {
    New-MadThread `
        -Scriptblock  $ThreadScript `
        -RunspacePool $RunspacePool `
        -UseEmbeddedParameters }
```

## Wait-MadThread

Wait for given threads to complete, optionally disposing of them.

```powershell
$Threads | Wait-MadThread
```

## Start-MadOutputThread

Start a thread to export items from given queues to given CSV files.

Takes a `-Queue` and a `-Path` or an array of queues and paths as `-Reports`.

```powershell
Start-MadOutputThread -Queue $OutputQueue -Path $OutputCSV
```

```powershell
$Reports = @(
    @{ Queue = $OutputQueue; Path = $OutputCSV    }
    @{ Queue = $LotQueue   ; Path = $LogPath      }
    @{ Queue = $ErrorQueue ; Path = $ErrorLogPath } )

Start-MadOutputThread -Reports $Reports
```

## Invoke-MadMultithread

Run input against a function in multiple threads.

Can be used to multithread almost any pipeline function.

The results aren't as efficient as writing custom multithreading code specific to your purpose, but writing and maintaining such custom code is time consuming and challenging. This is designed to be fast and easy.

This won't work for all pipeline functions. For example, it won't work for any function which acts on the entire collection or any function which does work in the End block. Measure-Object is an example of a function that cannot be multithreaded. But it works for most pipeline functions.

```powershell
#  Single thread
$Groups | Get-ADGroupMember

#  Default 2 threads
$Groups | Invoke-MadMultithread Get-ADGroupMember
```

```powershell
#  Single thread
$Groups | Get-ADGroupMember -Server DC01

#  4 threads
$Groups | Invoke-MadMultithread Get-ADGroupMember -Parameters @{ Server = 'DC01' } -Threads 4
```

```powershell
#  Single thread
$Groups | Get-ADGroupMember -Server DC01 -Recursive

#  8 threads
$Groups | Invoke-MadMultithread Get-ADGroupMember -Parameters @{ Server = 'DC01'; Recursive = $True } -Threads 8
```

```powershell
function YourFunction {
    Param ( [parameter( ValueFromPipeline = $True )]$InputParam, $Thing1, $Thing2 )
    Process
        {
        # Your pipeline code
        }
    }

#  Single thread
$InputObjects | YourFunction -Thing1 'Alpha' -Thing2 27

#  Default 2 threads
$InputObjects | Invoke-MadMultithread YourFunction -Parameters @{ Thing1 = 'Alpha'; Thing2 = 27 }
```

```powershell
function Start-RandomSleep {
    Param ( [parameter( ValueFromPipeline = $True )]$Thing1 )
    Process
        {
        Start-Sleep -Seconds ( Get-Random 5 )
        $Thing1
        }
    }

#  Default sorting - Output is in same order as input
1..10 | Invoke-MadMultithread Start-RandomSleep -Threads 10

#  returns 1, 2, 3, 4, 5, 6, 7, 8, 9, 10


#  NoSort option - Results returned when ready
1..10 | Invoke-MadMultithread Start-RandomSleep -Threads 10 -NoSort

#  returns 7, 4, 5, 1, 6, 8, 10, 2, 3, 9
```

Additional tips and techniques at [MadWithPowerShell.com](https://MadWithPowerShell.com).
Issues, comments, suggestions, and other feedback welcome at [https://github.com/TimCurwick/MadMultithreading](https://github.com/TimCurwick/MadMultithreading).
