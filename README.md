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
