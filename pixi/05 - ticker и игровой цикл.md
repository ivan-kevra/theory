# Ticker и игровой цикл

> **Ticker** — `requestAnimationFrame` с delta time и единым FPS-лимитом для логики и (в связке с Application) рендера.

## Базовое использование

```ts
import { Application } from 'pixi.js';

const app = new Application();
await app.init();

app.ticker.add((ticker) => {
  // ticker.deltaTime — множитель ~1 при 60 FPS (frame-independent)
  // ticker.deltaMS — миллисекунды с прошлого кадра
  player.x += speed * ticker.deltaTime;
});

app.ticker.maxFPS = 60; // 0 = без лимита
app.ticker.stop();
app.ticker.start();
```

## deltaTime vs deltaMS

| Поле | Назначение |
|------|------------|
| `deltaMS` | Реальное время кадра в мс |
| `deltaTime` | `deltaMS / (1000/60)` — «сколько 60-FPS кадров прошло» |

**На собесе:** движение через `speed * deltaTime` — скорость не зависит от 30 vs 60 FPS.

```ts
// Плохо: привязка к FPS
sprite.x += 5;

// Хорошо
sprite.x += 300 * (ticker.deltaMS / 1000); // 300 px/s
```

## Один ticker на приложение

- `app.ticker` — общий цикл.
- Несколько подписчиков: `add(fn)`, удаление: `remove(fn)` или `destroy()`.
- **Утечка:** забытый listener держит замыкания → всегда `remove` при destroy сцены.

## Связь с рендером

`Application` после ticker обновляет отображение (если не отключено). Отдельный ручной loop нужен редко:

```ts
// Кастомный loop без Application.ticker auto-render
import { Ticker } from 'pixi.js';

const ticker = Ticker.shared;
ticker.add(() => {
  update();
  renderer.render(stage);
});
```

## FPS и профилирование

```ts
console.log(app.ticker.FPS); // сглаженное значение
```

Просадки: тяжёлый update в ticker, синхронная загрузка, GC JS, много draw calls.

## Пауза игры

```ts
app.ticker.stop();
// или
app.ticker.speed = 0; // v8: заморозка времени для всех подписчиков
```

UI поверх паузы — отдельный ticker или DOM вне Pixi.

## Частые вопросы

**requestAnimationFrame vs setInterval?**  
rAF синхронизирован с vsync, не рисует в скрытой вкладке (экономия), Pixi ticker = rAF.

**Fixed timestep?**  
Pixi не даёт из коробки; на собесе: накапливать `accumulator += deltaMS`, шаг физики 16.67ms в цикле while.

**Два ticker?**  
`Ticker.shared` и свой экземпляр — возможно, но обычно один источник правды.

## Шпаргалка

> Ticker — rAF-цикл с deltaTime для frame-independent логики. Подписываемся через add, отписываемся при уничтожении. Application связывает ticker с render. Скорость в «единицах на секунду», умножать на deltaMS/1000 или deltaTime.

---

**←** [04 - рендеринг](./04%20-%20рендеринг.md) · **→** [06 - события и интерактивность](./06%20-%20события%20и%20интерактивность.md)
