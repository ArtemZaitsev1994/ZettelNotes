Способ группировки подов, который может пригодиться в следующих случаях:
- Логическая группировка компонентов.
- Группировка сервисов, для избегания конфликтов между двумя параллельно работающими командами.
- Шарить сервисы между двумя разными окружениями.
- Ограничение лимитов для каких-то групп.

[[configmap]] и [[secrets]] не шарятся между нейспейсами, но к [[Публикация приложения (services)|сервисам]] могут иметь доступ из других неймспесов.