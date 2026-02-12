# AutoLISP — Примеры скриптов для армирования

## 1. Сетка колонн

**Задача:** Нарисовать сетку колонн 6×9м, 4 пролёта по X, 3 по Y. Сечение колонны 400×400.

```lisp
;;; Сетка колонн
(defun c:COLUMN-GRID (/ nx ny sx sy cw pt x y i j)
  (setq nx (getint "\nПролётов по X: "))
  (setq ny (getint "\nПролётов по Y: "))
  (setq sx (getreal "\nШаг по X (мм): "))
  (setq sy (getreal "\nШаг по Y (мм): "))
  (setq cw (getreal "\nСечение колонны (мм): "))
  (setq pt (getpoint "\nБазовая точка: "))

  (setq i 0)
  (repeat (+ nx 1)
    (setq j 0)
    (repeat (+ ny 1)
      (setq x (+ (car pt) (* i sx)))
      (setq y (+ (cadr pt) (* j sy)))
      ;; Прямоугольник колонны
      (command "RECTANG"
        (list (- x (/ cw 2.0)) (- y (/ cw 2.0)))
        (list (+ x (/ cw 2.0)) (+ y (/ cw 2.0)))
      )
      ;; Маркировка оси
      (command "TEXT" "J" "MC" (list x (- y (/ cw 2.0) 200)) 150 0
        (strcat (chr (+ 65 i)) (itoa (+ j 1)))
      )
      (setq j (+ j 1))
    )
    (setq i (+ i 1))
  )
  (princ)
)
```

## 2. Раскладка продольной арматуры в балке

**Задача:** Нарисовать сечение балки с продольной арматурой (верх и низ).

```lisp
;;; Сечение балки с арматурой
(defun c:BEAM-SECTION (/ bw h cover d-bot n-bot d-top n-top pt
                         x0 y0 x1 y1 r-bot r-top spacing i cx)
  (setq bw    (getreal "\nШирина балки (мм): "))
  (setq h     (getreal "\nВысота балки (мм): "))
  (setq cover (getreal "\nЗащитный слой (мм) <25>: "))
  (if (not cover) (setq cover 25.0))
  (setq d-bot (getreal "\nДиаметр нижней арматуры (мм): "))
  (setq n-bot (getint  "\nКоличество нижних стержней: "))
  (setq d-top (getreal "\nДиаметр верхней арматуры (мм): "))
  (setq n-top (getint  "\nКоличество верхних стержней: "))
  (setq pt    (getpoint "\nЛевый нижний угол сечения: "))

  (setq x0 (car pt) y0 (cadr pt))
  (setq x1 (+ x0 bw) y1 (+ y0 h))
  (setq r-bot (/ d-bot 2.0))
  (setq r-top (/ d-top 2.0))

  ;; Слой контура
  (command "LAYER" "M" "АР-Каркас" "C" "5" "" "")

  ;; Контур сечения
  (command "RECTANG" pt (list x1 y1))

  ;; Слой арматуры
  (command "LAYER" "M" "АР-Сечение" "C" "1" "" "")

  ;; Нижняя арматура
  (setq spacing (/ (- bw (* 2 cover) (* 2 r-bot)) (max (- n-bot 1) 1)))
  (setq i 0)
  (repeat n-bot
    (setq cx (+ x0 cover r-bot (* i spacing)))
    (command "CIRCLE" (list cx (+ y0 cover r-bot)) r-bot)
    (setq i (+ i 1))
  )

  ;; Верхняя арматура
  (setq spacing (/ (- bw (* 2 cover) (* 2 r-top)) (max (- n-top 1) 1)))
  (setq i 0)
  (repeat n-top
    (setq cx (+ x0 cover r-top (* i spacing)))
    (command "CIRCLE" (list cx (- y1 cover r-top)) r-top)
    (setq i (+ i 1))
  )

  ;; Слой текста
  (command "LAYER" "M" "АР-Текст" "")

  ;; Маркировка
  (command "TEXT" "J" "BC" (list (+ x0 (/ bw 2.0)) (- y0 300)) 100 0
    (strcat (itoa n-bot) "d" (rtos d-bot 2 0) " A500C (низ)")
  )
  (command "TEXT" "J" "TC" (list (+ x0 (/ bw 2.0)) (+ y1 150)) 100 0
    (strcat (itoa n-top) "d" (rtos d-top 2 0) " A500C (верх)")
  )

  (princ)
)
```

