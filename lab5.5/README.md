# Лабораторная работа № 5

## Реализация структур данных: стек, очередь, дек, приоритетная очередь

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

Реализовать основные линейные структуры данных: стек, очередь, дек и приоритетную очередь. Освоить принципы LIFO (стек) и FIFO (очередь). Научиться реализовывать эти структуры на основе динамического массива и связного списка. Изучить применение стека для решения задач (проверка скобок, обратная польская запись). Реализовать приоритетную очередь на основе кучи (heap).

---

## 2. Задание

**Вариант 12 (Продвинутый уровень).**

Разработать четыре линейные структуры данных:

- **Стек** — с поддержкой операции `getMiddle()` (середина стека).
- **Очередь** — с разделением на две: чётные и нечётные позиции.
- **Дек** — с операцией `insertAt(index)`.
- **Приоритетная очередь** — с проверкой, является ли массив двоичной кучей.

Дополнительно:

- Стек — динамический массив с амортизированным `push` O(1) и `getMiddle` O(1).
- Очередь — кольцевой буфер с автоматическим расширением.
- Дек — двусвязный список с произвольной вставкой.
- Приоритетная очередь — max-heap с операциями `push`/`pop` за O(log n) и `top` за O(1).
- Обработка ошибок — исключения `std::underflow_error` / `std::out_of_range`.

---

## 3. Код программы

Поскольку классы шаблонные, их реализация находится полностью в заголовочном файле `.h` (стандартное требование C++ для шаблонов).

### Файл `Structures.h`

