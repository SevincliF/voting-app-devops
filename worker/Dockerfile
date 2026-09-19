FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build

WORKDIR /src

COPY Worker.csproj .

RUN dotnet restore

COPY . .

RUN dotnet publish -c Release -o /app/publish



FROM mcr.microsoft.com/dotnet/runtime:7.0 AS final

WORKDIR /app

COPY --from=build /app/publish /app/

RUN useradd --create-home workeruser
USER workeruser

ENTRYPOINT [ "dotnet", "Worker.dll" ]