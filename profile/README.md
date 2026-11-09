# Wolp Messenger для Android

Android-версия мессенджера Hormiga (`org.wolpmessenger`) с доверием к устройству на аппаратном уровне.
В приложение пускают не только по логину и паролю: нужно ещё зарегистрированное устройство, которое подтвердило свою подлинность через Android Key Attestation, и оригинальное приложение, подписанное нашим ключом.

## Как это устроено

- **Приложение** — оболочка Capacitor над веб-клиентом мессенджера; веб и Android используют один UI. Целевая схема — интерфейс упакован в APK и подписан вместе с приложением.
- **Доверие к устройству.** Ключи создаются в TEE (StrongBox — опционально) и не извлекаются. Шлюз проверяет аттестацию; программные ключи, root, разблокированный загрузчик и чужая подпись APK отклоняются.
- **Взаимный TLS.** Клиентский сертификат живёт 72 часа и продлевается с новым ключом и новой аттестацией. mTLS завершается на шлюзе, бэкенд мессенджера о нём не знает.
- **Одно устройство на пользователя.** Регистрация — только по одноразовому паролю от администратора; новая регистрация отзывает прежнее устройство.
- **RASP.** При признаках компрометации приложение сообщает шлюзу, стирает ключи и блокируется.
- **E2EE.** Ключи Signal обёрнуты неизвлекаемым ключом в TEE.
- **Без Google Play.** APK распространяется файлом, Android 13+.

## Репозитории

| Репозиторий | Назначение |
|---|---|
| [`messenger`](https://github.com/android-messenger/messenger) | веб-клиент (React PWA): одна кодовая база для браузера и для приложения |
| [`gate`](https://github.com/android-messenger/gate) | шлюз: KratosGate (Ory Kratos) + модуль `devices` (регистрация, аттестация, реестр mTLS) + `device-signer` |
| [`firebase-push`](https://github.com/android-messenger/firebase-push) | сервис push-уведомлений FCM для приложения |
| [`HormigaMessanger`](https://github.com/android-messenger/HormigaMessanger) | бэкенд мессенджера (Quarkus) |
| [`wolp-android`](https://github.com/android-messenger/wolp-android) | Android-приложение: оболочка Capacitor, WebView с mTLS, плагины, звонки, обновления |
| [`wolp-device-trust`](https://github.com/android-messenger/wolp-device-trust) | библиотека доверия к устройству: Keystore, аттестация, сертификаты, RASP, Vault |
| [`deploy`](https://github.com/android-messenger/deploy) | послойный деплой всего стека (Docker Compose) |
| [`docs`](https://github.com/android-messenger/docs) | архитектура, спецификации, концепция безопасности, инструкции по установке |

## Документация

- [Архитектура и решения D1–D19](https://github.com/android-messenger/docs/blob/main/ARCHITECTURE.md)
- [Концепция безопасности](https://github.com/android-messenger/docs/tree/main/security) — модель угроз, аттестация, PKI, mTLS, RASP, E2EE
- [Установка: сервер](https://github.com/android-messenger/docs/blob/main/install/server.md) · [телефон](https://github.com/android-messenger/docs/blob/main/install/phone.md)
- [Спецификации S1–S3](https://github.com/android-messenger/docs/tree/main/specs)

Репозитории закрытые: ссылки открываются участникам организации.

## Безопасность

Уязвимости, пожалуйста, не публикуйте в issues. Сообщайте о них владельцу организации напрямую.