```cpp
#pragma once
#include <iostream>
#include <stdexcept>
#include <utility>

using namespace std;

// ==========================================
// 1. СТЕК (Stack) на динамическом массиве
//    + операция getMiddle() — вариант 12
// ==========================================
template <typename T>
class Stack {
private:
    T* data;
    int topIndex;
    int capacity;

    void resize() {
        int newCapacity = capacity * 2;
        T* newData = new T[newCapacity];
        for (int i = 0; i <= topIndex; i++)
            newData[i] = data[i];
        delete[] data;
        data = newData;
        capacity = newCapacity;
    }
public:
    Stack(int initialCapacity = 10)
        : capacity(initialCapacity), topIndex(-1) {
        data = new T[capacity];
    }
    ~Stack() { delete[] data; }

    Stack(const Stack&) = delete;
    Stack& operator=(const Stack&) = delete;

    void push(const T& value) {
        if (topIndex >= capacity - 1) resize();
        data[++topIndex] = value;
    }

    T pop() {
        if (isEmpty()) throw std::underflow_error("Stack is empty");
        return data[topIndex--];
    }

    T top() const {
        if (isEmpty()) throw std::underflow_error("Stack is empty");
        return data[topIndex];
    }

    // ВАРИАНТ 12: getMiddle()
    T getMiddle() const {
        if (isEmpty()) throw std::underflow_error("Stack is empty");
        int idx = (topIndex + 1) / 2;
        return data[idx];
    }

    bool isEmpty() const { return topIndex == -1; }
    int size() const { return topIndex + 1; }

    void print() const {
        cout << "Stack (top->bottom): [";
        for (int i = topIndex; i >= 0; i--) {
            cout << data[i];
            if (i > 0) cout << ", ";
        }
        cout << "]" << endl;
    }
};

// ==========================================
// 2. ОЧЕРЕДЬ (Queue) на кольцевом буфере
//    + разделение на чётные/нечётные позиции — вариант 12
// ==========================================
template <typename T>
class Queue {
private:
    T* data;
    int frontIndex;
    int rearIndex;
    int count;
    int capacity;

    void resize() {
        int newCapacity = capacity * 2;
        T* newData = new T[newCapacity];
        for (int i = 0; i < count; i++)
            newData[i] = data[(frontIndex + i) % capacity];
        delete[] data;
        data = newData;
        frontIndex = 0;
        rearIndex = count;
        capacity = newCapacity;
    }
public:
    Queue(int initialCapacity = 10)
        : capacity(initialCapacity), frontIndex(0), rearIndex(0), count(0) {
        data = new T[capacity];
    }
    ~Queue() { delete[] data; }

    Queue(const Queue&) = delete;
    Queue& operator=(const Queue&) = delete;

    void enqueue(const T& value) {
        if (count >= capacity) resize();
        data[rearIndex] = value;
        rearIndex = (rearIndex + 1) % capacity;
        count++;
    }

    T dequeue() {
        if (isEmpty()) throw std::underflow_error("Queue is empty");
        T value = data[frontIndex];
        frontIndex = (frontIndex + 1) % capacity;
        count--;
        return value;
    }

    T front() const {
        if (isEmpty()) throw std::underflow_error("Queue is empty");
        return data[frontIndex];
    }

    bool isEmpty() const { return count == 0; }
    int size() const { return count; }

    void print() const {
        cout << "Queue (front->back): [";
        for (int i = 0; i < count; i++) {
            cout << data[(frontIndex + i) % capacity];
            if (i < count - 1) cout << ", ";
        }
        cout << "]" << endl;
    }

    // ВАРИАНТ 12
    void splitByPosition(Queue<T>& evenPositions, Queue<T>& oddPositions) const {
        for (int i = 0; i < count; i++) {
            T value = data[(frontIndex + i) % capacity];
            if ((i + 1) % 2 == 0) evenPositions.enqueue(value);
            else                  oddPositions.enqueue(value);
        }
    }
};

// ==========================================
// 3. ДЕК (Deque) на двусвязном списке
//    + операция insertAt(index) — вариант 12
// ==========================================
template <typename T>
class Deque {
private:
    struct Node {
        T data;
        Node* prev;
        Node* next;
        Node(const T& value) : data(value), prev(nullptr), next(nullptr) {}
    };
    Node* head;
    Node* tail;
    int count;
public:
    Deque() : head(nullptr), tail(nullptr), count(0) {}
    ~Deque() { clear(); }

    Deque(const Deque&) = delete;
    Deque& operator=(const Deque&) = delete;

    void push_front(const T& value) {
        Node* newNode = new Node(value);
        if (head == nullptr) {
            head = tail = newNode;
        } else {
            newNode->next = head;
            head->prev = newNode;
            head = newNode;
        }
        count++;
    }

    void push_back(const T& value) {
        Node* newNode = new Node(value);
        if (tail == nullptr) {
            head = tail = newNode;
        } else {
            tail->next = newNode;
            newNode->prev = tail;
            tail = newNode;
        }
        count++;
    }

    T pop_front() {
        if (isEmpty()) throw std::underflow_error("Deque is empty");
        Node* temp = head;
        T value = temp->data;
        head = head->next;
        if (head != nullptr) head->prev = nullptr;
        else tail = nullptr;
        delete temp;
        count--;
        return value;
    }

    T pop_back() {
        if (isEmpty()) throw std::underflow_error("Deque is empty");
        Node* temp = tail;
        T value = temp->data;
        tail = tail->prev;
        if (tail != nullptr) tail->next = nullptr;
        else head = nullptr;
        delete temp;
        count--;
        return value;
    }

    T front() const {
        if (isEmpty()) throw std::underflow_error("Deque is empty");
        return head->data;
    }
    T back() const {
        if (isEmpty()) throw std::underflow_error("Deque is empty");
        return tail->data;
    }

    // ВАРИАНТ 12
    void insertAt(int index, const T& value) {
        if (index < 0 || index > count)
            throw std::out_of_range("insertAt: index out of range");
        if (index == 0)     { push_front(value); return; }
        if (index == count) { push_back(value);  return; }
        Node* cur = head;
        for (int i = 0; i < index; i++) cur = cur->next;
        Node* newNode = new Node(value);
        newNode->prev = cur->prev;
        newNode->next = cur;
        cur->prev->next = newNode;
        cur->prev = newNode;
        count++;
    }

    bool isEmpty() const { return count == 0; }
    int size() const { return count; }

    void print() const {
        cout << "Deque (front->back): [";
        Node* cur = head;
        while (cur != nullptr) {
            cout << cur->data;
            if (cur->next != nullptr) cout << ", ";
            cur = cur->next;
        }
        cout << "]" << endl;
    }

    void clear() {
        while (head != nullptr) {
            Node* temp = head;
            head = head->next;
            delete temp;
        }
        tail = nullptr;
        count = 0;
    }
};

// ==========================================
// 4. ПРИОРИТЕТНАЯ ОЧЕРЕДЬ (max-heap)
//    + проверка, является ли массив двоичной кучей — вариант 12
// ==========================================
template <typename T>
class PriorityQueue {
private:
    T* heap;
    int count;
    int capacity;

    void resize() {
        int newCapacity = capacity * 2;
        T* newHeap = new T[newCapacity];
        for (int i = 0; i < count; i++) newHeap[i] = heap[i];
        delete[] heap;
        heap = newHeap;
        capacity = newCapacity;
    }

    void heapifyUp(int index) {
        while (index > 0) {
            int parent = (index - 1) / 2;
            if (heap[index] > heap[parent]) {
                std::swap(heap[index], heap[parent]);
                index = parent;
            } else break;
        }
    }

    void heapifyDown(int index) {
        while (true) {
            int left = 2 * index + 1;
            int right = 2 * index + 2;
            int largest = index;
            if (left < count && heap[left] > heap[largest]) largest = left;
            if (right < count && heap[right] > heap[largest]) largest = right;
            if (largest == index) break;
            std::swap(heap[index], heap[largest]);
            index = largest;
        }
    }
public:
    PriorityQueue(int initialCapacity = 10)
        : capacity(initialCapacity), count(0) {
        heap = new T[capacity];
    }
    ~PriorityQueue() { delete[] heap; }

    PriorityQueue(const PriorityQueue&) = delete;
    PriorityQueue& operator=(const PriorityQueue&) = delete;

    void push(const T& value) {
        if (count >= capacity) resize();
        heap[count] = value;
        heapifyUp(count);
        count++;
    }

    T pop() {
        if (isEmpty()) throw std::underflow_error("PriorityQueue is empty");
        T maxValue = heap[0];
        heap[0] = heap[count - 1];
        count--;
        heapifyDown(0);
        return maxValue;
    }

    T top() const {
        if (isEmpty()) throw std::underflow_error("PriorityQueue is empty");
        return heap[0];
    }

    bool isEmpty() const { return count == 0; }
    int size() const { return count; }

    // ВАРИАНТ 12
    static bool isBinaryHeap(const T* arr, int n) {
        if (n <= 1) return true;
        for (int i = 0; i <= (n - 2) / 2; i++) {
            int left = 2 * i + 1;
            int right = 2 * i + 2;
            if (left < n && arr[left] > arr[i]) return false;
            if (right < n && arr[right] > arr[i]) return false;
        }
        return true;
    }

    void print() const {
        cout << "PriorityQueue (heap array): [";
        for (int i = 0; i < count; i++) {
            cout << heap[i];
            if (i < count - 1) cout << ", ";
        }
        cout << "]" << endl;
    }
 };
 ```

