# 🚀 Домашнее задание 1.3 — Kubernetes: Deployment, Service и Init-контейнеры

Репозиторий содержит решения двух практических заданий по работе с Kubernetes (MicroK8s). 
В каждом разделе описана задача, приведены ссылки на манифесты, команды проверки и продемонстрированы результаты.

---

## 📁 Структура репозитория

| Файл | Назначение |
|------|------------|
| [`deploy.yaml`](./deploy.yaml) | Deployment с двумя контейнерами (`nginx` + `multitool`) |
| [`service_1.yaml`](./service_1.yaml) | Service для доступа к репликам приложения из задания 1 |
| [`multitool-pod.yaml`](./multitool-pod.yaml) | Отдельный Pod с `multitool` для проверки доступа |
| [`deployment.yaml`](./deployment.yaml) | Deployment с Init-контейнером (`sleep 30`) |
| [`service.yaml`](./service.yaml) | Service для приложения из задания 2 |

---

## 🧩 Задание 1. Deployment и доступ к репликам из другого Pod

### 📌 Условие
Создать Deployment из двух контейнеров — **nginx** и **multitool**. 
Обеспечить доступ к репликам приложения из отдельного Pod с `multitool`. 
Продемонстрировать масштабирование и работу Service.

### ⚠️ Возникшая ошибка и её решение
При попытке запустить `nginx` и `multitool` в одном Pod **без изменения настроек** возникает конфликт портов: оба контейнера по умолчанию слушают порт `80`, а в Pod’е используется общая сетевая среда. 

**Решение:** для контейнера `multitool` задана переменная окружения `HTTP_PORT=8080`, что переводит его веб-сервер на другой порт.

**Манифест:** 👉 [Открыть `deploy.yaml`](./deploy.yaml)

### 🚀 Развёртывание, масштабирование и Service

```bash
# Применяем Deployment
kubectl apply -f deploy.yaml

# Проверяем количество подов ДО масштабирования (ожидаем 1 под)
kubectl get pods -l app=nginx-multitool

# Увеличиваем количество реплик до 2
kubectl scale deployment nginx-multitool-deployment --replicas=2

# Проверяем количество подов ПОСЛЕ масштабирования (ожидаем 2 пода)
kubectl get pods -l app=nginx-multitool

# Создаем Service
kubectl apply -f service_1.yaml
kubectl get svc nginx-multitool-svc
```

### 🧪 Проверка доступа из отдельного Pod
Для проверки используется отдельный под с утилитой `multitool` (манифест: 👉 [Открыть `multitool-pod.yaml`](./multitool-pod.yaml)).

```bash
# Запускаем тестовый под
kubectl apply -f multitool-pod.yaml

# Проверка nginx (порт 80)
kubectl exec -it multitool-client -- curl -s http://nginx-multitool-svc:80

# Проверка multitool (порт 8080)
kubectl exec -it multitool-client -- curl -s http://nginx-multitool-svc:8080
```
> **Ожидаемый результат:** в ответ на оба запроса приходит HTML-страница (от `nginx` и от `multitool` соответственно).

---

## ⏳ Задание 2. Init-контейнер с ожиданием (`sleep 30`)

### 📌 Условие
Создать Deployment с приложением **nginx** и Init-контейнером (**busybox**), который выполняется перед запуском основного контейнера. 
В данном варианте Init-контейнер просто «спит» 30 секунд — это демонстрирует механизм блокировки старта основного контейнера до завершения инициализации.

**Манифесты:** 👉 [Открыть `deployment.yaml`](./deployment.yaml) | 👉 [Открыть `service.yaml`](./service.yaml)

### 📊 Демонстрация состояний пода

```bash
# Применяем Deployment (Service пока не создаём)
kubectl apply -f deployment.yaml

# Смотрим статус пода сразу после создания
kubectl get pods -l app=nginx
# Ожидается: Init:0/1 (nginx ещё не запущен)

# Ждём 30 секунд и наблюдаем смену статусов в реальном времени
kubectl get pods -l app=nginx -w
# Init:0/1 → PodInitializing → Running (1/1)

# Создаем Service
kubectl apply -f service.yaml
kubectl get endpoints nginx-service
```
> **Результат:** после завершения Init-контейнера (через 30 секунд) под переходит в `Running`, а Service получает эндпоинты и становится доступным.

---

## 🖼️ Скриншоты выполнения

### Задание 1
**Терминал (выполнение команд):**

![Терминал Задание 1](https://github.com/user-attachments/assets/a47ba1a5-a4ae-4e20-8d7f-9f55a1d587a3)

**Веб-интерфейс Kubernetes Dashboard:**

![Dashboard Задание 1](https://github.com/user-attachments/assets/51c18dcd-4acc-45b5-b831-86939acfd371)

---

### Задание 2
*Чтобы картинки отобразились, загрузите их в репозиторий и укажите правильный путь вместо `./screenshot3.png`*

**Терминал (проверка статусов пода):**

![Терминал Задание 2](./screenshot3.png)

**Веб-интерфейс Kubernetes Dashboard:**

![Dashboard Задание 2](./screenshot4.png)

---

## 🛠️ Используемые инструменты

- **Kubernetes** (MicroK8s)
- **kubectl**
- Образы: `nginx:1.27`, `wbitt/network-multitool`, `busybox:1.36`

---
<p align="center">
  <i>Документ подготовлен в рамках выполнения домашнего задания.</i>
</p>
