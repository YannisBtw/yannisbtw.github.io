# Методы оптимизации вычисления кода с помощью потоков, процессов, Cython, отпускания GIL

## Цель
Исследовать методы оптимизации вычислительного кода, используя потоки, процессы, Cython и отпускание GIL, на основе сравнения времени вычисления численного интеграла методом прямоугольников.

## Итерация 1 - чистый Python
- Функция `integrate()` оформлена docstring (PEP 257) и аннотациями типов (PEP 484) - файл `integrate_base.py`.
- В docstring встроены примеры `doctest`.
- Добавлены unit-тесты (pytest и unittest) - каталог `tests/`.
- Замеры времени: `iteration1_pure_python.py` и общий прогон `bench_all.py`.

## Итерация 2 - потоки
- Реализована функция `integrate_async()` (ThreadPoolExecutor) - файл `iteration2_threads.py`.
- Замеры для 2, 4, 6, 8 потоков.

Вывод: для CPU-bound кода на чистом Python потоки почти не ускоряют вычисления из-за GIL.

## Итерация 3 - процессы
- Реализована функция `integrate_async_processes()` (ProcessPoolExecutor) - файл `iteration3_processes.py`.
- Замеры для 2, 4, 6, 8 процессов.

Вывод: процессы дают реальный параллелизм и ускорение, но имеют накладные расходы.

## Итерация 4 - профилирование и Cython
### Профилирование
Файл `profile_integrate.py` строит профиль выполнения и сохраняет `profile.pstats`.

### Cython-оптимизация
Модуль `cython_mod/integrate_cy.pyx` содержит:
- `integrate_sin`, `integrate_cos`, `integrate_poly2` - C-level варианты (минимум взаимодействия с Python/C-API)
- `integrate_pycall` - цикл ускорен, но вызов `f(x)` остается Python-вызовом в каждой итерации.

### annotate=True
Сборка с HTML-аннотацией:
```bash
CYTHON_ANNOTATE=1 python setup.py build_ext --inplace
```

## Итерация 5 - noGIL
Модуль `cython_mod/integrate_nogil.pyx`:
- `integrate_sin_nogil` - отпускание GIL и распараллеливание через `prange` (OpenMP).
- Замеры для 2, 4, 6 потоков и сравнение с процессами + чистый Python.

## Доп. вопрос: мьютексы/семафоры
Обычно не нужны:
- каждая задача считает локальную сумму
- итоговая сумма складывается после завершения задач
- блокировки внутри цикла ухудшают производительность

## Доп. вопрос: Python 3.14 и GIL
В стандартном CPython по умолчанию используется GIL, поэтому вывод про потоки (итерация 2) сохраняется.
Если используется экспериментальная сборка/режим без GIL, потоки могут начать ускорять CPU-bound код - тогда замеры нужно повторить.




## Итерации 1-3
```bash
python iteration1_pure_python.py
python iteration2_threads.py
python iteration3_processes.py
```

## Doctest
```bash
python -m doctest -v integrate_base.py
```

## Тесты
```bash
pip install -r requirements.txt
pytest -q
# или
python -m unittest tests/test_integrate_unittest.py -v
```

## Профилирование
```bash
python profile_integrate.py
```

## Cython (итерации 4-5)
Установка:
```bash
pip install -r requirements.txt
```

Сборка:
```bash
python setup.py build_ext --inplace
```

Сборка с HTML-аннотацией:
```bash
CYTHON_ANNOTATE=1 python setup.py build_ext --inplace
```

Замеры:
```bash
python iteration4_cython_bench.py
python iteration5_nogil_bench.py
```

## Общий прогон замеров и таблица
```bash
python bench_all.py
```

# Результаты замеров

n_iter = 2000000

|   iteration | method       | detail              |   n_jobs |   n_iter |        sec |
|------------:|:-------------|:--------------------|---------:|---------:|-----------:|
|           1 | python       | baseline sin        |        1 |  2000000 | 0.0938391  |
|           2 | threads      | ThreadPoolExecutor  |        2 |  2000000 | 0.0951086  |
|           2 | threads      | ThreadPoolExecutor  |        4 |  2000000 | 0.0942634  |
|           2 | threads      | ThreadPoolExecutor  |        6 |  2000000 | 0.0950183  |
|           2 | threads      | ThreadPoolExecutor  |        8 |  2000000 | 0.0947595  |
|           3 | processes    | ProcessPoolExecutor |        2 |  2000000 | 0.403144   |
|           3 | processes    | ProcessPoolExecutor |        4 |  2000000 | 0.433724   |
|           3 | processes    | ProcessPoolExecutor |        6 |  2000000 | 0.468149   |
|           3 | processes    | ProcessPoolExecutor |        8 |  2000000 | 0.544832   |
|           4 | cython       | C-level sin         |        1 |  2000000 | 0.00847953 |
|           4 | cython       | loop + py-call sin  |        1 |  2000000 | 0.0535731  |
|           5 | cython_nogil | OpenMP prange sin   |        2 |  2000000 | 0.0083781  |
|           5 | cython_nogil | OpenMP prange sin   |        4 |  2000000 | 0.007756   |
|           5 | cython_nogil | OpenMP prange sin   |        6 |  2000000 | 0.00487367 |

