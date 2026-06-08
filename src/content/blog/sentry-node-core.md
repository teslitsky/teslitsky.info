---
title: '@sentry/node-core: Sentry для помилок без конфліктів з OpenTelemetry'
description: 'Новий пакет @sentry/node-core, який не патчить OpenTelemetry і дає працювати у власному кастомному стеку – без магії.'
pubDate: 2025-08-21
---

Використовуєте Sentry лише для помилок, а Observability – окремо?

Є сенс звернути увагу на новий пакет `@sentry/node-core`.

Він не тягне за собою свої версії OpenTelemetry залежностей, не патчить пакети та не налаштовує експортери за замовчуванням. Тобто працює лише у вашому кастомному стеку – без магії.

Я використовую Sentry в NestJS-додатках для відстеження помилок та моніторингу
cron'ів. А traces з OpenTelemetry надсилаю деінде. Із попереднім SDK це було
незручно – доводилося лізти в сорси Sentry й OpenTelemetry, щоб зрозуміти, як
уникнути конфліктів.

## Було так

```ts
import * as Sentry from '@sentry/nestjs';
import { SentryPropagator } from '@sentry/opentelemetry';
import { CustomSampler } from './sampler';

const sdk = new NodeSDK({
  traceExporter,
  sampler: new CustomSampler(),
  spanProcessors: [new BatchSpanProcessor(traceExporter)],
  contextManager: new Sentry.SentryContextManager(),
  textMapPropagator: new SentryPropagator(),
  instrumentations: [...],
});
sdk.start();
```

У попередній версії ще бажано було додавати:

```ts
Sentry.init({
  skipOpenTelemetrySetup: true,
  registerEsmLoaderHooks: false,
});
```

## Стало так

```ts
const sdk = new NodeSDK({
  traceExporter,
  spanProcessors: [new BatchSpanProcessor(traceExporter)],
  instrumentations: [...],
});
sdk.start();
```

У `@sentry/node-core` цього не потрібно – він не втручається в OTel, доки ви самі
не захочете.

## Висновок

Якщо вам потрібен Sentry тільки для помилок – `@sentry/node-core` ідеально
підходить:

- менше сторонньої магії
- більше контролю
- менше шансів на поламану інтеграцію з OTel
- швидше старт додатку через меншу кількість патчів пакетів

Рекомендую спробувати замість `@sentry/nestjs`. Також варто спочатку прочитати
[Migration guide](https://docs.sentry.io/platforms/javascript/guides/node/migration/).

---

_Уперше опубліковано в моєму [Telegram-каналі](https://t.me/techleadsky/9)._