## 3. Хомуты (поперечная арматура)

**Задача:** Нарисовать хомут в сечении балки.

```lisp
;;; Хомут в сечении
(defun c:STIRRUP (/ bw h cover d-st pt x0 y0 r
                    xi yi xo yo)
  (setq bw    (getreal "\nШирина балки (мм): "))
  (setq h     (getreal "\nВысота балки (мм): "))
  (setq cover (getreal "\nЗащитный слой (мм) <25>: "))
  (if (not cover) (setq cover 25.0))
  (setq d-st  (getreal "\nДиаметр хомута (мм) <8>: "))
  (if (not d-st) (setq d-st 8.0))
  (setq pt    (getpoint "\nЛевый нижний угол сечения: "))

  (setq x0 (car pt) y0 (cadr pt))
  (setq r (/ d-st 2.0))

  ;; Внутренний контур хомута (по оси стержня)
  (setq xi (+ x0 cover r))
  (setq yi (+ y0 cover r))
  (setq xo (- (+ x0 bw) cover r))
  (setq yo (- (+ y0 h) cover r))

  (command "LAYER" "M" "АР-Хомуты" "C" "3" "" "")

  ;; Хомут — замкнутый прямоугольник (по оси стержня)
  (command "PLINE"
    (list xi yi)
    (list xo yi)
    (list xo yo)
    (list xi yo)
    "C"
  )

  ;; Крючки (загибы 135°) — упрощённо, линиями
  (command "LINE"
    (list xi yo)
    (list (+ xi (* 5 d-st)) (- yo (* 5 d-st)))
    ""
  )

  (princ)
)
```

## 4. Раскладка стержней вдоль балки (продольный вид)

**Задача:** Разложить арматурные стержни вдоль балки с указанием длин.

```lisp
;;; Раскладка арматуры вдоль балки
(defun c:REBAR-LAYOUT (/ L n-bot d-bot cover anc pt
                         x0 y0 spacing rebar-L i cy)
  (setq L     (getreal "\nДлина балки (мм): "))
  (setq n-bot (getint  "\nКоличество стержней: "))
  (setq d-bot (getreal "\nДиаметр (мм): "))
  (setq cover (getreal "\nЗащитный слой (мм) <25>: "))
  (if (not cover) (setq cover 25.0))
  (setq anc   (getreal "\nДлина анкеровки (мм) <0>: "))
  (if (not anc) (setq anc 0.0))
  (setq pt    (getpoint "\nНачало балки (левый низ): "))

  (setq x0 (car pt) y0 (cadr pt))
  (setq rebar-L (+ L (* 2 anc)))
  (setq spacing (/ d-bot 1.5)) ; расстояние между линиями стержней на виде

  (command "LAYER" "M" "АР-Арматура" "C" "1" "" "")

  ;; Рисуем стержни
  (setq i 0)
  (repeat n-bot
    (setq cy (+ y0 cover (/ d-bot 2.0) (* i spacing)))
    (command "LINE"
      (list (- x0 anc) cy)
      (list (+ x0 L anc) cy)
      ""
    )
    (setq i (+ i 1))
  )

  ;; Размер длины стержня
  (command "LAYER" "M" "АР-Размеры" "")
  (command "DIMLINEAR"
    (list (- x0 anc) (+ y0 cover))
    (list (+ x0 L anc) (+ y0 cover))
    (list (/ (+ x0 (+ x0 L)) 2.0) (- y0 200))
  )

  ;; Маркировка
  (command "LAYER" "M" "АР-Текст" "")
  (command "TEXT" "J" "BC"
    (list (+ x0 (/ L 2.0)) (- y0 500)) 100 0
    (strcat (itoa n-bot) "d" (rtos d-bot 2 0) " A500C L=" (rtos rebar-L 2 0))
  )

  (princ)
)
```

## 5. Спецификация арматуры (таблица)

**Задача:** Сгенерировать таблицу спецификации арматуры на чертеже.

