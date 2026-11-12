# syntax=docker/dockerfile:1

ARG DOTNET_VERSION=10.0

# ---------- Build ----------
# Portable (framework-dependent) publish, so the SDK stage runs natively on the build host.
FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:${DOTNET_VERSION} AS build
WORKDIR /src
COPY ManagerServer.csproj ./
RUN dotnet restore ManagerServer.csproj
COPY . .
RUN dotnet publish ManagerServer.csproj -c Release -o /app --no-restore /p:UseAppHost=false

# ---------- ASP.NET runtime ----------
FROM mcr.microsoft.com/dotnet/aspnet:${DOTNET_VERSION} AS aspnet

# ---------- Runtime ----------
# Debian instead of the default Ubuntu base: Ubuntu's chromium package is a snap stub,
# while PDF rendering (PuppeteerSharp) needs a real headless browser.
FROM debian:trixie-slim AS runtime
ARG APP_RELEASE=dev
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        ca-certificates tzdata libicu76 libssl3t64 zlib1g \
        chromium fonts-liberation fonts-noto-core fonts-noto-cjk \
    && rm -rf /var/lib/apt/lists/* \
    && groupadd --gid 1654 app \
    && useradd --uid 1654 --gid app --create-home --shell /usr/sbin/nologin app \
    && install -d -o app -g app /data
COPY --from=aspnet /usr/share/dotnet /usr/share/dotnet
RUN ln -s /usr/share/dotnet/dotnet /usr/bin/dotnet
ENV DOTNET_RUNNING_IN_CONTAINER=true \
    APP_RELEASE=$APP_RELEASE
WORKDIR /app
COPY --from=build /app ./
USER app
VOLUME ["/data"]
EXPOSE 8080
ENTRYPOINT ["dotnet", "ManagerServer.dll"]
CMD ["--urls", "http://*:8080", "--path", "/data"]
