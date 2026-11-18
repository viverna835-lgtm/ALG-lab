# Лабораторная работа № 7

## Быстрая сортировка, сортировка слиянием, пирамидальная сортировка

**Выполнил:** Утов Алим А.  
**Вариант:** 12 (Продвинутый уровень)

---

## 📋 Содержание

1. [Цель работы](#1-цель-работы)
2. [Задание](#2-задание)
3. [Код программы](#3-код-программы)
4. [Результаты работы программы](#4-результаты-работы-программы)
5. [Анализ сложности операций](#5-анализ-сложности-операций)
6. [Ответы на контрольные вопросы](#6-ответы-на-контрольные-вопросы)
7. [Выводы](#7-выводы)

---

## 1. Цель работы

Реализовать три эффективных алгоритма сортировки: быструю сортировку (Quick Sort), сортировку слиянием (Merge Sort) и пирамидальную сортировку (Heap Sort). Освоить принципы «разделяй и властвуй» на примере быстрой сортировки и сортировки слиянием. Изучить структуру данных «куча» (heap) для реализации пирамидальной сортировки. Проанализировать и сравнить производительность различных алгоритмов сортировки. Понять преимущества и недостатки каждого алгоритма в различных сценариях использования.

---

## 2. Задание

**Вариант 12 (Продвинутый уровень).**

Реализовать все три сортировки с полной функциональностью и дополнительными возможностями. Провести углублённый анализ.

**Дополнительное требование варианта 12:** реализовать **сортировку слиянием для распределённых систем**.

Модель распределённой сортировки слиянием:
- Массив разбивается на K блоков-«узлов».
- Каждый узел независимо сортирует свой блок (Map-этап).
- Корневой узел выполняет K-путевое слияние через min-heap (Reduce-этап).

Дополнительно (для анализа):
- Реализовать QuickSort и HeapSort.
- Сравнить производительность на разных типах данных.
- Сравнить с `std::sort`.

---

## 3. Код программы

Поскольку функции шаблонные (используются с `int`), их реализация находится в заголовочном файле `.h` (стандартное требование C++ для шаблонов).

### Файл `Sorts.h` (Объявление функций)

```cpp
#pragma once
#include <utility>
#include <vector>
#include <queue>
#include <algorithm>

// ==========================================
// СЧЁТЧИКИ ДЛЯ АНАЛИЗА
// ==========================================
struct SortStats {
    long long comparisons = 0;
    long long swaps = 0;
    long long allocations = 0;
};

// ==========================================
// 1. БЫСТРАЯ СОРТИРОВКА (Quick Sort)
// ==========================================
inline int partitionLast(int arr[], int low, int high, SortStats* stats = nullptr) {
    int pivot = arr[high];
    int i = low - 1;
    for (int j = low; j < high; j++) {
        if (stats) stats->comparisons++;
        if (arr[j] <= pivot) {
            i++;
            std::swap(arr[i], arr[j]);
            if (stats) stats->swaps++;
        }
    }
    std::swap(arr[i + 1], arr[high]);
    if (stats) stats->swaps++;
    return i + 1;
}

inline void quickSort(int arr[], int low, int high, SortStats* stats = nullptr) {
    if (low < high) {
        int pi = partitionLast(arr, low, high, stats);
        quickSort(arr, low, pi - 1, stats);
        quickSort(arr, pi + 1, high, stats);
    }
}

// ==========================================
// 2. ПИРАМИДАЛЬНАЯ СОРТИРОВКА (Heap Sort)
// ==========================================
inline void heapify(int arr[], int n, int i, SortStats* stats = nullptr) {
    int largest = i;
    int left = 2 * i + 1;
    int right = 2 * i + 2;

    if (left < n) {
        if (stats) stats->comparisons++;
        if (arr[left] > arr[largest]) largest = left;
    }
    if (right < n) {
        if (stats) stats->comparisons++;
        if (arr[right] > arr[largest]) largest = right;
    }
    if (largest != i) {
        std::swap(arr[i], arr[largest]);
        if (stats) stats->swaps++;
        heapify(arr, n, largest, stats);
    }
}

inline void heapSort(int arr[], int n, SortStats* stats = nullptr) {
    for (int i = n / 2 - 1; i >= 0; i--)
        heapify(arr, n, i, stats);
    for (int i = n - 1; i > 0; i--) {
        std::swap(arr[0], arr[i]);
        if (stats) stats->swaps++;
        heapify(arr, i, 0, stats);
    }
}

// ==========================================
// 3. СОРТИРОВКА СЛИЯНИЕМ (локальная версия)
// ==========================================
inline void mergeArrays(int arr[], int left, int mid, int right, SortStats* stats = nullptr) {
    int n1 = mid - left + 1;
    int n2 = right - mid;
    if (stats) stats->allocations += 2;

    int* L = new int[n1];
    int* R = new int[n2];

    for (int i = 0; i < n1; i++) L[i] = arr[left + i];
    for (int j = 0; j < n2; j++) R[j] = arr[mid + 1 + j];

    int i = 0, j = 0, k = left;
    while (i < n1 && j < n2) {
        if (stats) stats->comparisons++;
        if (L[i] <= R[j]) arr[k++] = L[i++];
        else              arr[k++] = R[j++];
    }
    while (i < n1) arr[k++] = L[i++];
    while (j < n2) arr[k++] = R[j++];

    delete[] L;
    delete[] R;
}

inline void mergeSort(int arr[], int left, int right, SortStats* stats = nullptr) {
    if (left < right) {
        int mid = left + (right - left) / 2;
        mergeSort(arr, left, mid, stats);
        mergeSort(arr, mid + 1, right, stats);
        mergeArrays(arr, left, mid, right, stats);
    }
}

// ==========================================
// 4. РАСПРЕДЕЛЁННАЯ СОРТИРОВКА СЛИЯНИЕМ
//    (вариант 12, продвинутый уровень)
//
//    Модель: массив разбивается на K "узлов".
//    Каждый узел сортирует свой блок независимо (Map),
//    затем корневой узел выполняет K-путевое слияние (Reduce).
// ==========================================
namespace distributed {

    inline void distributedMergeSort(int arr[], int n, int numNodes,
                                     SortStats* stats = nullptr) {
        if (n <= 1 || numNodes <= 1) {
            mergeSort(arr, 0, n - 1, stats);
            return;
        }
        if (numNodes > n) numNodes = n;

        int blockSize = (n + numNodes - 1) / numNodes;
        std::vector<std::vector<int>> nodes;
        nodes.reserve(numNodes);

        // MAP: каждый узел сортирует свой блок
        for (int node = 0; node < numNodes; node++) {
            int start = node * blockSize;
            int end = std::min(start + blockSize, n);
            if (start >= end) break;

            std::vector<int>& local = nodes.emplace_back(arr + start, arr + end);
            if (stats) stats->allocations++;

            if (local.size() > 1) {
                mergeSort(local.data(), 0, (int)local.size() - 1, stats);
            }
        }

        // REDUCE: K-путевое слияние через min-heap
        struct Item {
            int value;
            int nodeIdx;
            int pos;
        };
        struct Cmp {
            bool operator()(const Item& a, const Item& b) const {
                if (a.value != b.value) return a.value > b.value;
                return a.nodeIdx > b.nodeIdx;
            }
        };
        std::priority_queue<Item, std::vector<Item>, Cmp> pq;

        for (int i = 0; i < (int)nodes.size(); i++) {
            if (!nodes[i].empty())
                pq.push({ nodes[i][0], i, 0 });
        }

        int k = 0;
        while (!pq.empty()) {
            Item top = pq.top();
            pq.pop();
            arr[k++] = top.value;
            if (stats) stats->comparisons++;

            int nextPos = top.pos + 1;
            if (nextPos < (int)nodes[top.nodeIdx].size()) {
                pq.push({ nodes[top.nodeIdx][nextPos], top.nodeIdx, nextPos });
            }
        }
    }

} // namespace distributed
```
Файл main.cpp (Демонстрация работы)
```cpp
#include <iostream>
#include <locale>
#include <windows.h>
#include <chrono>
#include <cstdlib>
#include <algorithm>
#include <string>
#include <vector>
#include "Sorts.h"

using namespace std;
using namespace chrono;

int* copyArray(const int src[], int n) {
    int* dst = new int[n];
    for (int i = 0; i < n; i++) dst[i] = src[i];
    return dst;
}

bool isSorted(const int arr[], int n) {
    for (int i = 1; i < n; i++)
        if (arr[i - 1] > arr[i]) return false;
    return true;
}

long long measureQuick(int arr[], int n, SortStats& s) {
    auto t0 = high_resolution_clock::now();
    quickSort(arr, 0, n - 1, &s);
    auto t1 = high_resolution_clock::now();
    return duration_cast<microseconds>(t1 - t0).count();
}
long long measureMerge(int arr[], int n, SortStats& s) {
    auto t0 = high_resolution_clock::now();
    mergeSort(arr, 0, n - 1, &s);
    auto t1 = high_resolution_clock::now();
    return duration_cast<microseconds>(t1 - t0).count();
}
long long measureHeap(int arr[], int n, SortStats& s) {
    auto t0 = high_resolution_clock::now();
    heapSort(arr, n, &s);
    auto t1 = high_resolution_clock::now();
    return duration_cast<microseconds>(t1 - t0).count();
}
long long measureDistributed(int arr[], int n, int nodes, SortStats& s) {
    auto t0 = high_resolution_clock::now();
    distributed::distributedMergeSort(arr, n, nodes, &s);
    auto t1 = high_resolution_clock::now();
    return duration_cast<microseconds>(t1 - t0).count();
}
long long measureStdSort(int arr[], int n) {
    auto t0 = high_resolution_clock::now();
    std::sort(arr, arr + n);
    auto t1 = high_resolution_clock::now();
    return duration_cast<microseconds>(t1 - t0).count();
}

void runAll(const char* label, int* base, int n) {
    cout << "=== " << label << " (N = " << n << ") ===\n";

    SortStats sQ, sM, sH, sD;
    int* a;
    bool ok;

    a = copyArray(base, n);
    long long tQ = measureQuick(a, n, sQ);
    ok = isSorted(a, n); delete[] a;
    cout << "QuickSort:         " << tQ << " мкс, сравн. = " << sQ.comparisons
         << ", обменов = " << sQ.swaps << ", ok = " << (ok ? "да" : "НЕТ") << "\n";

    a = copyArray(base, n);
    long long tM = measureMerge(a, n, sM);
    ok = isSorted(a, n); delete[] a;
    cout << "MergeSort:         " << tM << " мкс, сравн. = " << sM.comparisons
         << ", обменов = " << sM.swaps << ", ok = " << (ok ? "да" : "НЕТ") << "\n";

    a = copyArray(base, n);
    long long tH = measureHeap(a, n, sH);
    ok = isSorted(a, n); delete[] a;
    cout << "HeapSort:          " << tH << " мкс, сравн. = " << sH.comparisons
         << ", обменов = " << sH.swaps << ", ok = " << (ok ? "да" : "НЕТ") << "\n";

    a = copyArray(base, n);
    long long tD = measureDistributed(a, n, 8, sD);
    ok = isSorted(a, n); delete[] a;
    cout << "Distributed Merge: " << tD << " мкс (8 узлов), сравн. = " << sD.comparisons
         << ", аллокаций = " << sD.allocations << ", ok = " << (ok ? "да" : "НЕТ") << "\n";

    a = copyArray(base, n);
    long long tS = measureStdSort(a, n);
    ok = isSorted(a, n); delete[] a;
    cout << "std::sort:         " << tS << " мкс, ok = " << (ok ? "да" : "НЕТ") << "\n";

    cout << "\n";
}

int main() {
    system("chcp 1251 > nul");
    setlocale(LC_ALL, "Russian");

    cout << "=== Лабораторная 7. Продвинутый уровень (Вариант 12) ===\n";
    cout << "=== Распределённая сортировка слиянием ===\n\n";

    srand(42);
    const int N = 200000;

    // Случайный массив
    {
        int* arr = new int[N];
        for (int i = 0; i < N; i++) arr[i] = rand() % N;
        runAll("Случайный массив", arr, N);
        delete[] arr;
    }

    // Массив с дубликатами
    {
        int* arr = new int[N];
        for (int i = 0; i < N; i++) arr[i] = rand() % 100;
        runAll("Массив с дубликатами (100 значений)", arr, N);
        delete[] arr;
    }

    // Обратно отсортированный массив
    {
        int* arr = new int[N];
        for (int i = 0; i < N; i++) arr[i] = N - i;
        runAll("Обратно отсортированный массив", arr, N);
        delete[] arr;
    }

    // Почти отсортированный массив
    {
        int* arr = new int[N];
        for (int i = 0; i < N; i++) arr[i] = i;
        for (int i = 0; i < 200; i++) {
            int a = rand() % N, b = rand() % N;
            std::swap(arr[a], arr[b]);
        }
        runAll("Почти отсортированный массив", arr, N);
        delete[] arr;
    }

    // Влияние количества узлов
    cout << "=== Влияние количества узлов на распределённый MergeSort (N = "
         << N << ", случайные данные) ===\n";
    cout << "Узлов\tВремя (мкс)\tУскорение к 1 узлу\n";

    {
        int* arr = new int[N];
        for (int i = 0; i < N; i++) arr[i] = rand() % N;

        long long baseline = 0;
        for (int nodes : {1, 2, 4, 8, 16, 32}) {
            int* a = copyArray(arr, N);
            SortStats st;
            long long t = measureDistributed(a, N, nodes, st);
            delete[] a;
            if (nodes == 1) baseline = t;
            double speedup = (t > 0) ? (double)baseline / t : 0.0;
            cout << nodes << "\t" << t << "\t\t" << speedup << "x\n";
        }
        delete[] arr;
    }
    cout << "\n";

    return 0;
}
```
## 4. Результаты работы программы
https://2026-11-04_19-02-44.png/

## 5. Анализ сложности операций
Алгоритм	Лучший	Средний	Худший	Память	Устойчивость
QuickSort (pivot = last)	O(n log n)	O(n log n)	O(n²)	O(log n) стек	Нет
MergeSort (локальный)	O(n log n)	O(n log n)	O(n log n)	O(n)	Да
HeapSort	O(n log n)	O(n log n)	O(n log n)	O(1)	Нет
Distributed MergeSort (K узлов)	O(n log n)	O(n log n)	O(n log n)	O(n) на узлах + O(K) на heap	Да
Детали по распределённой версии:

Map-этап: каждый узел сортирует блок размера n/K за O((n/K) log(n/K)). Итого по K узлам: O(n log(n/K)).

Reduce-этап (K-путевое слияние через min-heap): каждое извлечение — O(log K), всего n элементов → O(n log K).

Итоговая сложность: O(n log(n/K) + n log K) = O(n log n).

Практический выигрыш: в реальной распределённой системе узлы работают параллельно, и время Map-этапа становится O((n/K) log(n/K)). Но в нашей эмуляции (один поток) узлы работают последовательно — ускорения относительно локального MergeSort нет, зато наглядно видны накладные расходы («сеть» — копирование в vector + priority_queue).

## 6. Ответы на контрольные вопросы
Какова сложность быстрой сортировки в лучшем, среднем и худшем случае?
Лучший и средний — O(n log n); худший — O(n²).

Когда возникает худший случай для быстрой сортировки?
Когда pivot постоянно оказывается минимумом/максимумом подмассива — например, при выборе последнего элемента на уже отсортированном или обратно отсортированном массиве.

Как можно улучшить быструю сортировку для защиты от худшего случая?
Медиана трёх (first/middle/last), случайный pivot, 3-частное разделение, переключение на InsertionSort для маленьких подмассивов, Introsort (переход на HeapSort при глубокой рекурсии).

Какова сложность сортировки слиянием? Почему она всегда O(n log n)?
O(n log n). Разбиение всегда пополам (не зависит от данных), глубина рекурсии log n, каждый уровень делает O(n) работы на слиянии.

Какие преимущества и недостатки имеет сортировка слиянием перед быстрой?
Преимущества: гарантированная O(n log n), устойчивость. Недостатки: O(n) доп. памяти, чуть медленнее из-за аллокаций и копирования.

Как работает пирамидальная сортировка? Какова её сложность?
Строим max-heap за O(n), затем многократно извлекаем корень (swap с концом + heapify O(log n)). Итого O(n log n) во всех случаях, память O(1).

Какая из трёх сортировок устойчивая? Почему?
MergeSort — устойчива: при равных элементах в слиянии берём левый (L[i] <= R[j]). QuickSort и HeapSort — неустойчивы из-за обменов «через расстояние».

Какая из трёх сортировок требует дополнительной памяти? Сколько?
MergeSort — O(n). QuickSort — O(log n) стек. HeapSort — O(1).

Какая из трёх сортировок лучше работает на почти отсортированных данных?
MergeSort и HeapSort стабильны (O(n log n)). QuickSort с pivot=last деградирует — нужен median-of-three или random pivot.

Зачем в пирамидальной сортировке сначала строится куча?
Чтобы корень массива стал максимумом — дальше извлекаем его за O(1) и восстанавливаем кучу за O(log n). Построение кучи снизу вверх (heapify) занимает O(n).

Что такое «разделяй и властвуй»? Как он применяется в быстрой сортировке и сортировке слиянием?
Деление задачи на подзадачи, рекурсивное решение, объединение. QuickSort: partition вокруг pivot → рекурсия на двух частях (in-place). MergeSort: деление пополам → рекурсия → слияние двух отсортированных половин.

Можно ли реализовать быструю сортировку итеративно (без рекурсии)?
Да, с ручным стеком (low, high). Поведение и сложность те же.

Почему быстрая сортировка обычно быстрее сортировки слиянием на практике?
In-place (лучше кэш), без аллокаций на каждом шаге, меньше копирований.

Когда выгодно использовать пирамидальную сортировку?
При ограниченной памяти (O(1)) и необходимости гарантированной O(n log n) без «сюрпризов».

Какой алгоритм сортировки вы бы выбрали для встроенных систем с ограниченной памятью?
HeapSort — O(1) доп. памяти + гарантированная O(n log n). Альтернатива — Introsort.

## 7. Выводы
В ходе выполнения работы я реализовал три эффективных алгоритма сортировки: QuickSort, MergeSort и HeapSort, а для варианта 12 (продвинутый уровень) дополнительно реализовал распределённую сортировку слиянием для распределённых систем. Распределённая версия использует модель MapReduce: массив разбивается на K блоков-«узлов», каждый узел независимо сортирует свой блок, затем корневой узел выполняет K-путевое слияние через min-heap.

Ключевые выводы:

Теоретическая сложность распределённой версии — O(n log n), но в однопоточной эмуляции она медленнее локального MergeSort из-за накладных расходов на «сетевую пересылку» (копирование в vector) и K-путевое слияние. Это наглядно показывает, что распределённые алгоритмы оправданы только при реальном параллелизме и/или при объёме данных, не помещающемся в RAM одного узла.

На случайных данных QuickSort и MergeSort близки по времени; std::sort стабильно быстрее всех (это оптимизированный Introsort).

HeapSort — единственный с O(1) доп. памяти, но немного медленнее из-за большого числа сравнений в heapify.

MergeSort — единственный устойчивый, что критично для сортировки по нескольким ключам.

Для реальных задач (особенно при работе с большими данными и распределёнными системами) разумно сочетать подходы: распределённый MergeSort для первичной обработки больших объёмов и std::sort / Introsort для финальной сортировки на одном узле.