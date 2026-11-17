# Boardfly 4×8×32

Минимальный стенд: иерархия TPU 8i (4 чипа × 8 блоков × **32 группы** = **1024** чипа) в [BookSim 2.0](https://github.com/booksim/booksim2). Это соответствует «up to 1,024 active chips» из [deep dive](https://cloud.google.com/blog/products/compute/tpu-8t-and-tpu-8i-technical-deep-dive); физическая раскладка Google допускает до 36 групп / 1152 позиций.

![Иерархия Boardfly: K4 на плате, K8 в группе, группы через OCS](images/boardfly-hierarchy.png)

Слева — группа: 8 плат с полносвязными 4 TPU на каждой, платы тоже полносвязны. Справа — pod из групп, связанных через OCS.

Порты платы по официальному deep dive: **8 copper + 4 OCS**. В графе: K8 даёт 7 медных рёбер, восьмой copper-порт — parallel matching; K32 даёт 31 оптический линк на группу при бюджете \(8 \times 4 = 32\) — укладка «плата `q%8`, чип `q//8`» даёт **≤4 OCS на плату и ровно один на чип**.

```bash
sudo apt-get install -y build-essential flex bison python3
git submodule update --init --recursive
make
```

`scripts/boardfly.py` строит граф (K4 / K8+matching / K32), проверяет порт-бюджет и диаметр, гоняет uniform-трафик.

Диаметр **7**: худший путь между чипами разных групп — три hops в группе-источнике, один оптический hop через OCS и три hops в группе назначения.

![Диаметр Boardfly: путь из 7 hops](images/boardfly-diameter.png)

Ожидаемый вывод:

```
n=1024 deg=4-7 mean=5.969 diam=7 asp=5.079 links=3056
ports copper=8-8 ocs=3-4 (mean 3.875) chip_ocs≤1
inj=0.02 latency=26.299 hops=6.062 accepted=0.020
```
