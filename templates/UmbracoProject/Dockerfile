FROM mcr.microsoft.com/dotnet/aspnet:11.0@sha256:623c52ed3fad0fa423b8cc860910492364b0d6245870a415e997017d4a4f4513 AS base
USER $APP_UID
WORKDIR /app
EXPOSE 8080
EXPOSE 8081

FROM mcr.microsoft.com/dotnet/sdk:11.0@sha256:bfabf51e74c5a7cf2fc4eac852145db86f4867d475257d809518d6a2b0fe71e1 AS build
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
