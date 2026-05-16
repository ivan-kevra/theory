# Graphics и текст

> **Graphics** — векторные примитивы (линии, заливки). **Text** — canvas/raster текст. **BitmapText** — глифы из bitmap font (быстрее для частых обновлений).

## Graphics

```ts
import { Graphics } from 'pixi.js';

const g = new Graphics();
g.rect(0, 0, 100, 50).fill(0xff0000);
g.circle(50, 50, 20).stroke({ width: 2, color: 0xffffff });

app.stage.addChild(g);
```

- В v8 API **chainable**: `rect().fill()`, `moveTo().lineTo().stroke()`.
- Геометрия пересобирается при изменении — частые перерисовки **дороги** (новая mesh/batch).
- Для статичных фигур нарисовал один раз; для анимированных линий — ограничить частоту `clear()` + redraw.

**На собесе:** тысячи динамических Graphics хуже, чем спрайты из атласа; UI часто — NineSliceSprite + текстуры.

## Graphics vs Sprite

| | Graphics | Sprite |
|---|----------|--------|
| Источник | Процедурная геометрия | Текстура |
| Batching | Отдельный пайплайн | Основной sprite batch |
| Лучше для | Отладка, простые формы, маски-вектор | Игровая графика |

## Text (обычный)

```ts
import { Text } from 'pixi.js';

const label = new Text({
  text: 'Score: 0',
  style: {
    fontFamily: 'Arial',
    fontSize: 24,
    fill: 0xffffff,
  },
});
```

- Рендер через **canvas** → текстура на GPU.
- **Смена текста/стиля** — перерисовка canvas → дорого при каждом кадре.
- Для счётчика очков — обновлять только при изменении значения.

```ts
label.text = `Score: ${score}`; // не каждый frame без нужды
```

## BitmapText

```ts
import { BitmapText, Assets } from 'pixi.js';

await Assets.load('fonts/myfont.fnt');
const bmp = new BitmapText({
  text: '9999',
  style: { fontFamily: 'myfont', fontSize: 32 },
});
```

- Символы — спрайты из атласа глифов.
- Быстро для **часто меняющихся** чисел (счёт, таймер).
- Нужен заранее подготовленный **bitmap font** (BMFont, etc.).

## Text vs BitmapText vs HTML

| Подход | Плюсы | Минусы |
|--------|-------|--------|
| Text | Любой шрифт, стили | Дорогой update |
| BitmapText | Производительность | Ограниченный набор глифов |
| HTML overlay | Сложная вёрстка | Не в WebGL batch, другой слой DOM |

## destroy

```ts
text.destroy({ style: true });
graphics.destroy({ context: true });
```

## Частые вопросы

**Почему текст размытый?**  
Низкое `resolution`, масштабирование Text без учёта DPR — поднять resolution renderer или fontSize.

**Как много текста на экране?**  
BitmapText, один Text на блок, избегать сотен Text с обновлением каждый кадр.

## Шпаргалка

> Graphics — процедурная векторная графика, не для тяжёлой анимации каждый кадр. Text — гибкий, но перерисовывает текстуру при изменении. BitmapText — для динамических цифр/лейблов в играх. Статику рисуем редко, динамику — спрайты или bitmap.

---

**←** [06 - события и интерактивность](./06%20-%20события%20и%20интерактивность.md) · **→** [08 - фильтры и маски](./08%20-%20фильтры%20и%20маски.md)
