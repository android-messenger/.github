# Wolp Messenger для Android

Android-версия мессенджера Hormiga (`org.wolpmessenger`) с доверием к устройству на аппаратном уровне.
В приложение пускают не только по логину и паролю: нужно ещё зарегистрированное устройство, которое подтвердило свою подлинность через Android Key Attestation.

## Как это устроено

- **Приложение** — оболочка Capacitor над существующим веб-клиентом. Интерфейс не переписывается на Kotlin: веб и Android используют один и тот же UI.
- **Доверие к устройству.** Ключи создаются в TEE или StrongBox и не извлекаются. Шлюз проверяет аттестацию; программные ключи, root и разблокированный загрузчик отклоняются.
- **Взаимный TLS.** Клиентский сертификат живёт 72 часа и продлевается с новым ключом и новой аттестацией. mTLS завершается на шлюзе, бэкенд мессенджера о нём не знает.
- **Одно устройство на пользователя.** Перерегистрация идёт только через администратора, по одноразовому паролю онбординга. Прежнее устройство при этом отзывается автоматически.
- **RASP на устройстве.** При признаках компрометации приложение сообщает шлюзу, стирает ключи и блокируется.
- **Push без метаданных.** FCM передаёт только сигнал «разбудись», без текста сообщений и имён.
- **Без Google Play.** APK распространяется файлом, minSdk 33 (Android 13).

## Репозитории

| Репозиторий | Что это | Стек |
|---|---|---|
| [`wolp-android`](https://github.com/android-messenger/wolp-android) | Android-приложение: WebView-оболочка, FCM, звонки, обновления | Kotlin, Capacitor |
| [`wolp-device-trust`](https://github.com/android-messenger/wolp-device-trust) | Библиотека доверия к устройству: Keystore, аттестация, mTLS-сертификаты, RASP, Vault | Kotlin |
| [`gate`](https://github.com/android-messenger/gate) | Шлюз идентификации на базе Ory Kratos, модуль `devices` (регистрация, аттестация, реестр mTLS) и `device-signer` | Go |
| [`firebase-push`](https://github.com/android-messenger/firebase-push) | Сервис push-уведомлений FCM для приложения | Go |
| [`messenger`](https://github.com/android-messenger/messenger) | Веб-клиент (React PWA): работает в браузере и внутри WebView приложения | TypeScript |
| [`HormigaMessanger`](https://github.com/android-messenger/HormigaMessanger) | Бэкенд мессенджера | Java, Quarkus |
| [`deploy`](https://github.com/android-messenger/deploy) | Послойный one-shot деплой всего стека | Shell, Docker Compose |
| [`docs`](https://github.com/android-messenger/docs) | Архитектурные решения и спецификации S1–S3 | — |

## Схема

```
 Android-устройство                    Сервер
┌──────────────────────┐   mTLS   ┌──────────────┐     ┌──────────────────────┐
│ wolp-android         │─────────▶│ nginx        │────▶│ HormigaMessanger     │
│  └ messenger (WebView)│          │ + gate       │     │ (бэкенд, Kafka)      │
│  └ wolp-device-trust │  enroll  │  (Kratos,    │     └──────────┬───────────┘
│     (TEE/StrongBox)  │─────────▶│   devices,   │                │
└──────────▲───────────┘          │   signer)    │       chat.offline.notify
           │  FCM «разбудись»      └──────────────┘                │
           └────────────────────── firebase-push ◀────────────────┘
```

## Документация

Все решения (D1–D19), модель угроз и правила совместимости описаны в [`docs/ARCHITECTURE.md`](https://github.com/android-messenger/docs/blob/main/ARCHITECTURE.md). Спецификации компонентов лежат в [`docs/specs`](https://github.com/android-messenger/docs/tree/main/specs).

## Безопасность

Уязвимости, пожалуйста, не публикуйте в issues. Сообщайте о них владельцу организации напрямую.
