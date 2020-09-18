FROM mcr.microsoft.com/dotnet/core/aspnet:3.1-buster-slim AS base
WORKDIR /app


FROM mcr.microsoft.com/dotnet/core/sdk:3.1-buster AS build
WORKDIR /src
COPY kanikodemo/kanikodemo.csproj kanikodemo/

RUN dotnet restore "kanikodemo/kanikodemo.csproj"
COPY . .
WORKDIR "/src/kanikodemo"

RUN dotnet build "kanikodemo.csproj" -c Release -o /app/build

FROM build AS publish

RUN dotnet publish "kanikodemo.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "kanikodemo.dll"]