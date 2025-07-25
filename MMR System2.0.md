
### **1. Перевод эмблем**

* Bronze  = 0.25
* Silver  = 0.5
* Gold    = 0.75
* Iridescent = 1.0

---

### **2. Для каждого сурва определяем:**

* **max\_emblem** = максимальный уровень эмблемы
* **multirole** = True, если хотя бы 2 эмблемы ≥ 0.7 (Gold или Iridescent)
* **escaped** = True/False (сбежал или нет)

---

### **3. Определение базового значения (без учета ММР и командного результата):**

| Статус | max\_emblem <0.5 | 0.5–0.74 |  0.75+  |
| ------ | :--------------: | :------: | :-----: |
| Умер   |      **-15**     |  **-7**  |  **-2** |
| Сбежал |      **+7**      |  **+18** | **+28** |

* **multirole**: +4 (если True)
* **TEAM\_RESULT**: победа +3, ничья 0, поражение -3

---

### **4. Коррекция по разнице в MMR**

* **mmr\_ratio = mmr\_self / mmr\_killer** (если киллер)
* Если mmr\_ratio < 0.5: итог × 1.15 (Underdog boost)
* Если mmr\_ratio > 2: итог × 0.7 (за “нагиб слабых”)
* **Ограничения:** минимум -20, максимум +35

---

### **5. Для киллера — аналогично, только status:**

* 0–1 сбежал: победа (TEAM\_RESULT=1), 2 ничья (0), 3–4 сбежали: поражение (-1)
* Аналогично рассчитываем максимальную эмблему и мультироль
* Умершие сурвы = "выжившие" для киллера (инверсия)

---

## **Пошаговый расчет для каждого игрока:**

1. Получить список эмблем, перевести в числа.
2. Найти max\_emblem.
3. Проверить мультироль (2+ эмблемы ≥ 0.7).
4. Определить сбежал или умер.
5. Посчитать базовое значение по таблице.
6. Прибавить/вычесть за командный результат.
7. Прибавить за мультироль.
8. Умножить на коэффициент из разницы по MMR.
9. Ограничить итог -20..+35.

---

## **Пример: средний матч**

|    | OBJ    | CHASE  | ALTR   | SURV   | Итог   | max\_emblem | multirole | Эскейп | База | Team | Итог (до ММР) |
| -- | ------ | ------ | ------ | ------ | ------ | ----------- | --------- | ------ | ---- | ---- | ------------- |
| S1 | Silver | Gold   | Silver | Gold   | Сбежал | 0.75        | Да        | Да     | 28   | 0    | 32            |
| S2 | Silver | Silver | Silver | Silver | Сбежал | 0.5         | Нет       | Да     | 18   | 0    | 18            |
| S3 | Silver | Silver | Gold   | Silver | Умер   | 0.75        | Да        | Нет    | -2   | 0    | 2             |
| S4 | Gold   | Silver | Bronze | Silver | Умер   | 0.75        | Нет       | Нет    | -2   | 0    | -2            |

* TEAM\_RESULT: ничья, бонус/штраф 0.
* mmr\_self = 1000, mmr\_killer = 1000 → коэф. 1

**Результаты (ограничиваем по диапазону):**

|    | Итог MMR |
| -- | -------- |
| S1 | 32       |
| S2 | 18       |
| S3 | 2        |
| S4 | -2       |

---

### **Худший вариант (в среднем матче):**

* S4: умер, max\_emblem 0.75, мультироли нет, бонусов нет, итог -2 (или еще хуже если эмблемы слабее, например -7)

---

### **Лучший вариант:**

* S1: сбежал, 2+ эмблемы Gold, бонус, максимум: 32 (но не больше 35)

---

### **В плохом матче (пример):**

