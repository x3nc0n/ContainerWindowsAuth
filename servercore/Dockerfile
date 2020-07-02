# escape=`
FROM mcr.microsoft.com/windows/servercore/iis
SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'Continue'; $verbosePreference='Continue';"]
WORKDIR app
#Adding Windows Authentication Feature since not installed in default image
RUN Install-WindowsFeature "Web-Windows-Auth", "Web-Asp-Net45"
#Adding local user for testing local Windows Authentication
RUN  new-LocalUser -Name "ContainerAdmin" -Password  (ConvertTo-SecureString  "A123456!" -AsPlainText -Force); `
     Add-LocalGroupMember -Group Administrators -Member "ContainerAdmin" ; 
# Add failure audit logging to debug Authentication Issues
RUN Auditpol /set /category:"Logon/Logoff" /Success:enable /failure:enable
COPY ["default.aspx", "c:\\inetpub\\wwwroot\\app\\"]
RUN Import-Module IISAdministration; `
    Start-IISCommitDelay; `
    (Get-IISConfigSection -SectionPath 'system.webServer/security/authentication/windowsAuthentication').Attributes['enabled'].value = $true; `
    (Get-IISConfigSection -SectionPath 'system.webServer/security/authentication/anonymousAuthentication').Attributes['enabled'].value = $false; `
    (Get-IISServerManager).Sites[0].Applications[0].VirtualDirectories[0].PhysicalPath = 'c:\inetpub\wwwroot\app' ; `
    Stop-IISCommitDelay

# Make logs readable by docker/k8s
RUN Set-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' -filter 'system.applicationHost/log' -name 'centralLogFileMode' -value 'CentralW3C'; `  
    Set-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' -filter 'system.applicationHost/log/centralW3CLogFile' -name 'truncateSize' -value 4294967295; `
    Set-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' -filter 'system.applicationHost/log/centralW3CLogFile' -name 'period' -value 'MaxSize'; `
    Set-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' -filter 'system.applicationHost/log/centralW3CLogFile' -name 'directory' -value 'c:\iislog'

# Make sure the file exists to avoid errors.
CMD Start-Service W3SVC; `  
    Invoke-WebRequest http://localhost -UseBasicParsing | Out-Null; `
    netsh http flush logbuffer | Out-Null; `
    Get-Content -path 'c:\iislog\W3SVC\u_extend1.log' -Tail 1 -Wait 

EXPOSE 80