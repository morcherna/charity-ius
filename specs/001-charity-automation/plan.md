# Implementation Plan: Автоматизация деятельности благотворительной организации

**Branch**: `001-charity-automation` | **Date**: 2026-11-22 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `specs/001-charity-automation/spec.md`

## Summary

Система предоставляет единый рабочий контур для реестра членов, контрагентов,
планов доходов и расходов, первичных документов, банковских выписок и отчётности.
Серверная веб-система использует PostgreSQL и ролевой доступ. Финансовые действия
утверждаются человеком, расчёты взносов и планов выполняются детерминированно,
а выписки загружаются из файла 1C «клиент-банк».

## Technical Context

**Language/Version**: Python 3.12, FastAPI; фронтенд React.

Primary Dependencies: FastAPI, SQLAlchemy, Alembic, Pydantic; openpyxl (XLSX/CSV); ReportLab или WeasyPrint (PDF); React.

**Storage**: PostgreSQL 16.

**Testing**: Приёмочные, интеграционные, модульные сценарии и проверки журналирования.

**Target Platform**: Веб-интерфейс и сервер организации.

**Project Type**: Веб-приложение с серверной частью.

**Performance Goals**: Поиск по реестру должен выполняться до 2 с;
годовой отчёт должен формироваться до 30 с; загрузка выписки до 5000 строк
должна завершаться до 60 с.

**Constraints**: Один сервер организации; резервное копирование обеспечивает RPO до 1 часа
и восстановление в пределах RTO до 4 часов.

**Scale/Scope**: До 50000 членов и контрагентов в сумме,
до 20 одновременных пользователей; одна организация; роли, документы, планы,
банковские выписки и отчёты.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- Спецификация является исходным требованием для планирования и реализации.
- Процессы, роли, документы и отчётность не изменяются ради технической реализации.
- Платежи и финансовые документы утверждаются и проводятся только человеком.
- Расчёты взносов и планов выполняются без ИИ с детерминированным результатом.
- Данные и журнальные действия фиксируются до и после изменения.
- Журнал аудита является append-only; действия ИИ содержат гейт утверждения.
- Доступ ограничен ролью, а персональные данные защищены.
- Требования ссылаются на FR/SC и проверяемые критерии.

**Gate result**: PASS при сохранении указанных ограничений в модели и контрактах.

### Проверка после проектирования

Модель данных, контракты и сценарии быстрой проверки сохраняют гейты человека,
append-only журнал аудита, детерминированные расчёты и ролевое разграничение.
Артефакты не вводят прямую банковскую интеграцию и не требуют ИИ для расчётов или планов.

**Post-design Gate result**: PASS.

## Project Structure

### Documentation (this feature)

```text
specs/001-charity-automation/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── business-operations.md
└── spec.md
```

### Application structure

Реализация будет разделена на веб-интерфейс и серверные сценарии:

```text
frontend/
└── screens and role workflows

backend/
├── member and category registers
├── counterparty and settlement flows
├── income and expense planning
├── document preparation and approval
├── bank statement import
├── reporting and file exchange
└── audit log
```

**Structure Decision**: Веб-приложение с серверной частью и PostgreSQL.
Банк интегрируется только через импорт файла выписки.

## Complexity Tracking

Конституционных нарушений нет; таблица исключений не требуется.
