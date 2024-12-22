# Base runtime image
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 8080
EXPOSE 8443

# Build image with SDK
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["DotaInfo.Api/DotaInfo.Api.csproj", "DotaInfo.Api/"]
RUN dotnet restore "DotaInfo.Api/DotaInfo.Api.csproj"
COPY . .
WORKDIR "/src/DotaInfo.Api"
RUN dotnet build "DotaInfo.Api.csproj" -c Release -o /app/build

# Generate SSL certificate during build
RUN apt-get update && apt-get install -y openssl && \
    mkdir -p /app/certs && \
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /app/certs/server.key -out /app/certs/server.crt \
    -subj "/CN=localhost" && \
    openssl pkcs12 -export -out /app/certs/certificate.pfx \
    -inkey /app/certs/server.key -in /app/certs/server.crt \
    -password pass:YourSecurePassword

# Publish the application
FROM build AS publish
RUN dotnet publish "DotaInfo.Api.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Final runtime image
FROM base AS final
WORKDIR /app

# Create certs directory
RUN mkdir -p /app/certs

# Copy the certificate from build stage
COPY --from=build /app/certs/certificate.pfx /app/certs/

# Copy published app
COPY --from=publish /app/publish .

# Configure HTTPS environment variables
ENV ASPNETCORE_URLS="http://+:8080;https://+:8443" \
    ASPNETCORE_Kestrel__Certificates__Default__Path=/app/certs/certificate.pfx \
    ASPNETCORE_Kestrel__Certificates__Default__Password=YourSecurePassword

ENTRYPOINT ["dotnet", "DotaInfo.Api.dll"]