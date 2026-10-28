FROM mcr.microsoft.com/dotnet/aspnet:11.0@sha256:bae0269049d5061ab7f49995cc1acebf54b75471c096c4a1f874e27661d44045 AS base
USER $APP_UID
WORKDIR /app
EXPOSE 8080
EXPOSE 8081

FROM mcr.microsoft.com/dotnet/sdk:11.0@sha256:3a9175b6c54b2f076817af1360f691c9994447b192faa48a1a1f112a516844c7 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["UmbracoProject/UmbracoProject.csproj", "UmbracoProject/"]
RUN dotnet restore "UmbracoProject/UmbracoProject.csproj"
COPY . .
WORKDIR "/src/UmbracoProject"
RUN dotnet build "UmbracoProject.csproj" -c $BUILD_CONFIGURATION -o /app/build

FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "UmbracoProject.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
# We need to make sure that the user running the app has write access to the umbraco folder, in order to write logs and other files.
# Since these are volumes they are created as root by the docker daemon.
USER root
RUN mkdir umbraco
RUN mkdir umbraco/Logs
RUN chown $APP_UID umbraco --recursive
#if (UmbracoRelease = 'LTS')
RUN chown $APP_UID wwwroot/umbraco --recursive
#endif
USER $APP_UID
ENTRYPOINT ["dotnet", "UmbracoProject.dll"]