### Файл main.cpp.

 ```cpp
 #include <iostream>
 #include <locale>
  #include <windows.h>
 #include "Structures.h"

 using namespace std;

 int main() {
    system("chcp 1251 > nul");
    setlocale(LC_ALL, "Russian");

    cout << "=== Лабораторная 5. Продвинутый уровень (Вариант 12) ===\n";
    cout << "=== Стек, очередь, дек, приоритетная очередь ===\n\n";

    // 1. СТЕК + getMiddle()
    cout << "--- 1. Стек с getMiddle() ---\n";
    Stack<int> st;
    for (int i = 10; i <= 50; i += 10) st.push(i);
    st.print();
    cout << "size = " << st.size() << "\n";
    cout << "top = " << st.top() << "\n";
    cout << "getMiddle() = " << st.getMiddle() << "  (ожидается 30)\n";

    st.pop();
    st.print();
    cout << "getMiddle() = " << st.getMiddle() << "  (ожидается 30)\n";

    st.pop();
    st.print();
    cout << "getMiddle() = " << st.getMiddle() << "  (ожидается 20)\n";

    try {
        Stack<int> emptySt;
        emptySt.pop();
    } catch (const std::exception& e) {
        cout << "Ошибка при pop из пустого стека: " << e.what() << "\n";
    }
    cout << "\n";

    // 2. ОЧЕРЕДЬ + разделение по позициям
    cout << "--- 2. Очередь: разделение на чётные/нечётные позиции ---\n";
    Queue<int> q;
    for (int i = 1; i <= 7; i++) q.enqueue(i * 10);
    q.print();

    Queue<int> evenQ, oddQ;
    q.splitByPosition(evenQ, oddQ);
    cout << "Очередь с чётными позициями:  "; evenQ.print();
    cout << "Очередь с нечётными позициями: "; oddQ.print();

    try {
        Queue<int> emptyQ;
        emptyQ.dequeue();
    } catch (const std::exception& e) {
        cout << "Ошибка при dequeue из пустой очереди: " << e.what() << "\n";
    }
    cout << "\n";

    // 3. ДЕК + insertAt(index)
    cout << "--- 3. Дек с insertAt(index) ---\n";
    Deque<int> dq;
    dq.push_back(10);
    dq.push_back(20);
    dq.push_back(40);
    dq.print();

    dq.insertAt(2, 30);
    cout << "После insertAt(2, 30): "; dq.print();

    dq.insertAt(0, 5);
    cout << "После insertAt(0, 5):  "; dq.print();

    dq.insertAt(dq.size(), 50);
    cout << "После insertAt(size, 50): "; dq.print();

    try {
        dq.insertAt(100, 999);
    } catch (const std::exception& e) {
        cout << "Ошибка insertAt: " << e.what() << "\n";
    }
    cout << "\n";

    // 4. ПРИОРИТЕТНАЯ ОЧЕРЕДЬ + isBinaryHeap
    cout << "--- 4. Приоритетная очередь: проверка двоичной кучи ---\n";
    int heapArr[] = {50, 30, 40, 10, 20};
    int n1 = sizeof(heapArr) / sizeof(heapArr[0]);
    cout << "Массив {50, 30, 40, 10, 20} — это max-куча? "
         << (PriorityQueue<int>::isBinaryHeap(heapArr, n1) ? "ДА" : "НЕТ") << "\n";

    int notHeapArr[] = {10, 50, 20};
    int n2 = sizeof(notHeapArr) / sizeof(notHeapArr[0]);
    cout << "Массив {10, 50, 20} — это max-куча? "
         << (PriorityQueue<int>::isBinaryHeap(notHeapArr, n2) ? "ДА" : "НЕТ") << "\n";

    PriorityQueue<int> pq;
    pq.push(30);
    pq.push(10);
    pq.push(50);
    pq.push(20);
    pq.push(40);
    cout << "Приоритетная очередь: "; pq.print();
    cout << "Извлечение по убыванию: ";
    while (!pq.isEmpty()) cout << pq.pop() << " ";
    cout << "\n";

    try {
        pq.pop();
    } catch (const std::exception& e) {
        cout << "Ошибка при pop из пустой приоритетной очереди: " << e.what() << "\n";
    }

    return 0;
}
```
## 4. Результаты работы программы
https://2026-11-04_19-02-44.png/

