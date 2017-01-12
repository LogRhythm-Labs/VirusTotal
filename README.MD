
    #====================================================#
    # LogRhythm Labs                                     #
    # Virus Total Process Analysis Automation            #
    # Greg Foss | @heinzarelli | greg.foss@logrhythm.com #
    # v0.1 -- November, 2015                             #
    #====================================================#

## [About]

Requires PowerShell version 3.0 or higher on both the source and target Windows hosts.

This script interrogates a local/remote systems process and takes a hash of the application that the process was launched from and submits it to Virus Total. If there is a record of that hash, the script will display the results of the previous scan, and if the sample is new, it will submit the sample to be scanned.

Ideally, this script should be integrated with the organization's Active Defense frameworks to automate rapid analysis of suspect processes and/or files.

## [How To]

#####Run vt-check on local host:

    Check a file against Virus Total using their API
        PS C:\> .\vt-check.ps1 -file "C:\Users\taco\Desktop\eicar.txt" -VTApiKey [key]

    Check a running process against Virus Total using their API
        Process ID
            PS C:\> .\vt-check.ps1 -processID 1234 -VTApiKey [key]

        Process Name (less accurate than process ID if there are multiple processes with the same name)
            PS C:\> .\vt-check.ps1 -processName chrome -VTApiKey [key]

    Send results via email
        PS C:\> .\vt-check.ps1 -processID 1234 -smtpServer [127.0.0.1] -emailTo [greg.foss[at]logrhythm.com] -emailFrom [virustotal[at]logrhythm.com] -VTApiKey [key]

#####Run vt-check on remote host:

    PS C:\> .\vt-check.ps1 -remote -target [computer] [arguments - EX: -processID -file -username -password -VTApiKey]
        Caveats:
            You will need to ensure that psremoting and unsigned execution is enabled on the remote host.  // dangerous to leave enabled!
            Be careful, this may inadvertently expose administrative credentials when authenticating to a remote compromised host.
    
    Caveats:
        You will need to ensure that psremoting and unsigned execution is enabled on the remote host.  <== dangerous to leave enabled!!
        Be careful, this may inadvertently expose administrative credentials when authenticating to a compromised host.

#####What if PSRemoting and Unrestricted Execution are disabled?

    Remotely enable PSRemoting and Unrestricted PowerShell Execution using PsExec and PSSession, then run vt-check

        Option 1 -- WMI:
            PS C:\> wmic /node:"10.10.10.10" process call create "powershell -noprofile -command Enable-PsRemoting -Force" -Credential Get-Credential

        Option 2 - PsExec:
            PS C:\> PsExec.exe \\10.10.10.10 -u [admin account name] -p [admin account password] -h -d powershell.exe "Enable-PSRemoting -Force"
        
        Next...

            PS C:\> Test-WSMan 10.10.10.10
            PS C:\> Enter-PSSession 10.10.10.10
            [10.10.10.10]: PS C:\> Set-ExecutionPolicy Unrestricted -Force
    
    Be careful! This will open the system up to unnecessary risk!!
    You could also inadvertently expose administrative credentials when authenticating to a compromised host.

## [Parameter Breakdown]

	Virus Total API

         -VTApiKey  :   Virus Total API Key

         The free version will work but understand you are limited in the number of queries allowed per day
         It is recommended to hard-code this into the script, so you don't have to enter it every time

    Processes and Files:

        -file       :   Designates a file to be scanned
        -processName:   Designates a process name to be scanned
        -processID  :   Designates a process ID to be scanned

    Remote Execution:

		-remote 	:	Switch to run vt-check against a remote host
		-target 	:	Define the remote host to extract data from

	Send Virus Total Scan Results via Email:

		-sendEmail 	: 	Allows the script to send the HTML report over SMTP
        -smtpServer : 	Sets the remote SMTP Server that will be used to forward reports
        -emailTo 	: 	Defines the email recipient. Multiple recipients can be separated by commas
        -emailFrom 	: 	Defines the email sender

        It is recommended to hard-code these values into the script, so you don't have to enter them every time

    Credentials - Required for remote execution and interaction with Active Directory.

    	-username 	:	Administrative Username - can be supplied on the command-line or hard-coded into the script
        -password 	: 	Administrative Password - can be supplied on the command-line or hard-coded into the script <== Bad idea!!
        
        If neither parameter is supplied, you will be prompted for credentials -- the safest option aside from local execution

## [Use Cases]

#####1) Basic Process Capture and Analysis

Quickly analyze processes and/or files to check for signs of malware. This can be run locally or remotely as necessary.

#####2) SIEM Integration for Incident Response Automation

Integrate the script with your SIEM to automatically scan suspicious processes and/or files with VirusTotal. This script is best-utilized in high-security environments where process monitoring is in place and application whitelisting is in effect.

## [Notes]

Keep in mind that while using a free VirusTotal API key is a great way to gather intel and potentially halt an infection, it will expose these samples to the internet, and thus give away information to a potential attacker. This is bad OpSec and is reason enough to purchase a commercial license to VirusTotal if you are considering running this rule in a production setting. Not only that but be aware that Anti-Virus software is inherently flawed and is unable to detect more sophisticated attacks due to the fact that it primarily relies on signatures. However, by automating VirusTotal scans you gain the power of over 55 different AV vendors covering all new processes/files as opposed to just one AV solution. Chances are, at least one vendor will catch a majority of malware submitted.

## [License]

Copyright (c) 2016 LogRhythm

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
