# Boardfly 4×8×36

Минимальный стенд: иерархия TPU 8i (4 чипа × 8 блоков × 36 групп = 1152 позиций) в [BookSim 2.0](https://github.com/booksim/booksim2). Google отдельно указывает до 1024 active chips на тех же 36 группах.

![Иерархия Boardfly: K4 на плате, K8 в группе, K36 в pod](images/boardfly-hierarchy.png)

Слева — группа: 8 плат с полносвязными 4 TPU на каждой, платы тоже полносвязны. Справа — 36 таких групп в pod.

Порты платы по [официальному deep dive](https://cloud.google.com/blog/products/compute/tpu-8t-and-tpu-8i-technical-deep-dive): **8 copper + 4 OCS**. В графе: K8 даёт 7 медных рёбер, восьмой copper-порт — parallel matching; 35 оптических линков K36 распределены по 4–5 на плату (номинальные 4, без плат с тремя портами).

```bash
sudo apt-get install -y build-essential flex bison python3
git submodule update --init --recursive
make
```

`scripts/boardfly.py` строит граф (K4 / K8+matching / K36), проверяет порт-бюджет и диаметр, гоняет uniform-трафик.

Диаметр **7**: худший путь между чипами разных групп — три hops в группе-источнике, один оптический hop через OCS и три hops в группе назначения.

![Диаметр Boardfly: путь из 7 hops](images/boardfly-diameter.png)

Ожидаемый вывод:

```
n=1152 deg=5-8 mean=6.094 diam=7 asp=4.985 links=3510
ports copper=8-8 ocs=4-5 (mean 4.375)
inj=0.02 latency=25.970 hops=5.979 accepted=0.020
```
