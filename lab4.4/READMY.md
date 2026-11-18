# Лабораторная работа № 4

## Односвязный и двусвязный списки

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

Реализовать шаблонный класс односвязного списка (`SinglyLinkedList`) с базовыми операциями. Реализовать шаблонный класс двусвязного списка (`DoublyLinkedList`) с базовыми операциями. Освоить операции вставки, удаления, поиска и обхода элементов в списках. Сравнить производительность и особенности использования односвязного и двусвязного списков. Научиться правильно управлять памятью при работе со списками (удаление узлов).

---

## 2. Задание

**Вариант 12 (Продвинутый уровень).**

Разработать классы связных списков, содержащие:

- **Односвязный список (SinglyLinkedList):** базовые операции `push_front`, `push_back`, `pop_front`, `pop_back`, `find`, `remove`, `print`, `clear`.
- **Двусвязный список (DoublyLinkedList):**
  - **Обязательные операции варианта 12:** `push_back`, `pop_back`, `empty`.
  - **Дополнительная функциональность варианта 12:** скопировать список в другой.
  - **Продвинутый уровень:** итераторы (`begin`, `end`, `operator*`, `operator++`, `operator--`, `operator==`, `operator!=`) и метод `reverse()` — разворот списка за O(n) без выделения дополнительной памяти.

Управление памятью: деструкторы, `clear()`, глубокое копирование (без утечек и двойного `delete`).

---

## 3. Код программы

Поскольку классы шаблонные, их реализация находится полностью в заголовочном файле `.h` (стандартное требование C++ для шаблонов).

### Файл `Lists.h`

