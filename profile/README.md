# Wolp Messenger для Android

Android-версия мессенджера Hormiga (`org.wolpmessenger`) с доверием к устройству на аппаратном уровне.
В приложение пускают не только по логину и паролю: нужно ещё зарегистрированное устройство, которое подтвердило свою подлинность через Android Key Attestation, и оригинальное приложение, подписанное нашим ключом.

## Как это устроено

- **Приложение** — оболочка Capacitor над веб-клиентом мессенджера; веб и Android используют одну кодовую базу UI. Интерфейс собирается внутрь APK и подписывается вместе с приложением.
- **Доверие к устройству.** Ключи создаются в TEE (StrongBox — опционально) и не извлекаются. Шлюз проверяет аттестацию; программные ключи, root, разблокированный загрузчик и чужая подпись APK отклоняются.
- **Взаимный TLS.** Клиентский сертификат живёт 72 часа и продлевается с новым ключом и новой аттестацией. mTLS завершается на шлюзе, бэкенд мессенджера о нём не знает.
- **Одно устройство на пользователя.** Регистрация — только по одноразовому паролю от администратора; новая регистрация отзывает прежнее устройство.
- **RASP.** При признаках компрометации приложение сообщает шлюзу, стирает ключи и блокируется.
- **E2EE.** Ключи Signal обёрнуты неизвлекаемым ключом в TEE.
- **Без Google Play.** APK распространяется файлом, Android 13+.

## Репозитории

| Репозиторий | Назначение | Состояние |
|---|---|---|
| [`messenger-ui`](https://github.com/android-messenger/messenger-ui) | веб-клиент мессенджера (React PWA): копия фронта Hormigas с правками для Android; одна кодовая база для браузера и для приложения | работает: браузер и UI внутри APK |
| [`gate`](https://github.com/android-messenger/gate) | шлюз: KratosGate (Ory Kratos) + модуль `devices` (регистрация, аттестация, реестр mTLS) + `device-signer` + офлайн-CA `device-ca`; один образ, три точки входа | работает |
| [`firebase-push`](https://github.com/android-messenger/firebase-push) | сервис push-уведомлений FCM для приложения (на основе сервиса Web Push Hormigas) | работает |
| [`messanger`](https://github.com/android-messenger/messanger) | копия бэкенда мессенджера Hormigas (Quarkus); нужна только для одной добавочной правки (отмена звонка в push) | правка сделана |
| [`wolp-android`](https://github.com/android-messenger/wolp-android) | Android-приложение: оболочка Capacitor, WebView с mTLS, мост, хранилище ключей E2EE; тестовый стенд шлюза | пилот: сообщения, звонки, push; автообновление — позже |
| [`wolp-device-trust`](https://github.com/android-messenger/wolp-device-trust) | библиотека доверия к устройству: Keystore, аттестация, сертификаты и mTLS, RASP, Vault | готова |
| [`deploy`](https://github.com/android-messenger/deploy) | композиция стека `wolpertinger` (Docker Compose, nginx, мониторинг, скрипты администратора) | работает |
| [`docs`](https://github.com/android-messenger/docs) | архитектура, спецификации, концепция безопасности, инструкции администратора | — |
| [`.github`](https://github.com/android-messenger/.github) | этот публичный профиль организации | — |

## Система

Один стек `wolpertinger` (Docker Compose за nginx): Ory Kratos и Oathkeeper, шлюз `gate` с модулем `devices` и `device-signer`, бэкенд мессенджера, каталог ключей E2EE, `firebase-push`, веб-клиент, свой coturn, Postgres/Kafka/Redis/MinIO, Prometheus + Alertmanager (тревоги в Telegram). Браузер ходит на `:443`, приложение — на `:8444` (вход, регистрация устройства) и `:8443` (mTLS).

| Задача | Где |
|---|---|
| что в системе, какой репозиторий за что | [ops/README](https://github.com/android-messenger/docs/blob/main/ops/README.md) |
| развернуть с нуля | [ops/01](https://github.com/android-messenger/docs/blob/main/ops/01-deploy.md) |
| пользователи, пароль онбординга, телефон, отзыв устройства | [ops/02](https://github.com/android-messenger/docs/blob/main/ops/02-users.md) |
| выпустить версию APK | [ops/03](https://github.com/android-messenger/docs/blob/main/ops/03-release.md) |
| обновления, оповещения, логи, откат, частые проблемы | [ops/04](https://github.com/android-messenger/docs/blob/main/ops/04-operations.md) |

## Документация

- [Архитектура и решения D1–D19](https://github.com/android-messenger/docs/blob/main/ARCHITECTURE.md)
- [Концепция безопасности](https://github.com/android-messenger/docs/tree/main/security) — модель угроз, аттестация, PKI, mTLS, RASP, E2EE
- [Создание и управление системой](https://github.com/android-messenger/docs/tree/main/ops) · [установка на телефон](https://github.com/android-messenger/docs/blob/main/install/phone.md)
- [Спецификации S1–S3](https://github.com/android-messenger/docs/tree/main/specs)

Репозитории закрытые: ссылки открываются участникам организации.

## Безопасность

Уязвимости, пожалуйста, не публикуйте в issues. Сообщайте о них владельцу организации напрямую.
