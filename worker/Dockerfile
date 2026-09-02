FROM mcr.microsoft.com/dotnet/sdk:10.0

WORKDIR /app

RUN chown ubuntu:ubuntu /app

USER ubuntu

COPY --chown=ubuntu:ubuntu worker.csproj ./

RUN dotnet restore

COPY --chown=ubuntu:ubuntu . .

RUN dotnet build

CMD ["dotnet", "run", "--no-launch-profile"]