```cpp
#pragma once
#include <iostream>
#include <stdexcept>
#include <utility>

using namespace std;

// ==========================================
// УЗЕЛ ОДНОСВЯЗНОГО СПИСКА
// ==========================================
template <typename T>
struct Node {
    T data;
    Node<T>* next;
    Node(const T& value) : data(value), next(nullptr) {}
};

// ==========================================
// ОДНОСВЯЗНЫЙ СПИСОК (SinglyLinkedList)
// ==========================================
template <typename T>
class SinglyLinkedList {
private:
    Node<T>* head;
    Node<T>* tail;
    int count;
public:
    SinglyLinkedList() : head(nullptr), tail(nullptr), count(0) {}
    ~SinglyLinkedList() { clear(); }

    // Запрет копирования (защита от двойного delete)
    SinglyLinkedList(const SinglyLinkedList&) = delete;
    SinglyLinkedList& operator=(const SinglyLinkedList&) = delete;

    // --- Вставка ---
    void push_front(const T& value) {
        Node<T>* newNode = new Node<T>(value);
        newNode->next = head;
        head = newNode;
        if (tail == nullptr) tail = newNode;
        count++;
    }

    void push_back(const T& value) {
        Node<T>* newNode = new Node<T>(value);
        if (head == nullptr) {
            head = tail = newNode;
        } else {
            tail->next = newNode;
            tail = newNode;
        }
        count++;
    }

    // --- Удаление ---
    void pop_front() {
        if (head == nullptr) throw std::underflow_error("List is empty");
        Node<T>* temp = head;
        head = head->next;
        delete temp;
        count--;
        if (head == nullptr) tail = nullptr;
    }

    void pop_back() {
        if (head == nullptr) throw std::underflow_error("List is empty");
        if (head->next == nullptr) {
            delete head;
            head = tail = nullptr;
        } else {
            Node<T>* current = head;
            while (current->next != tail) current = current->next;
            delete tail;
            tail = current;
            tail->next = nullptr;
        }
        count--;
    }

    // --- Поиск ---
    Node<T>* find(const T& value) const {
        Node<T>* current = head;
        while (current != nullptr) {
            if (current->data == value) return current;
            current = current->next;
        }
        return nullptr;
    }

    // --- Удаление по значению ---
    bool remove(const T& value) {
        if (head == nullptr) return false;
        if (head->data == value) { pop_front(); return true; }
        Node<T>* current = head;
        while (current->next != nullptr && current->next->data != value)
            current = current->next;
        if (current->next == nullptr) return false;
        Node<T>* toDelete = current->next;
        current->next = toDelete->next;
        if (toDelete == tail) tail = current;
        delete toDelete;
        count--;
        return true;
    }

    // --- Вывод ---
    void print() const {
        cout << "[";
        Node<T>* current = head;
        while (current != nullptr) {
            cout << current->data;
            if (current->next != nullptr) cout << ", ";
            current = current->next;
        }
        cout << "]" << endl;
    }

    // --- Очистка ---
    void clear() {
        while (head != nullptr) {
            Node<T>* temp = head;
            head = head->next;
            delete temp;
        }
        tail = nullptr;
        count = 0;
    }

    int size() const { return count; }
    bool empty() const { return head == nullptr; }
};

// ==========================================
// УЗЕЛ ДВУСВЯЗНОГО СПИСКА
// ==========================================
template <typename T>
struct DNode {
    T data;
    DNode<T>* prev;
    DNode<T>* next;
    DNode(const T& value) : data(value), prev(nullptr), next(nullptr) {}
};

// ==========================================
// ДВУСВЯЗНЫЙ СПИСОК (DoublyLinkedList)
// ==========================================
template <typename T>
class DoublyLinkedList {
private:
    DNode<T>* head;
    DNode<T>* tail;
    int count;
public:
    // ---------------- Итератор (продвинутый уровень) ----------------
    class Iterator {
    private:
        DNode<T>* ptr;
    public:
        Iterator(DNode<T>* p = nullptr) : ptr(p) {}

        T& operator*()  { return ptr->data; }
        T* operator->() { return &ptr->data; }

        Iterator& operator++() { ptr = ptr->next; return *this; }
        Iterator  operator++(int) { Iterator tmp = *this; ptr = ptr->next; return tmp; }

        Iterator& operator--() { ptr = ptr->prev; return *this; }
        Iterator  operator--(int) { Iterator tmp = *this; ptr = ptr->prev; return tmp; }

        bool operator==(const Iterator& other) const { return ptr == other.ptr; }
        bool operator!=(const Iterator& other) const { return ptr != other.ptr; }
    };

    DoublyLinkedList() : head(nullptr), tail(nullptr), count(0) {}

    // Конструктор копирования (глубокое копирование) — вариант 12
    DoublyLinkedList(const DoublyLinkedList& other)
        : head(nullptr), tail(nullptr), count(0) {
        for (DNode<T>* cur = other.head; cur != nullptr; cur = cur->next)
            push_back(cur->data);
    }

    // Оператор присваивания (copy-and-swap)
    DoublyLinkedList& operator=(const DoublyLinkedList& other) {
        if (this != &other) {
            DoublyLinkedList tmp(other);
            swap(tmp);
        }
        return *this;
    }

    ~DoublyLinkedList() { clear(); }

    void swap(DoublyLinkedList& other) noexcept {
        std::swap(head,  other.head);
        std::swap(tail,  other.tail);
        std::swap(count, other.count);
    }

    // --- Вставка ---
    void push_front(const T& value) {
        DNode<T>* newNode = new DNode<T>(value);
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
        DNode<T>* newNode = new DNode<T>(value);
        if (tail == nullptr) {
            head = tail = newNode;
        } else {
            tail->next = newNode;
            newNode->prev = tail;
            tail = newNode;
        }
        count++;
    }

    // --- Удаление ---
    void pop_front() {
        if (head == nullptr) throw std::underflow_error("List is empty");
        DNode<T>* temp = head;
        head = head->next;
        if (head != nullptr) head->prev = nullptr;
        else tail = nullptr;
        delete temp;
        count--;
    }

    void pop_back() {
        if (tail == nullptr) throw std::underflow_error("List is empty");
        DNode<T>* temp = tail;
        tail = tail->prev;
        if (tail != nullptr) tail->next = nullptr;
        else head = nullptr;
        delete temp;
        count--;
    }

    // --- Поиск ---
    DNode<T>* find(const T& value) const {
        for (DNode<T>* cur = head; cur != nullptr; cur = cur->next)
            if (cur->data == value) return cur;
        return nullptr;
    }

    // --- Вставка после узла ---
    void insert_after(DNode<T>* node, const T& value) {
        if (node == nullptr) { push_front(value); return; }
        DNode<T>* newNode = new DNode<T>(value);
        newNode->next = node->next;
        newNode->prev = node;
        if (node->next != nullptr) node->next->prev = newNode;
        else tail = newNode;
        node->next = newNode;
        count++;
    }

    // --- Удаление узла ---
    void erase(DNode<T>* node) {
        if (node == nullptr) return;
        if (node == head) { pop_front(); return; }
        if (node == tail) { pop_back();  return; }
        node->prev->next = node->next;
        node->next->prev = node->prev;
        delete node;
        count--;
    }

    // --- Реверс (продвинутый уровень) ---
    void reverse() {
        if (head == nullptr) return;
        DNode<T>* cur = head;
        DNode<T>* tmp = nullptr;
        while (cur != nullptr) {
            tmp = cur->prev;
            cur->prev = cur->next;
            cur->next = tmp;
            cur = cur->prev;
        }
        std::swap(head, tail);
    }

    // --- Вывод ---
    void print() const {
        cout << "[";
        for (DNode<T>* cur = head; cur != nullptr; cur = cur->next) {
            cout << cur->data;
            if (cur->next != nullptr) cout << ", ";
        }
        cout << "]" << endl;
    }

    void printReverse() const {
        cout << "[";
        for (DNode<T>* cur = tail; cur != nullptr; cur = cur->prev) {
            cout << cur->data;
            if (cur->prev != nullptr) cout << ", ";
        }
        cout << "]" << endl;
    }

    // --- Очистка ---
    void clear() {
        while (head != nullptr) {
            DNode<T>* temp = head;
            head = head->next;
            delete temp;
        }
        tail = nullptr;
        count = 0;
    }

    int size()  const { return count; }
    bool empty() const { return head == nullptr; }

    Iterator begin() { return Iterator(head); }
    Iterator end()   { return Iterator(nullptr); }
};
```
Файл main.cpp
```cpp
#include <iostream>
#include <locale>
#include <windows.h>
#include "Lists.h"

using namespace std;

// Демонстрация варианта 12: копирование списка в другой
template <typename T>
DoublyLinkedList<T> copyList(DoublyLinkedList<T>& source) {
    DoublyLinkedList<T> result;
    for (auto it = source.begin(); it != source.end(); ++it)
        result.push_back(*it);
    return result;
}

int main() {
    system("chcp 1251 > nul");
    setlocale(LC_ALL, "Russian");

    cout << "=== Лабораторная 4. Продвинутый уровень (Вариант 12) ===\n";
    cout << "=== Односвязный и двусвязный списки ===\n\n";

    // 1. ОДНОСВЯЗНЫЙ СПИСОК
    cout << "--- 1. Односвязный список ---\n";
    SinglyLinkedList<int> sll;
    sll.push_back(10);
    sll.push_back(20);
    sll.push_back(30);
    sll.push_front(5);
    cout << "Список: ";          sll.print();
    cout << "Размер: " << sll.size() << "\n";
    sll.pop_front();
    cout << "После pop_front: "; sll.print();
    sll.pop_back();
    cout << "После pop_back:  "; sll.print();
    cout << "find(20): " << (sll.find(20) ? "найден" : "не найден") << "\n";
    cout << "remove(20): " << (sll.remove(20) ? "удалён" : "не удалён") << "\n";
    cout << "Итог: ";            sll.print();
    cout << "\n";

    // 2. ДВУСВЯЗНЫЙ СПИСОК
    cout << "--- 2. Двусвязный список ---\n";
    DoublyLinkedList<int> dll;
    for (int i = 1; i <= 5; i++)
        dll.push_back(i * 10);

    cout << "Исходный список:     "; dll.print();
    cout << "Обратный порядок:    "; dll.printReverse();
    cout << "Размер: " << dll.size() << "\n";
    cout << "Список пуст?         " << (dll.empty() ? "да" : "нет") << "\n\n";

    // 3. ИТЕРАТОРЫ (продвинутый уровень)
    cout << "--- 3. Обход итератором ---\n";
    cout << "Прямой обход:  ";
    for (auto it = dll.begin(); it != dll.end(); ++it)
        cout << *it << " ";
    cout << "\n\n";

    // 4. РЕВЕРС (продвинутый уровень)
    cout << "--- 4. Реверс списка ---\n";
    dll.reverse();
    cout << "После reverse():     "; dll.print();
    dll.reverse();
    cout << "После повторного reverse(): "; dll.print();
    cout << "\n";

    // 5. POP_BACK (вариант 12)
    cout << "--- 5. pop_back ---\n";
    dll.pop_back();
    cout << "После pop_back:      "; dll.print();
    cout << "\n";

    // 6. КОПИРОВАНИЕ СПИСКА (вариант 12)
    cout << "--- 6. Копирование списка ---\n";
    DoublyLinkedList<int> copyList1 = copyList(dll);
    cout << "Копия (copyList):    "; copyList1.print();

    copyList1.push_back(999);
    cout << "Копия + push_back:   "; copyList1.print();
    cout << "Оригинал не изменён: "; dll.print();

    DoublyLinkedList<int> copyList2(dll);
    cout << "Копия (конструктор): "; copyList2.print();
    cout << "\n";

    // 7. CLEAR И EMPTY
    cout << "--- 7. Очистка списка ---\n";
    dll.clear();
    cout << "После clear() размер = " << dll.size()
         << ", empty = " << (dll.empty() ? "да" : "нет") << "\n\n";

    return 0;
}
```
## 4. Результаты работы программы
https://2026-11-04_19-02-44.png/

