FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /source

# Copy csproj and restore dependencies (removed the broken -a flag)
COPY *.csproj .
RUN dotnet restore

# Copy source code and build (removed the broken -a flag)
COPY . .
RUN dotnet publish -c release -o /app --no-restore

# Create the final runtime image
FROM mcr.microsoft.com/dotnet/runtime:7.0
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "Worker.dll"]
