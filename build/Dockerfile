FROM mcr.microsoft.com/dotnet/sdk:6.0-alpine
WORKDIR /app

ARG NUGET=http://172.17.145.17/nuget
RUN dotnet nuget add source ${NUGET} -n "Modulus package source" 