## 5. Анализ сложности операций
Операция	Односвязный	Двусвязный	Обоснование
push_front	O(1)	O(1)	Меняем head и один-два указателя.
push_back	O(1) с tail	O(1) с tail	Есть указатель на хвост.
pop_front	O(1)	O(1)	Меняем head.
pop_back	O(n)	O(1)	В односвязном нужен обход до предпоследнего.
find	O(n)	O(n)	Линейный поиск.
remove(value)	O(n)	O(n) поиск + O(1)	Поиск доминирует.
insert_after(node)	O(1)	O(1)	Если узел известен — перестановка указателей.
erase(node)	—	O(1)	Есть prev.
reverse()	O(n)	O(n)	Один проход, O(1) памяти.
Итерация (begin/end)	O(1)	O(1)	Возвращаем указатель.
operator++	O(1)	O(1)	ptr = ptr->next.
operator--	—	O(1)	ptr = ptr->prev.
Копирование	O(n)	O(n)	Глубокое копирование.
Память на узел	данные + 1 указатель	данные + 2 указателя	—
## 6. Ответы на контрольные вопросы
Что такое связный список? Чем он отличается от массива?
Связный список — структура данных, где элементы (узлы) хранятся в памяти не непрерывно. Каждый узел содержит данные и указатель(и) на следующий (и/или предыдущий) узел. В отличие от массива, доступ по индексу — O(n), зато вставка/удаление в начало — O(1) без сдвига элементов.

