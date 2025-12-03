FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS base
WORKDIR /app
EXPOSE 8080
EXPOSE 8081

# Используем SDK образ для сборки
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

# Копируем .csproj файл и восстанавливаем зависимости
# Контекст сборки: ./backend/LLMService/, поэтому LLMService.csproj лежит в корне контекста
COPY ["LLMService.csproj", "."]
RUN dotnet restore "LLMService.csproj" # Восстанавливаем зависимости

# Копируем весь исходный код
# Контекст сборки: ./backend/LLMService/, поэтому копируем всё содержимое этой папки в /src
COPY . .
# Результат: /src содержит LLMService.csproj, Program.cs, Controllers/, Models/ и т.д.

# Собираем проект
# Файл LLMService.csproj теперь лежит прямо в /src (не в подпапке)
RUN dotnet build "LLMService.csproj" -c $BUILD_CONFIGURATION -o /app/build

# Стадия публикации
FROM build AS publish
ARG BUILD_CONFIGURATION=Release
# Публикуем проект, файл LLMService.csproj лежит в /src (не в подпапке)
RUN dotnet publish "LLMService.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

# Финальный образ
FROM base AS final
WORKDIR /app
# Копируем опубликованные файлы из стадии publish
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "LLMService.dll"]