ARG DOTNET_SDK_VERSION8_TAG=8.0-jammy
ARG DOTNET_SDK_VERSION9_TAG=9.0-preview-noble
ARG MINVER_IMAGE_TAG=latest
ARG MINVER_IMAGE=teslaconsulting/minver-cli:$MINVER_IMAGE_TAG
ARG DOCKER_CLI_IMAGE=docker:cli
ARG TEAMCITYAGENT_IMAGE_REPOS=jetbrains/teamcity-agent
ARG TEAMCITYAGENT_IMAGE_TAG=2023.05.6-linux-sudo

FROM mcr.microsoft.com/dotnet/sdk:${DOTNET_SDK_VERSION9_TAG} AS dotnetsdk9
FROM mcr.microsoft.com/dotnet/sdk:${DOTNET_SDK_VERSION8_TAG} AS dotnetsdk8
FROM $MINVER_IMAGE AS minver
FROM $DOCKER_CLI_IMAGE AS dockercli

FROM scratch AS build
# make sure folder exist
WORKDIR /usr/local/bin
WORKDIR /usr/share
COPY --from=dotnetsdk8 /usr/share/dotnet /usr/share/dotnet
COPY --from=dotnetsdk9 /usr/share/dotnet /usr/share/dotnet
COPY --from=dotnetsdk9 /usr/share/powershell /usr/share/powershell
COPY --from=minver /usr/share/minver-cli /usr/share/minver-cli
COPY --from=dockercli /usr/local/libexec/docker/cli-plugins /usr/local/libexec/docker/cli-plugins

FROM ${TEAMCITYAGENT_IMAGE_REPOS}:${TEAMCITYAGENT_IMAGE_TAG} AS final

USER root


ARG DOTNET_SDK_VERSION8=8.0.0
ARG ASPNET_VERSION8=8.0.0
ARG DOTNET_VERSION8=8.0.0

ARG DOTNET_SDK_VERSION9=9.0.0
ARG ASPNET_VERSION9=9.0.0
ARG DOTNET_VERSION9=9.0.0

ARG POWERSHELL_VERSION=7.4.1
ARG MINVER_VERSION=4.3.0
ARG DOCKER_COMPOSE_VERSION=2.24.6

ENV DOTNET_SDK_VERSION8=${DOTNET_SDK_VERSION8} \
  ASPNET_VERSION8=${ASPNET_VERSION8} \
  DOTNET_VERSION8=${DOTNET_VERSION8} \
  \
  DOTNET_SDK_VERSION9=${DOTNET_SDK_VERSION9} \
  ASPNET_VERSION9=${ASPNET_VERSION9} \
  DOTNET_VERSION9=${DOTNET_VERSION9} \
  \
  DOTNET_SDK_VERSION=${DOTNET_SDK_VERSION8} \
  ASPNET_VERSION=${ASPNET_VERSION8} \
  DOTNET_VERSION=${DOTNET_VERSION8} \
  \
  POWERSHELL_VERSION=${POWERSHELL_VERSION} \
  POWERSHELL_DISTRIBUTION_CHANNEL=PSDocker-DotnetSDK-Ubuntu-20.04 \
  DOTNET_MINVER_VERSION=${MINVER_VERSION} \
  DOCKER_COMPOSE_VERSION=${DOCKER_COMPOSE_VERSION}

COPY --from=build /usr /usr

RUN  ln -s /usr/share/powershell/pwsh /usr/bin/pwsh \
  && chmod 755 /usr/share/powershell/pwsh \
  && ln -s /usr/share/minver-cli/minver /usr/bin/minver \
  && chmod +x /usr/share/minver-cli/minver \
  && rm /usr/local/bin/docker-compose \
  && ln -s /usr/local/libexec/docker/cli-plugins/docker-compose /usr/local/bin/docker-compose \
  && chmod +x /usr/libexec/docker/cli-plugins/docker-* \
  # Install module Pester - for root user
  && pwsh -Command "Install-Module -Name Pester -Scope AllUsers -Force" \
  && dotnet --help \
  && dotnet --info \
  && dotnet nuget locals all --clear

USER buildagent