```lisp
;;; Спецификация арматуры
(defun c:REBAR-SPEC (/ pt x0 y0 rh cw items i item
                       pos-x name-x diam-x len-x qty-x mass-x)
  (setq pt (getpoint "\nЛевый верхний угол таблицы: "))
  (setq x0 (car pt) y0 (cadr pt))
  (setq rh 300)  ; высота строки

  ;; Ширины колонок
  (setq pos-x  400)   ; Поз.
  (setq name-x 1200)  ; Наименование
  (setq diam-x 600)   ; Диаметр
  (setq len-x  800)   ; Длина
  (setq qty-x  500)   ; Кол-во
  (setq mass-x 800)   ; Масса

  ;; Данные (пример — заменить на реальные)
  (setq items (list
    '("1" "КР-1" "12" "2400" "4" "10.66")
    '("2" "КР-2" "16" "3100" "6" "29.49")
    '("3" "Хомут" "8"  "1200" "24" "5.69")
  ))

  (command "LAYER" "M" "АР-Текст" "")

  ;; Заголовок
  (setq cw (+ pos-x name-x diam-x len-x qty-x mass-x))
  (command "RECTANG" (list x0 y0) (list (+ x0 cw) (- y0 rh)))
  (command "TEXT" "J" "MC" (list (+ x0 (/ pos-x 2.0)) (- y0 (/ rh 2.0))) 120 0 "Поз.")
  (command "TEXT" "J" "MC" (list (+ x0 pos-x (/ name-x 2.0)) (- y0 (/ rh 2.0))) 120 0 "Наименование")
  (command "TEXT" "J" "MC" (list (+ x0 pos-x name-x (/ diam-x 2.0)) (- y0 (/ rh 2.0))) 120 0 "d, мм")
  (command "TEXT" "J" "MC" (list (+ x0 pos-x name-x diam-x (/ len-x 2.0)) (- y0 (/ rh 2.0))) 120 0 "L, мм")
  (command "TEXT" "J" "MC" (list (+ x0 pos-x name-x diam-x len-x (/ qty-x 2.0)) (- y0 (/ rh 2.0))) 120 0 "Шт.")
  (command "TEXT" "J" "MC" (list (+ x0 pos-x name-x diam-x len-x qty-x (/ mass-x 2.0)) (- y0 (/ rh 2.0))) 120 0 "Масса, кг")

  ;; Данные
  (setq i 0)
  (foreach item items
    (setq i (+ i 1))
    (setq cy (- y0 (* rh (+ i 0.5))))
    (command "RECTANG"
      (list x0 (- y0 (* rh i)))
      (list (+ x0 cw) (- y0 (* rh (+ i 1))))
    )
    (command "TEXT" "J" "MC" (list (+ x0 (/ pos-x 2.0)) cy) 100 0 (nth 0 item))
    (command "TEXT" "J" "MC" (list (+ x0 pos-x (/ name-x 2.0)) cy) 100 0 (nth 1 item))
    (command "TEXT" "J" "MC" (list (+ x0 pos-x name-x (/ diam-x 2.0)) cy) 100 0 (nth 2 item))
    (command "TEXT" "J" "MC" (list (+ x0 pos-x name-x diam-x (/ len-x 2.0)) cy) 100 0 (nth 3 item))
    (command "TEXT" "J" "MC" (list (+ x0 pos-x name-x diam-x len-x (/ qty-x 2.0)) cy) 100 0 (nth 4 item))
    (command "TEXT" "J" "MC" (list (+ x0 pos-x name-x diam-x len-x qty-x (/ mass-x 2.0)) cy) 100 0 (nth 5 item))
  )

  (princ)
)
```

## 6. Расчёт массы арматуры

**Задача:** Посчитать массу арматуры по диаметру и длине.

