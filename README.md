# API v1

Всё взаимодействие происходит в формате JSON (как запросы к серверу, так и ответы от сервера).
Все ресурсы должны отдаваться по HTTPS
Во все запросы нужно добавлять следующие заголовки:

* `Content-Type: application/json`
* `Accept: application/json`
* `Authorization`: token-пользователя, который генерируется при регистрации (кроме запроса на регистрацию)
* `X-Timezone: -5` (для определения таймзоны я использую скрипт https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Date/getTimezoneOffset)

### Ошибки

    {
      "error": {
        "code": 0,
        "message": "string"
      },
      "success": false
    }

## Пользователь
### Отправка СМС
    POST /api/v1/user/verify
    AUTH no
    PARAMS (POST body)
      {
          "phone": "+79999999999"
      }
    RESPONSE
    {
      "success": true,
      "payload": {}
    }

### Авторизация
    POST /api/v1/user/auth
    AUTH no
    PARAMS (POST body)
      {
        "phone": "+79999999999",
        "code": "0000"
      }
    RESPONSE
    {
      "success": true,
      "payload": {
        "token": "string"
      }
    }

### Информация о текущем пользователе
    GET /api/v1/user
    AUTH yes
    PARAMS no
    RESPONSE
    {
      "success": true,
      "payload": {
        "id": 1,
        "name": "Ivan Ivanov",
        "phone": "+7 999 999-99-99",
        "email": "ivanivanov@mail.com",
        "mail_confirmed": true, //Почта подтверждена
        "token_notification": String (Пришлю тебе, как только пользователь разрешит получение Уведомлений)
        "cards": [
            {
                "number": "*2878",
                "type": "Visa",
                "image": "https://...png" (иконка типа карты (Apay, Gpay, Visa, Mastercard и т.п.)
                
            }
        ]
      }
    }
### Изменние пользовательских данных
    PUT /api/v1/user
    AUTH no
    PARAMS (POST body)
      {
          "name": "Ivangovnov",
          "email": "uru@ru.ru",
          ....
      }
    RESPONSE
    {
      "success": true,
      "payload": {}
    }
### Изменние телефона
После изменения телефона уходит СМС с кодом подтвердени

    POST /api/v1/user/phone
    AUTH no
    PARAMS (POST body)
      {
          "phone": "+79999999999"
      }
    RESPONSE
    {
      "success": true,
      "payload": {}
    }

### Подтверждение изменения телефона
    POST /api/v1/user/auth
    AUTH no
    PARAMS (POST body)
      {
        "phone": "+79999999999",
        "code": "0000"
      }
    RESPONSE
    {
      "success": true,
    }
### Изменние емайла
После изменения email должна прийти ссылка с подтверждением на почту

    POST /api/v1/user/email
    AUTH no
    PARAMS (POST body)
      {
          "email": "uru@ru.ru"
      }
    RESPONSE
    {
      "success": true,
      "payload": {}
    }
      
## Карты оплаты
Скорее всего этот функционал будет реализован через SDK в обход сервера, 
поэтому все что касается карт можешь пропускать
### Список карт
    GET /api/v1/user/cards
    AUTH no
    PARAMS 
    RESPONSE
    {
      "success": true,
      "payload": [
        {
            "id": 1,
            "number": "*2878",
            "type": "Visa",
            "image": "https://sun9-20.userapi.com/c854320/v854320145/15e9ce/kV_sZx0aaFE.jpg" (иконка типа карты (Apay, Gpay, Visa, Mastercard и т.п.)
        }
      ]
    }
### Привязка карты
    POST /api/v1/user/card
    AUTH no
    PARAMS (POST body)
      {
          "number": "4111111111111111",
          "cvv": "123",
          "expDate": "10/20",
          "name": "IVAN IVANOV"
      }
    RESPONSE
    {
      "success": true
    }
### Удаление карты
    DELETE /api/v1/user/card/:card_id
    AUTH no
    PARAMS 
    RESPONSE
    {
      "success": true
    }


## Кофейни
### Список кофейн
    GET /api/v1/coffeehouse
      AUTH yes
      PARAMS (QUERY STRING)
        lat=55.55 (lat и lon используете для подсчета растояния)
        lon=55.55 
        page=1 (сортируете по дистанции)
        search="Пат" (поиск по строке)
      RESPONSE
      {
        "success": true,
        "payload": [
          {
            "id": 1,
            "name": "Патрики",
            "address": "Большой Патриарший переулок, 8к2",
            "coordinates": {
              "lat": 56.2235232 (число),
              "lon": 35.5463434
            },
            "access_mode": true (пропускной режим) 
            "working_time": {
               "current": { //График работы на текущий день
                 "open_time": 1583495005, //откроется 06.03.2020, 16:43:25
                 "close_time": 1583500525 //закроется 06.03.2020, 19:05:25
               },
              "next": { //График работы на следующий день
                  "open_time": 1583673325, //откроется 08.03.2020, 18:15:25
                  "close_time": 1583759725 //закроектся 09.03.2020, 18:15:25
               }
            },
            "preview_image": "https://...png", (миниатура 100х100, обязательно по https) 
            "distance": 1.221, (расстояние до кофейни в км.)
            "active": true, (обслуживает ли в данный момент кофейня)
            "order_step": [date] (4 даты с шагом в 5 минут, начиная с 10 мин. смотрим макет Оформление заказа, мб в timestamp)
          }
        ]
      }
### На карте
    GET /api/v1/coffeehouse/maps
      AUTH yes
      PARAMS (QUERY STRING)
        lat=55.55
        lon=55.55
        latitudeDelta=0.0922 (используя эти данные вы вычисляете область видимости на карте)
        longitudeDelta=0.0421 
      RESPONSE
      {
          "success": true,
          "payload": [
              { 
                "id": 1,
                "coordinates": {
                  "lat": 56.2235232,
                  "lon": 35.5463434
                },
                "active": true (обслуживает ли в данный момент кофейня)
              }
          ]

      }
### Карточка кофейни
    GET /api/v1/coffeehouse/:id
      AUTH yes
      PARAMS no
      RESPONSE
      {
          "success": true,
          "payload":  [
             {
                "id": 1,
                "coordinates": {
                  "lat": 56.2235232,
                  "lon": 35.5463434
                },
                "name": "Патрики",
                "address": "Большой Патриарший переулок, 8к2",
                "title": "Как добраться",
                "description": "Выходите из метро «Чистые пруды» ...",
                "phone": "+7999999999",
                "image": "https://...png", (детальное изображение, обязательно по https),
                "access_mode": true (пропускной режим) 
                "working_time": {
                   "current": { //График работы на текущий день
                     "open_time": 1583495005, //откроется 06.03.2020, 16:43:25
                     "close_time": 1583500525 //закроется 06.03.2020, 19:05:25
                   },
                  "next": { //График работы на следующий день
                      "open_time": 1583673325, //откроется 08.03.2020, 18:15:25
                      "close_time": 1583759725 //закроектся 09.03.2020, 18:15:25
                   }
                },
                "active": true (обслуживает ли в данный момент кофейня)
              }
          ]

      }

## Магазин
### Категории
    GET /api/v1/categoryes
      AUTH yes
      PARAMS (QUERY STRING)
        cafe_id=1
      RESPONSE
      {
          "success": true,
          "payload":  [
              {
                "id": 1,
                "name": "Все напитки",
                "sort": 1,
                "in_search": false (чтоб я мог исключить из поиска данный каталог, см. Макет Поиска по напиткам)
                "count": 50 (кол-во привзянных товаров)
              },
              {
                "id": 2,
                "name": "Недавние напитки", (содержит ранее приобретенные напитки, уникален для каждого пользователя)
                "sort": 2,
                "in_search": false,
                "count": 6
              },
              {
                "id": 3,
                "name": "Черный кофе",
                "sort": 3,
                "in_search": true,
                "count": 4
              }
          ]
      }
### Товары
    GET /api/v1/categories/:category_id/products
      AUTH yes
      PARAMS (QUERY STRING)
        cafe_id=1
        search="Кофе"
      RESPONSE
      {
          "success": true,
          "payload":  [
              {
                "id": 1,
                "name": "Большой капучино",
                "description": "Двойная порция эспрессо и молоко: насыщенный кофейный вкус c ...",
                "price": 350,
                "image": [
                    "https://..../image.png"
                ],
                "label": [
                    {
                        color: "#F8B878",
                        value: "Новинка"
                    },
                    {
                        color: "red",
                        value: "Хит продаж"
                    }
                ]
              }
          ]
      }  
### Товар
    GET /api/v1/product/:id
      AUTH yes
      PARAMS (QUERY STRING)
        cafe_id=1
      RESPONSE
      {
          "success": true,
          "payload":  {
                "id": 1,
                "name": "Большой капучино",
                "description": "Двойная порция эспрессо и молоко: насыщенный кофейный вкус c ...",
                "price": 350,
                "sort": 1,
                "image": [
                    "https://..../image.png"
                ],
                "label": [
                    {
                        color: "#F8B878",
                        velue: "Новинка"
                    }
                ],
                "offer": { //главный адон, закреплен за товаром, без него продажа не состоится
                    "title": "Выберите вкус зерна",
                    "items": [
                        {
                            "id": 1,
                            "name": "Нейтральный",
                            "price": 0,
                            "active": true, (активным может быть только у одного из списка)
                            "images": [
                                "https:.../1.png",
                                "https:.../2.png",
                            ],
                            
                        }
                    ]
                },
                "title_type_prouduct" : "Добавить в напиток",
                "addons": [
                    {
                        id: 1,
                        title: "Выберите молоко", //группа адонов
                        image: "http://kolomna.galaxybowling.ru/smarty/timthumb.php?src=pics/subrestaurant/225.png&w=163&h=163",
                        price: 100, //Не обязательный параметр, Цена товара по умолчанию, у которого установлен active, можешь ее не вычислять если не хочешь
                        items: [
                            {
                                id: 1,
                                name: "Овсяное 2%",
                                max_quantity: 1,
                                price: 100,
                                active: false,
                                images: [],
                                quantity: 0, //необязательный, но если будешь передавать, то можно будет влиять на содержимое товара по умолчанию
                                options: [
                                    {
                                        name: "Подогреть",
                                        value: false
                                    }
                                ],
                            },
                            {
                                id: 2,
                                name: "Коровье 3.9%",
                                max_quantity: 2,
                                price: 0,
                                active: true,
                                options: [],
                                quantity: 0,
                                images: ["https://velo-port.ru/img/icons8-no_gluten 1.png", "https://velo-port.ru/img/icons8-no_gluten 1.png"],
                            },
                            {
                                id: 3,
                                name: "Козье 2.2%",
                                max_quantity: 2,
                                price: 75,
                                active: false,
                                options: [],
                                quantity: 0,
                                images: [],
                            },
                        ]
                      }    
                ]
                
         }
          
      }      
### Добавление в Корзину
    POST /api/v1/baskets
        AUTH yes
        PARAMS (POST body)
          {
            "cafe_id": 1,
            "product_id": 1,
            "price": 300,
            "quantity": 1,
            "offer": {
                "id": 1,
                "?name": "Нейтральный"
            },
            "more_products": [
                {
                    "id": 1
                    "?name": "Сахар",
                    "price": 50
                    "quantity": 1
                }
            ],
            "sum_price": 350
          }
        RESPONSE (возвращает Корзину)
### Изменение кол-ва в Корзине
    PUT /api/v1/baskets/:id
        AUTH yes
        PARAMS (POST body)
          {
            "quantity": 1
          }
        RESPONSE (возвращает Корзину)
###  Удаление из корзины
    DELETE /api/v1/baskets/:id
        AUTH yes
        PARAMS (POST body)
          { }
        RESPONSE (возвращает Корзину)
### Корзина
    GET /api/v1/baskets
      AUTH yes
      PARAMS (QUERY STRING)
        cafe_id=1
      RESPONSE
      {
          "success": true,
          "payload":  [
              {
                "id": 112,
                "product_id": 1,
                "name": "Большой капучино",
                "price": 350, (с учетом всех доп товаров)
                "quantity": 1,
                "image": "https://..../image.png",
                "offer": {
                    "id": 1,
                    "name": "Нейтральный"
                },
                "more_products": [
                    {
                        "id": 1
                        "name": "Доп. экспрессо",
                        "price": 50
                        "quantity": 1
                    }
                ]
              }
          ]
      }  
### Дополнить заказ товарами при оформлении заказа
    в products добавь возможность фильтровать по типу товара
    {
      type: "Дополнительный || Основной" (с Павлом обсуди типы)
    }
### Оформление заказа
Корзина прикрепляется автоматом, оформление у нас только после оплаты, пока этот момент я не описываю

    POST /api/v1/order
        AUTH yes
        PARAMS (POST body)
          {
            "cafe_id": 1,
            "takeaway": true, (возьмет с собой)
            "ready_date": timestamp, //Дата к которому времени должен быть готов заказ
            "comment": "Cъешь еще этих мягких французских булок да выпей чаю"
          }
        RESPONSE 
        {
          "success": true,
          "payload":  [
              {
                "order_id": 228,
                "ready_time": Date (мб в timestamp??)
              }
          ]
        }  

### Уведомления

Как только изменился статус заказа, должно уйти уведомление

     POST https://fcm.googleapis.com/fcm/send
        HEADERS 
        {
            "Authorization": "key=...", (Firebase/Settings/Cloud Messaging/Ключ сервера)
            "Content-Type": "application/json"
        }
        PARAMS (POST body)
          {
            "to" : @User.token_notification,
            "notification" : {
                "body" : "Ваш заказ готов и ожидает вас на барной стойке",
                "title" : "@Cafe.name",
                "content_available" : true,
                "sound": 1,
                "priority" : "high",
                "order_id": 411
            },
            "data" : {
                "body" : "Ваш заказ готов и ожидает вас на барной стойке",
                "title" : "@Cafe.name",
                "content_available" : true,
                "sound": 1,
                "priority" : "high",
                "order_id": 411
            }
        }
       
### Оформленные заказы
Реализовывай через статусы как корзину ("Готовится/Отменен/Готов")


    GET /api/v1/orders
        AUTH yes
        PARAMS 
        {
          status: "closed"
        }
        RESPONSE 
        {
          "success": true,
          "payload":  [
              {
                "order_id": 228,
                "create_date": timestamp // дата создания
                "ready_date": timestamp, //Дата к которому времени должен быть готов заказ
                "price": 500,
                "basket": [@Корзина] (см. экземляр корзины),
              }
          ]
        } 
### Заказ
    GET /api/v1/orders/:order_id
        AUTH yes
        PARAMS (QUERY STRING)
        RESPONSE 
        {
          "success": true,
          "payload": 
          {
                "order_id": 228,
                "create_date": timestamp // дата создания
                "ready_date": timestamp, //Дата к которому времени должен быть готов заказ
                "price": 500,
                "takeaway": true, //заберет с собой
                "status": "done",
                "coffehouses": {
                    "id": 1,
                    "name": "Патрики",
                },
                "basket": [@Корзина] (см. экземляр корзины),
                "receipt": { //Фиксальный чек
                    "inn": "022702672955",
                    "ogrn": "305861925900019",
                    "order_id": 228,
                    "date": Date, (Дата выдачи)
                    "barista": "Габдуллин Н. В.",
                    "file": "https://.../file.pdf" (файл чека в pdf)
                }
          }
          
        } 
### Повторить заказ
    POST /api/v1/orders/:order_id/repeat
        AUTH yes
        PARAMS (POST body)
          {
            "cafe_id": 1,
            "order_id": 228,
          }
        RESPONSE 
        {
          "success": true
        } 
        
### Отзыв о заказе
    POST /api/v1/orders/:order_id/review
        AUTH yes
        PARAMS (POST body)
          {
            "cafe_id": 1,
            "order_id": 228,
            "text": "text text",
          }
        RESPONSE 
        {
          "success": true
        }  
  