|    | OBJ    | CHASE  | ALTR   | SURV   | Итог   | max\_emblem | multirole | Эскейп | База | Team | Итог (до ММР) |
| -- | ------ | ------ | ------ | ------ | ------ | ----------- | --------- | ------ | ---- | ---- | ------------- |
| S1 | Bronze | Silver | Bronze | Bronze | Умер   | 0.5         | Нет       | Нет    | -7   | -3   | -10           |
| S2 | Bronze | Bronze | Bronze | Bronze | Умер   | 0.25        | Нет       | Нет    | -15  | -3   | -18           |
| S3 | Silver | Bronze | Silver | Bronze | Умер   | 0.5         | Нет       | Нет    | -7   | -3   | -10           |
| S4 | Gold   | Gold   | Silver | Gold   | Сбежал | 0.75        | Да        | Да     | 28   | -3   | 29            |

* TEAM\_RESULT: проигрыш, -3 для всех.

**Результаты:**

|    | Итог MMR |
| -- | -------- |
| S1 | -10      |
| S2 | -18      |
| S3 | -10      |
| S4 | 29       |

---

### **В хорошем матче (все сбежали, много Gold):**

|    | OBJ    | CHASE  | ALTR | SURV | Итог   | max\_emblem | multirole | Эскейп | База | Team | Итог (до ММР) |
| -- | ------ | ------ | ---- | ---- | ------ | ----------- | --------- | ------ | ---- | ---- | ------------- |
| S1 | Gold   | Gold   | Gold | Gold | Сбежал | 0.75        | Да        | Да     | 28   | 3    | 35            |
| S2 | Gold   | Gold   | Gold | Gold | Сбежал | 0.75        | Да        | Да     | 28   | 3    | 35            |
| S3 | Silver | Gold   | Gold | Gold | Сбежал | 0.75        | Да        | Да     | 28   | 3    | 35            |
| S4 | Gold   | Silver | Gold | Gold | Сбежал | 0.75        | Да        | Да     | 28   | 3    | 35            |

* TEAM\_RESULT: победа, +3

**Результаты (cap 35):**

|    | Итог MMR |
| -- | -------- |
| S1 | 35       |
| S2 | 35       |
| S3 | 35       |
| S4 | 35       |

---

## **Для киллера аналогично:**

* Если TEAM\_RESULT = -1 (проигрыш), плохие эмблемы — минус, хорошие — маленький плюс или 0.
* Если TEAM\_RESULT = 1 (победа), хорошие эмблемы — +25…35, плохие — +10…+18.
* Коррекция по ММР: если сурвы были в 2 раза слабее — итог ×0.7, если сильнее — ×1.15.

---

### **Пример для киллера (средний матч):**

|    | Gatekeeper | Devout | Malicious | Chaser | Max  | multirole | База | Team | Итог (до ММР) | Итог (MМР) |
| -- | ---------- | ------ | --------- | ------ | ---- | --------- | ---- | ---- | ------------- | ---------- |
| K1 | Silver     | Gold   | Gold      | Gold   | 0.75 | Да        | 28   | 0    | 32            | 32         |

---

### **Плохой для киллера (проиграл слабым):**

|    | Gatekeeper | Devout | Malicious | Chaser | Max | multirole | База | Team | Итог (до ММР) | Итог (MМР) |
| -- | ---------- | ------ | --------- | ------ | --- | --------- | ---- | ---- | ------------- | ---------- |
| K1 | Silver     | Silver | Bronze    | Bronze | 0.5 | Нет       | -7   | -3   | -10           | -10×0.7=-7 |

---

### **Лучший для киллера (убил всех, сурвы были сильнее):**

|    | Gatekeeper | Devout | Malicious | Chaser | Max  | multirole | База | Team | Итог (до ММР) | Итог (MМР)          |
| -- | ---------- | ------ | --------- | ------ | ---- | --------- | ---- | ---- | ------------- | ------------------- |
| K1 | Gold       | Gold   | Gold      | Gold   | 0.75 | Да        | 28   | 3    | 35            | 35×1.15=35 (cap 35) |

---


* В среднем и плохом матче слабые уходят в минус (и умершие, и пассивные).
* Сильные (сбежавшие и мультирольные) получают до +35.
* В хорошем матче плюсы у всех, но у пассивных не будет “жирного” плюса.
* Киллер — по той же логике!
