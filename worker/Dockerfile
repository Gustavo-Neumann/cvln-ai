FROM mcr.microsoft.com/dotnet/sdk:10.0

WORKDIR /app

COPY worker.csproj ./
RUN dotnet restore

COPY . .
RUN dotnet build --no-restore

CMD ["dotnet", "run", "--no-restore", "--no-launch-profile"]
