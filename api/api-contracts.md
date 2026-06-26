# Проектирование API и контрактов (этап 2)

## Декомпозиция на микросервисы
Мы разбиваем монолит на 4 независимых сервиса, у каждого из которых своя база данных.

1. **Identity Service (User Service):** отвечает только за регистрацию, автоматизацию и профили. Ничего не знает о комнатах.
2. **Catalog Serice (Space Service):** отвечает за витрину. Хранит данные о комнатах, их характеристиках и базовом расписании.
3. **Booking Service:** отвечает за транзакции бронирования. Хранит данные о том, кто заброниовал, что забронировал, на какое время и когда.
4. **PaymentService:** отвечает за оплату помещений после бронирования

## Описание API (REST Endpoints)

**Identity Service**
* POST /api/v1/Users - регистрация
* GET /api/v1/Users/{Id} - получить профиль

**Catalog Service**
*GET /api/v1/spaces - список всех комнат (с фмльтрами)
* GET /api/v1/spaces/{Id}/availablity?date=YYYY-DD-MM - получить свободные слоты для комнаты на дату

**Booking Service**
* POST /api/v1/bookings - создать бронь
* GET /api/v1/bookings/{Id} - детали брони
* DELETE /api/v1/bookings/{Id} - отменить бронь

**PaymentService**
* POST /api/v1/payments - Создать платеж
* GET  /api/v1/payments/{id} - Статус платежа
* POST /api/v1/payments/{id}/confirm - подтвердить оплату

## Контракты (DTO)

Формат - JSON

**Зпрос на создание брони (POST /api/v1/bookings):**
```
{
  "userId": "123e4567-e89b-12d3-a456-426614174000",
  "spaceId": "987fcdeb-51a2-43d7-9012-3456789abcde",
  "startTime": "2023-11-20T10:00:00Z",
  "endTime": "2023-11-20T11:30:00Z"
}
```

**Успешный ответ (201 Created):**
```
{
  "bookingId": "550e8400-e29b-41d4-a716-446655440000",
  "status": "Active",
  "totalPrice": 1500.00,
  "createdAt": "2023-11-19T15:00:00Z"
}
```

## Обработка ошибок

Используем стандартный HTTP коды. Сервис возвращает единый объект ошибки:
```
{
  "errorCode": "SPACE_ALREADY_BOOKED",
  "message": "The selected time slot is already reserved.",
  "timestamp": "2023-11-19T15:05:00Z"
}
```



* 400 Bad Request - некорректные даты
* 401 Unauthorized - пользователь не авторизован
* 404 Not Found - комната или бронь не найдена
* 409 Conflict - время уже занято, конфликт данных


## Диаграмма последовательности: Создание бронирования

<img width="948" height="951" alt="image" src="https://github.com/user-attachments/assets/67a0e947-2cb6-4914-9546-6e4daca757d9" />




### Описание сценария:
1. Клиент отправляет запрос на создание брони.
2. Booking Service проверяет доступность комнаты через Catalog Service
3. Проверяется наличие пересечения по времни в БД.
4. При успехе бронь сохраняется и клиенту возвращается её Id.


## Алтернативный сценарий:

Диаграмма для случая, когда комната уже занята

<img width="1040" height="858" alt="image" src="https://github.com/user-attachments/assets/56cac4fb-79fb-4118-9c89-0070f60d5585" />