```lisp
;;; Калькулятор массы арматуры
(defun c:REBAR-MASS (/ d L n mass-per-m total)
  ;; Масса погонного метра по ГОСТ 34028-2016
  (defun mass-per-meter (d / table)
    (setq table '(
      (6 . 0.222) (8 . 0.395) (10 . 0.617) (12 . 0.888)
      (14 . 1.208) (16 . 1.578) (18 . 1.998) (20 . 2.466)
      (22 . 2.984) (25 . 3.853) (28 . 4.834) (32 . 6.313)
      (36 . 7.990) (40 . 9.865)
    ))
    (cdr (assoc d table))
  )

  (setq d (getint "\nДиаметр арматуры (мм): "))
  (setq L (getreal "\nДлина стержня (мм): "))
  (setq n (getint "\nКоличество стержней: "))

  (setq mass-per-m (mass-per-meter d))
  (if mass-per-m
    (progn
      (setq total (* mass-per-m (/ L 1000.0) n))
      (princ (strcat
        "\n─────────────────────────"
        "\nДиаметр: d" (itoa d)
        "\nДлина: " (rtos L 2 0) " мм"
        "\nКоличество: " (itoa n) " шт."
        "\nМасса 1 п.м.: " (rtos mass-per-m 2 3) " кг"
        "\nМасса 1 стержня: " (rtos (* mass-per-m (/ L 1000.0)) 2 2) " кг"
        "\nОбщая масса: " (rtos total 2 2) " кг"
        "\n─────────────────────────"
      ))
    )
    (princ (strcat "\nНеизвестный диаметр: d" (itoa d) ". Допустимые: 6,8,10,12,14,16,18,20,22,25,28,32,36,40"))
  )
  (princ)
)
```

## 7. Армирование колонны (полное сечение)

**Задача:** Нарисовать сечение колонны с арматурой по периметру + хомут.

```lisp
;;; Армирование сечения колонны
(defun c:COL-REINF (/ pt b h cover ds dh n-b n-h step-b step-h
                      x0 y0 cx cy i j)
  (setq pt    (getpoint "\nЛевый нижний угол: "))
  (setq b     (getreal "\nШирина b (мм): "))
  (setq h     (getreal "\nВысота h (мм): "))
  (setq cover (getreal "\nЗащитный слой (мм) <20>: "))
  (if (not cover) (setq cover 20.0))
  (setq ds    (getreal "\nДиаметр рабочей (мм): "))
  (setq dh    (getreal "\nДиаметр хомута (мм) <8>: "))
  (if (not dh) (setq dh 8.0))
  (setq n-b   (getint "\nСтержней по ширине: "))
  (setq n-h   (getint "\nСтержней по высоте: "))

  (setq x0 (car pt) y0 (cadr pt))

  ;; Контур сечения
  (command "_.RECTANG" pt (list (+ x0 b) (+ y0 h)))

  ;; Хомут
  (command "_.RECTANG"
    (list (+ x0 cover (/ dh 2.0)) (+ y0 cover (/ dh 2.0)))
    (list (- (+ x0 b) cover (/ dh 2.0)) (- (+ y0 h) cover (/ dh 2.0)))
  )

  ;; Стержни по периметру (DONUT для закрашенных сечений)
  (setq step-b (/ (- b (* 2 cover) dh ds) (max (1- n-b) 1)))
  (setq step-h (/ (- h (* 2 cover) dh ds) (max (1- n-h) 1)))
  (setq cx (+ x0 cover (/ dh 2.0) (/ ds 2.0)))
  (setq cy (+ y0 cover (/ dh 2.0) (/ ds 2.0)))

  ;; Нижний ряд
  (setq i 0)
  (repeat n-b
    (command "_.DONUT" 0.0 ds (list (+ cx (* i step-b)) cy) "")
    (setq i (1+ i))
  )
  ;; Верхний ряд
  (setq i 0)
  (repeat n-b
    (command "_.DONUT" 0.0 ds (list (+ cx (* i step-b)) (+ cy (* (1- n-h) step-h))) "")
    (setq i (1+ i))
  )
  ;; Боковые промежуточные
  (setq j 1)
  (repeat (- n-h 2)
    (command "_.DONUT" 0.0 ds (list cx (+ cy (* j step-h))) "")
    (command "_.DONUT" 0.0 ds (list (+ cx (* (1- n-b) step-b)) (+ cy (* j step-h))) "")
    (setq j (1+ j))
  )
  (princ)
)
```

## 8. Выноска позиции арматуры

**Задача:** Выноска с номером позиции, диаметром и шагом.

