FROM mcr.microsoft.com/dotnet/runtime:6.0.4-alpine3.15-amd64
LABEL maintainer="yohann.nicolas@modulusgroup.eu"
LABEL vendor="Modulus Group"

RUN apk update
RUN apk upgrade
RUN apk add bash iputils nano icu-libs krb5-libs libgcc libintl libssl1.1 libstdc++ zlib
RUN apk add libgdiplus --repository https://dl-3.alpinelinux.org/alpine/edge/testing/
RUN env
ENV \
    # ASP.NET Core version
    ASPNET_VERSION=6.0.4 \
    # Set the default console formatter to JSON
    Logging__Console__FormatterName=Json \
    PATH="$PATH:$HOME/.dotnet/tools" \
    DOTNET_ROOT="$HOME/.dotnet" \
    PATH="$PATH:$DOTNET_ROOT"

RUN env
# Install ASP.NET Core
RUN wget -O aspnetcore.tar.gz https://dotnetcli.azureedge.net/dotnet/aspnetcore/Runtime/$ASPNET_VERSION/aspnetcore-runtime-$ASPNET_VERSION-linux-musl-x64.tar.gz \
    && aspnetcore_sha512='f54624306b74d9be0a670c2b1d465991b2c1ef67a4c216532fba9dc85f525a68d9ba6e1405945905dc834e073e676f0234d18edc5c9507d5b6c420bb2d073a40' \
    && echo "$aspnetcore_sha512  aspnetcore.tar.gz" | sha512sum -c - \
    && tar -oxzf aspnetcore.tar.gz -C /usr/share/dotnet ./shared/Microsoft.AspNetCore.App \
    && rm aspnetcore.tar.gz

RUN mkdir /mnt/share