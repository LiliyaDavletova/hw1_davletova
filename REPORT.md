# ДЗ 1 «Фотолаборатория» — отчёт

**Автор:** Лилия Давлетова · **Дата сдачи:** 10.10.2026

## 1. Числа

### 1.1. Гамма (`data/dark.jpg`)

| γ | среднее | медиана | p1 | p99 | обрезано чёрных, % | обрезано белых, % |
|---|---------|---------|----|-----|--------------------|-------------------|
| исходник | 46.5 | 38 | 3 | 117 | 0.01 | 0.00 |
| 0.5 | 96.8 | 98 | 26 | 173 | 0.00 | 0.00 |
| 2.0 | 14.0 | 6 | 0 | 54 | 34.01 | 0.00 |

### 1.2. Баланс белого (`data/bright.jpg`)

| Метод | усиление B, G, R | средние до | средние после |
|-------|------------------|-----------|---------------|
| grey-world | 0.949, 1.025, 1.030 | 172.4, 159.7, 159.1 | 163.7, 163.6, 163.6 |
| white-patch (p99) | 1.004, 1.012, 1.012 | 172.4, 159.7, 159.1 | 173.1, 161.6, 161.0 |

### 1.3. `equalizeHist` против CLAHE (`data/dark.jpg`, clip 2.0, tiles 8)

| Кадр | σ в тенях | среднее в тенях | среднее в светах |
|------|-----------|-----------------|------------------|
| исходный | 0.6 | 2.2 | 108.8 |
| equalizeHist | 11.7 | 39.1 | 229.5 |
| CLAHE | 1.5 | 7.7 | 137.9 |

### 1.4. Хромакей (`data/object.jpg`, границы HSV)

| Что | Значение |
|-----|----------|
| границы lo / hi | lo = (78, 50, 80), hi = (105, 255, 255) |
| доля маски, % | 2.92 |
| компонент до / после морфологии | 318 / 23; крупнейшая компонента использована для композита |
| дыр | 1 |
| `ball-lamp.jpg` курса: recall / precision | 91.4 % / 100.0 % |

### 1.5. Геометрия

| Что | Значение |
|-----|----------|
| поворот 30°: размер до → после, пустой холст, % | 960 × 1280 → 1471 × 1589, пустой холст 47.4 % |
| уменьшение ×0.25: PSNR naive / blur / area к Lanczos, своё фото | 30.2 / 38.9 / 38.8 дБ (`data/text.jpg`) |
| то же на `iu-building.jpg` | 27.9 / 36.0 / 37.2 дБ |
| выпрямление: стороны в кадре, `--aspect`, размер выхода | top 551.9, right 1131.0, bottom 721.1, left 828.1 px; aspect 0.707; выход 721 × 1020 |
| `poster-angle.jpg` курса: совпадение с `poster-gt.png`, % | 99.1 % |

### 1.6. Контактный лист

N = 10, сетка 2 × 5, размер листа 664 × 1378 — `out/sheet.jpg`.

## 2. Три вопроса

**2.1. Какое цветовое пространство выбрано для хромакея и почему? Что было бы в RGB?**

Я использовала HSV. Для щётки важен оттенок голубого/бирюзового, а яркость меняется из-за бликов и теней. В HSV можно отдельно ограничить H и S, а V оставить широким диапазоном. В RGB тот же объект хуже отделяется от пёстрого фона, потому что яркость и цвет смешаны в каждом канале: тень на объекте может стать похожей на фон. На моём кадре маска HSV занимает 2.92 % изображения; на контрольном `ball-lamp.jpg` HSV дал recall 91.4 % и precision 100.0 %.

**2.2. Что происходит с гистограммой при γ < 1 и при γ > 1? Куда деваются обрезанные пиксели?**

При γ < 1 тёмные значения растягиваются вправо: средняя яркость выросла с 46.5 до 96.8, p1 поднялся с 3 до 26, детали в тенях стали виднее. При γ > 1 значения сдвигаются влево: средняя яркость стала 14.0, медиана упала до 6, а 34.01 % пикселей стали чёрными. Обрезанные пиксели — это значения, которые после округления и `clip` попали в 0 или 255; детали внутри таких областей уже не различаются.

**2.3. Почему CLAHE лучше `equalizeHist` на вашем фото — по числам из 1.3? Если не лучше — что измерили и почему?**

На моём тёмном фото CLAHE выглядит осторожнее: среднее в светах поднялось до 137.9, а у `equalizeHist` до 229.5, то есть глобальная эквализация сильно переосветляет яркие области. В тенях `equalizeHist` даёт σ = 11.7 против 1.5 у CLAHE: он сильнее вытягивает шум и артефакты. Поэтому для этого кадра CLAHE лучше как более локальный и менее агрессивный способ усилить детали.

## 3. Где сломалось

Сложнее всего сработал хромакей на `data/object.jpg`: фон очень пёстрый и на нём есть голубые линии, похожие на цвет щётки. Поэтому после морфологии осталось несколько фрагментов и одна дырка в маске. Помогло бы снять объект на более однотонном фоне или добавить ручную маску/точнее выбрать цветовой диапазон.

## 4. Команды

```bash
python photolab.py gamma data/dark.jpg --gamma 0.5 -o out/dark_g05.jpg --hist out/dark_g05_hist.png
python photolab.py gamma data/dark.jpg --gamma 2.0 -o out/dark_g20.jpg --hist out/dark_g20_hist.png
python photolab.py wb data/bright.jpg --method grey-world -o out/bright_wb_grey.jpg --hist out/bright_wb_grey_hist.png
python photolab.py wb data/bright.jpg --method white-patch --percentile 99 -o out/bright_wb_patch.jpg --hist out/bright_wb_patch_hist.png
python photolab.py clahe data/dark.jpg --clip 2.0 --tiles 8 -o out/dark_clahe.jpg --compare out/dark_clahe_compare.jpg --hist out/dark_clahe_hist.png
python photolab.py chromakey data/object.jpg --bg data/bg.jpg --lo 78,50,80 --hi 105,255,255 --open 5 --close 15 --feather 2.0 -o out/object_chromakey.jpg --mask out/object_mask.png
python photolab.py rotate data/poster.jpg --deg 30 -o out/poster_rot30.jpg
python photolab.py resize data/text.jpg --scale 0.25 --method naive --report -o out/text_resize_naive.jpg
python photolab.py resize data/text.jpg --scale 0.25 --method blur --report -o out/text_resize_blur.jpg
python photolab.py resize data/text.jpg --scale 0.25 --method area --report -o out/text_resize_area.jpg
python photolab.py rectify data/poster.jpg --corners 150,318,622,32,779,1152,58,1141 --aspect 0.707 -o out/poster_rectified.jpg
python photolab.py sheet data/dark.jpg out/dark_g05.jpg data/bright.jpg out/bright_wb_grey.jpg data/object.jpg out/object_chromakey.jpg data/poster.jpg out/poster_rectified.jpg data/text.jpg out/text_resize_area.jpg --cols 2 --cell-w 320 --cell-h 240 -o out/sheet.jpg
```

## 5. Инструменты

Код написан в `photolab.py` на основе материалов лабораторных занятий 1–5. Для проверки использовались автотесты из шаблона и прямой запуск функций/CLI. При подготовке использовался ИИ-помощник ChatGPT/Codex: он помог перенести решения из лабораторных в шаблон, подобрать параметры для фото и заполнить отчёт.
