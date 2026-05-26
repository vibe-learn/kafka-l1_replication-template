        # kafka — Репликация: лидеры, последователи, ISR

        Homework-шаблон для урока **l1_replication** (Репликация: лидеры, последователи, ISR) на платформе Vibe Learn.

        ## Что делать

        Реализуй Go-программу, которая демонстрирует поведение репликации Kafka под нагрузкой.

**Окружение:** docker-compose с 3 брокерами Kafka (broker-1, broker-2, broker-3) и
встроенным KRaft-контроллером.

**Шаги:**
1. Запусти продьюсер с `RequiredAcks: kafka.RequireAll` и `replication.factor=3`,
   `min.insync.replicas=2`. Убедись, что записи проходят.
2. Убей первый брокер (`docker kill broker-1`). Проверь, что записи продолжаются
   после короткого failover-паузы (лидер сменился).
3. Убей второй брокер (`docker kill broker-2`). При `min.insync.replicas=2` запись
   должна зависнуть (ISR < 2): producer получает `NotEnoughReplicasException`.
4. Восстанови брокеры и убедись, что записи возобновляются.

**CI-проверки в template repo:**
- `assert writes_after_kill_1 > 0` — запись продолжалась при одном упавшем брокере.
- `assert writes_during_kill_2 == 0` — запись заблокировалась при двух упавших брокерах.
- `assert writes_after_recovery > 0` — запись восстановилась.

## Контекст (из transfer-задачи урока)

У вас топик `payment_events` с `replication.factor=3` и `min.insync.replicas=2`.
Один из трёх брокеров падает на 4 часа.

**Ответьте на три вопроса:**

## Recap из урока

- **Leader** — единственная реплика, принимающая записи от продьюсеров. **Follower** только реплицирует данные с лидера через FetchRequest.
- **ISR (In-Sync Replicas)** — множество актуальных реплик, которые не отстали от лидера дольше `replica.lag.time.max.ms`. Только ISR-реплика может безопасно стать новым лидером без потери данных.
- При падении лидера Kafka-контроллер выбирает нового лидера **только из ISR** (при `unclean.leader.election.enable=false`). Это гарантирует отсутствие потери подтверждённых записей.
- **`unclean.leader.election.enable=true`** восстанавливает availability когда ISR пуст, но **гарантированно теряет данные**. Для production с критическими данными оставляй `false`.
- **Production standard: `replication.factor=3`**. Один брокер может упасть — ISR ещё содержит две реплики. `replication.factor=1` для критических данных — анти-паттерн: потеря брокера = потеря данных навсегда.

        ## Как работать

        1. Платформа Vibe Learn создаёт копию этого репо в твоём GitHub-аккаунте по клику «Начать домашку» на странице урока (через GitHub `/generate`, codecrafters-pattern).
        2. Склонируй копию локально, реализуй TODO в `main.go`, прогони тесты, запушь.
        3. CI (`.github/workflows/ci.yml`) запускает `go vet` + `go test ./...` на каждый push. Платформа слушает результат через webhook от GitHub Actions и обновляет статус домашки на странице урока.

        ## Локальное окружение

        - Go 1.22+
        - Docker + docker-compose — `docker compose -f docker-compose.yml up -d` поднимает 3-нодовый Kafka cluster на портах 9092/9093/9094, использовать в тестах через bootstrap `localhost:9092,localhost:9093,localhost:9094`.

        ## Запуск

        ```bash
        # Поднять локальный Kafka
        docker compose up -d

        # Прогнать тесты (часть из них стартует свой ephemeral testcontainers cluster, часть использует docker-compose выше)
        go test ./...

        # Запустить main (печатает marker; замени stub на реализацию)
        go run .
        ```

        ## Заметка автора

        Это baseline-шаблон, сгенерированный платформой. Бизнес-сущность задачи (что конкретно реализовать в `main.go`, какие тесты сделать строгими) расширяется по ходу итераций — параллельно с углублением теории урока.
