# Local DPI patches

Экспериментальный отдельный пакет патчей для Morphe. Встраивает локальный движок
[zapret/tpws](https://github.com/bol-van/zapret) в YouTube с патчами Morphe.
Это независимый проект, не официальный продукт Morphe или zapret.

**Статус: экспериментальный выпуск.** Данный профиль tpws воспроизводил видео
в нашей модификации SmartTube на одном телефоне и одной сети. Интеграция именно
с Morphe пока не прошла проверку воспроизведения на Android. Успешная сборка
не означает, что обход работает у каждого провайдера.

## Добавление в Morphe

Источник GitHub: `D3v3l0p3r0/local-dpi-patches`.

Скачайте `.mpp` из [Releases](https://github.com/D3v3l0p3r0/local-dpi-patches/releases).
Добавьте его как локальный пакет патчей, либо добавьте этот репозиторий как
сторонний источник, если ваша версия Morphe поддерживает источники GitHub.
Для сторонних пакетов может потребоваться экспертный режим.

## Требования и применение

1. Сначала подготовьте APK YouTube официальным пакетом Morphe с выбранным
   необязательным патчем **Network proxy**.
2. К полученному APK примените **Local DPI circumvention (tpws)** из этого пакета.
   Это дополнение к уже пропатченному APK: исходный APK YouTube не содержит
   необходимых классов. При их отсутствии патч должен завершиться ошибкой.
3. Установите итоговый APK. В настройках Morphe включите **Network proxy**.
   Адрес и порт локального прокси патч задаёт автоматически.
4. Проверяйте на телефоне без VPN: главную страницу, изображения, несколько
   роликов и перемотку. Выключение Network proxy отключает маршрутизацию через
   встроенный движок.

Android 6+ на уровне встроенного движка; актуальные YouTube/Morphe могут
требовать более новый Android. Архитектуры: arm64-v8a, armeabi-v7a, x86.
Root и внешний прокси-сервер не требуются. Настройки Morphe Network proxy,
совместимость Cronet и повторное патчирование APK ещё нуждаются в проверке.

## Как устроено

Cronet → локальный HTTP CONNECT на `127.0.0.1` → локальный SOCKS5 tpws → сайт.
HTTPS проходит через CONNECT без расшифровки и подмены сертификатов.
Прокси слушают только loopback. Если движок не запускается, соединение выдаёт
ошибку; автоматический выход в сеть напрямую отключён.

Профиль:

```text
--hostlist-domains=youtubei.googleapis.com
--split-pos=1 --disorder --oob --tlsrec=host+1
--new --split-pos=1,midsld --disorder --tlsrec=midsld
```

Это фиксированная начальная стратегия; автоматического подбора конфигурации
в этом выпуске нет. Движок не меняет системный DNS, не создаёт VPN и не
обрабатывает трафик других приложений.

## Сборка

Основа: официальный [morphe-patches-template](https://github.com/MorpheApp/morphe-patches-template).
Нужны JDK, Android SDK/NDK/CMake и доступ `read:packages` к GitHub Packages,
как описано в шаблоне. Перед Gradle соберите движок:

```text
python tools/prepare-native.py
gradlew.bat buildAndroid --no-daemon --max-workers=1
```

`tools/prepare-native.py` использует NDK 25.2.9519653 и CMake 3.22.1 из
`ANDROID_HOME`. Сначала добавьте их через SDK Manager. Результат `.mpp`:
`patches/build/libs/`. Исходники движка включены в `native/tpws`.

Локальный `build-local.cmd` содержит пути среды разработки и использует
соседний SDK. Их можно заменить своими. Не добавляйте токены в репозиторий.
Вспомогательная Windows-форма `tools/MorpheCredentials.cs` хранит учётные
данные только в Windows Credential Manager, не в исходниках.

Проверка HTTP/SOCKS-моста: `tools/test-bridge.cmd`.

## Лицензии

Код пакета распространяется по GPL-3.0, см. LICENSE.
zapret/tpws — MIT, см. `native/tpws/LICENSE.txt`; дополнительные уведомления
находятся в исходниках tpws (в том числе `andr/_musl_license.txt`).