```lisp
;;; Выноска арматуры: "поз.N ∅d A500C ш.200"
(defun c:REBAR-LABEL (/ pt1 pt2 pos-num diam class step txt-h label)
  (setq pt1     (getpoint "\nТочка на стержне: "))
  (setq pt2     (getpoint pt1 "\nТочка текста: "))
  (setq pos-num (getint "\nНомер позиции: "))
  (setq diam    (getint "\nДиаметр (мм): "))
  (setq class   (getstring "\nКласс (A500C/A400/A240): "))
  (setq step    (getint "\nШаг (мм, 0=без шага): "))
  (setq txt-h   100)

  ;; Формируем строку (%%C = символ диаметра ∅)
  (if (= step 0)
    (setq label (strcat "поз." (itoa pos-num) " %%C" (itoa diam) " " class))
    (setq label (strcat "поз." (itoa pos-num) " %%C" (itoa diam) " " class " ш." (itoa step)))
  )

  ;; Линия-выноска + текст
  (command "_.LINE" pt1 pt2 "")
  (command "_.TEXT" pt2 txt-h 0 label)
  (princ)
)
```

## 9. Простая типизация — группировка каркасов

**Задача:** Пользователь вводит параметры каркасов, скрипт группирует одинаковые.

```lisp
;;; Типизация каркасов (интерактивный ввод)
(defun c:REBAR-TYPE (/ frames frame d-work n-work L-work d-stir step-stir
                       types type-id found match tol i j)
  (setq frames '())
  (setq tol 50) ; допуск по длине, мм

  ;; Ввод каркасов
  (princ "\nВвод каркасов (Enter без диаметра для завершения):\n")
  (setq i 1)
  (while
    (progn
      (princ (strcat "\n--- Каркас #" (itoa i) " ---"))
      (setq d-work (getint "\n  Диаметр рабочей арматуры (мм, Enter=стоп): "))
      d-work
    )
    (setq n-work   (getint  "\n  Количество стержней: "))
    (setq L-work   (getreal "\n  Длина стержня (мм): "))
    (setq d-stir   (getint  "\n  Диаметр хомута (мм): "))
    (setq step-stir(getreal "\n  Шаг хомутов (мм): "))

    (setq frame (list d-work n-work L-work d-stir step-stir))
    (setq frames (append frames (list frame)))
    (setq i (+ i 1))
  )

  ;; Типизация
  (setq types '())
  (setq type-id 0)
  (foreach frame frames
    (setq found nil)
    (setq j 0)
    (foreach tp types
      (setq match (car tp))
      (if (and
            (= (nth 0 frame) (nth 0 match))   ; диаметр рабочей
            (= (nth 1 frame) (nth 1 match))   ; количество
            (<= (abs (- (nth 2 frame) (nth 2 match))) tol) ; длина ±50
            (= (nth 3 frame) (nth 3 match))   ; диаметр хомута
            (= (nth 4 frame) (nth 4 match))   ; шаг хомутов
          )
        (progn
          (setq found t)
          ;; Увеличить счётчик
          (setq types (subst
            (list match (+ (cadr tp) 1))
            tp
            types
          ))
        )
      )
      (setq j (+ j 1))
    )
    (if (not found)
      (progn
        (setq type-id (+ type-id 1))
        (setq types (append types (list (list frame 1))))
      )
    )
  )

  ;; Вывод
  (princ "\n\n══════ РЕЗУЛЬТАТ ТИПИЗАЦИИ ══════\n")
  (setq i 1)
  (foreach tp types
    (setq frame (car tp))
    (princ (strcat
      "\nКР-" (itoa i) ": "
      (itoa (nth 1 frame)) "d" (itoa (nth 0 frame))
      " L=" (rtos (nth 2 frame) 2 0)
      ", хомуты d" (itoa (nth 3 frame))
      " шаг " (rtos (nth 4 frame) 2 0)
      " — " (itoa (cadr tp)) " шт."
    ))
    (setq i (+ i 1))
  )
  (princ (strcat "\n\nВсего каркасов: " (itoa (length frames))
                 ", типов: " (itoa (length types)) "\n"))
  (princ)
)
```