В чём разница между односвязным и двусвязным списком?
В односвязном каждый узел хранит только указатель next, поэтому обход возможен только в одном направлении. В двусвязном узел хранит prev и next, что позволяет обходить список в обе стороны, вставлять/удалять узел за O(1) по указателю и удалять с конца за O(1).

Как реализовать вставку в начало односвязного списка?
Создать новый узел, установить newNode->next = head, затем head = newNode. Если список был пуст, обновить и tail. Сложность O(1).

Как реализовать вставку в конец списка?
При наличии указателя tail — O(1): tail->next = newNode; tail = newNode. Без tail пришлось бы идти до конца — O(n). В нашей реализации tail хранится, поэтому O(1).

Почему удаление с конца односвязного списка — O(n), а двусвязного — O(1)?
В односвязном, чтобы удалить последний узел, нужно найти предпоследний — для этого нужен обход с начала. В двусвязном у последнего узла есть prev, поэтому tail = tail->prev даёт O(1).

Как реализовать поиск элемента по значению?
Пройти по узлам от head до nullptr, сравнивая data с искомым значением. Сложность O(n) для обоих списков.

Как правильно удалить узел, чтобы не было утечки памяти?
Сохранить указатель на удаляемый узел, переставить указатели соседей, затем вызвать delete для узла. В двусвязном обязательно обновить и prev, и next.