## 5. Анализ сложности операций
Операция	Сложность	Обоснование
push (стек)	O(1) амортизированная	В среднем добавление в конец массива занимает константное время. При переполнении ёмкость удваивается и происходит копирование (O(n)), но это происходит редко.
pop (стек)	O(1)	Удаление верхнего элемента не требует сдвига остальных.
getMiddle (стек)	O(1)	Индекс (topIndex + 1) / 2 вычисляется за константу, доступ по индексу — O(1).
enqueue (очередь)	O(1) амортизированная	Запись в rear и циклический сдвиг индекса; resize — редко.
dequeue (очередь)	O(1)	Сдвиг frontIndex по кругу.
splitByPosition	O(n)	Один проход по всем элементам очереди.
push_front / push_back (дек)	O(1)	Перестановка указателей head/tail.
pop_front / pop_back (дек)	O(1)	Перестановка указателей.
insertAt(index) (дек)	O(index)	Обход списка от head до нужной позиции.
push (приоритетная очередь)	O(log n)	heapifyUp поднимается по дереву высотой log n.
pop (приоритетная очередь)	O(log n)	heapifyDown опускается по дереву высотой log n.
top (приоритетная очередь)	O(1)	Максимум всегда в корне кучи.
isBinaryHeap	O(n)	Проверка всех родителей (n/2 − 1).
## 6. Ответы на контрольные вопросы
В чём заключается принцип работы стека (LIFO)?
LIFO (Last In — First Out): последний добавленный элемент извлекается первым. Работает как стопка тарелок: push кладёт сверху, pop снимает сверху.

