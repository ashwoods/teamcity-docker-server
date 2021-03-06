ARG TAG=1803

FROM microsoft/windowsservercore:$TAG AS base

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

ARG TAG=$TAG

# Install Git
RUN [Net.ServicePointManager]::SecurityProtocol = 'tls12, tls11, tls' ; \
    Invoke-WebRequest https://github.com/git-for-windows/git/releases/download/v2.18.0.windows.1/MinGit-2.18.0-64-bit.zip -OutFile git.zip; \
    Expand-Archive git.zip -DestinationPath $Env:ProgramFiles\Git ; \
    Remove-Item -Force git.zip

# Install Mercurial
RUN Invoke-WebRequest https://bitbucket.org/tortoisehg/files/downloads/mercurial-4.6.1-x64.msi -OutFile hg.msi; \
    Start-Process msiexec -Wait -ArgumentList /q, /i, hg.msi ; \
    Remove-Item -Force hg.msi

# Prepare TeamCity server distribution
ADD TeamCity-*.tar.gz /
RUN New-Item C:/TeamCity/webapps/ROOT/WEB-INF/DistributionType.txt -type file -force -value "docker-windows-${env:TAG}" | Out-Null
COPY run-server.ps1 /TeamCity/run-server.ps1

FROM teamcity-base:latest-nanoserver-$TAG

COPY --from=base ["C:/Program Files/Git", "C:/Program Files/Git"]
COPY --from=base ["C:/Program Files/Mercurial", "C:/Program Files/Mercurial"]

ENV TEAMCITY_DIST="C:\TeamCity" \
    TEAMCITY_LOGS="C:\TeamCity\logs" \
    TEAMCITY_DATA_PATH="C:\ProgramData\JetBrains\TeamCity"

EXPOSE 8111

VOLUME $TEAMCITY_DATA_PATH \
       $TEAMCITY_LOGS

COPY --from=base $TEAMCITY_DIST $TEAMCITY_DIST

CMD pwsh C:/TeamCity/run-server.ps1

# In order to set system PATH, ContainerAdministrator must be used
USER ContainerAdministrator
RUN setx /M PATH "%PATH%;C:\Program Files\Git\cmd;C:\Program Files\Mercurial"
USER ContainerUser