Зачем нужен деструктор в классе списка?
Чтобы освободить память всех узлов. Деструктор вызывает clear(), который последовательно удаляет узлы через delete. Без этого возникла бы утечка памяти.

Что делает метод clear()?
Последовательно удаляет все узлы списка, обнуляет head, tail и count. После вызова список пуст, но объект списка остаётся работоспособным.

Что такое итератор? Зачем он нужен?
Итератор — объект, обеспечивающий единообразный доступ к элементам контейнера. Позволяет обходить контейнер в цикле (for (auto it = list.begin(); it != list.end(); ++it)), не раскрывая внутреннее устройство структуры.

Как реализовать оператор ++ для итератора двусвязного списка?
Iterator& operator++() { ptr = ptr->next; return *this; }. Для -- — ptr = ptr->prev.

Как работает метод reverse() для двусвязного списка?
Проходим по всем узлам, у каждого меняем местами prev и next. После обхода меняем местами head и tail. Сложность O(n), дополнительная память O(1).

Как скопировать один список в другой?
Двумя способами: (1) конструктор копирования — создаёт новые узлы и копирует значения; (2) вспомогательная функция copyList(), которая обходит исходный список итератором и добавляет элементы в новый через push_back. Оба варианта дают глубокую копию — изменение копии не влияет на оригинал.

Чем отличается поверхностное копирование от глубокого?
При поверхностном копировании копируются только указатели — оба списка указывают на одни и те же узлы, что приводит к двойному delete. При глубоком копировании создаются новые узлы с копиями данных — списки полностью независимы.

Как избежать двойного удаления при копировании?
Либо запретить копирование (= delete), либо реализовать глубокое копирование (конструктор копирования + оператор присваивания по схеме copy-and-swap). В нашей работе для DoublyLinkedList реализовано глубокое копирование, для SinglyLinkedList копирование запрещено.

## 7. Выводы
В ходе выполнения работы я реализовал шаблонные классы SinglyLinkedList<T> и DoublyLinkedList<T>, освоил операции вставки, удаления, поиска и обхода элементов.

Для односвязного списка реализованы базовые операции: push_front, push_back, pop_front, pop_back, find, remove, print, clear. Показано, что при наличии указателя tail операции вставки в конец и удаления из начала работают за O(1), а удаление с конца требует O(n) из-за обхода.

Для двусвязного списка реализованы обязательные операции варианта 12 (push_back, pop_back, empty) и дополнительная функциональность (copyList — копирование в другой список через конструктор копирования и вспомогательную функцию).

Дополнительно (продвинутый уровень) реализованы итераторы — позволяют использовать современный цикл for (auto it = list.begin(); it != list.end(); ++it) — и метод reverse() — разворот списка за O(n) без выделения дополнительной памяти.

Реализация глубокого копирования (конструктор копирования + copy-and-swap в операторе присваивания) гарантирует независимость копий: после добавления 999 в копию оригинал остался неизменным. Для односвязного списка копирование запрещено через = delete — это предотвращает двойное удаление и делает класс безопасным при работе с памятью.

Проведён анализ сложности: двусвязный список выигрывает при удалении с конца (O(1) против O(n) у односвязного) и поддерживает обратный обход, но требует больше памяти на узел (два указателя вместо одного). Это классический trade-off между памятью и производительностью.