В чём заключается принцип работы очереди (FIFO)?
FIFO (First In — First Out): первый добавленный элемент извлекается первым. Аналогия — очередь в магазине.

Чем дек отличается от обычной очереди?
Дек (double-ended queue) позволяет добавлять и удалять элементы с обоих концов (push_front, push_back, pop_front, pop_back), тогда как очередь — только с одного конца добавлять, с другого удалять.

Что такое приоритетная очередь? Где она применяется?
Структура, где каждый элемент имеет приоритет; извлекается элемент с наибольшим (или наименьшим) приоритетом. Применяется в планировщиках задач, алгоритмах Дейкстры и Прима, обработке событий, сжатии Хаффмана.

Как реализовать стек на основе динамического массива?
Хранить массив data, счётчик topIndex (индекс вершины). push увеличивает topIndex и записывает значение; при переполнении — удвоение ёмкости. pop возвращает data[topIndex--].

Как реализовать очередь на основе кольцевого буфера?
Хранить массив, индексы front и rear, счётчик count. При enqueue записываем в data[rear], затем rear = (rear+1) % capacity. При dequeue берём data[front], затем front = (front+1) % capacity. Это даёт O(1) для обеих операций без сдвига элементов.

Почему приоритетную очередь обычно реализуют на куче?
Куча даёт O(log n) на push/pop и O(1) на top. Альтернативы (отсортированный массив — O(n) на вставку; несортированный — O(n) на извлечение) хуже.

Какова сложность операций push и pop в стеке?
push — O(1) амортизированно (O(n) при редком resize); pop — O(1).

Какова сложность операций enqueue и dequeue в очереди?
Обе — O(1) амортизированно (при кольцевом буфере resize происходит редко).

Какова сложность операций в приоритетной очереди? Почему?
push и pop — O(log n), потому что после вставки/удаления нужно восстановить свойство кучи, «поднимаясь» или «опускаясь» по дереву высотой log n. top — O(1), так как максимум всегда в корне.

Как проверить правильность расстановки скобок с помощью стека?
Идём по строке: открывающую скобку — push; закрывающую — сравниваем с top(): если совпадает — pop, иначе — ошибка. В конце стек должен быть пуст.

Как реализовать очередь на двух стеках? Какова сложность?
enqueue — push в первый стек (O(1)). dequeue — если второй стек пуст, перекладываем в него все элементы из первого (порядок переворачивается), затем pop из второго. Амортизированная сложность dequeue — O(1), худшая — O(n).

Как реализовать дек на основе двусвязного списка?
Каждый узел хранит prev, next, data. Хранятся указатели head и tail. push_front/push_back вставляют узел за O(1), pop_front/pop_back удаляют крайний узел за O(1), front/back возвращают head->data/tail->data.

Как реализовать дек на основе кольцевого буфера?
Массив фиксированной/динамической ёмкости, индексы front и back, счётчик. push_front — front = (front-1+cap) % cap; push_back — back = (back+1) % cap; pop_front — front = (front+1) % cap; pop_back — back = (back-1+cap) % cap.

Как реализовать стек с поддержкой получения минимума за O(1)?
Хранить два стека: основной и вспомогательный minStack. При push(x) в основной кладём x, во вспомогательный — min(x, minStack.top()). При pop удаляем из обоих. getMin() = minStack.top() — O(1).

## 7. Выводы
В ходе выполнения работы я реализовал четыре линейные структуры данных: стек (на динамическом массиве), очередь (на кольцевом буфере), дек (на двусвязном списке) и приоритетную очередь (на двоичной max-куче).

Для варианта 12 реализовал обязательные операции и дополнительную функциональность: getMiddle() в стеке за O(1), разделение очереди по чётным/нечётным позициям за O(n), insertAt(index) в деке за O(index) и проверку массива на свойство двоичной кучи за O(n).

Освоил принципы LIFO и FIFO, научился обрабатывать ошибки при операциях с пустыми структурами (исключения std::underflow_error / std::out_of_range), а также проанализировал сложность всех операций. Приоритетная очередь на куче обеспечивает O(log n) на push/pop и O(1) на top, что делает её эффективной для задач планирования и обработки событий.