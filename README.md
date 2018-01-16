# Приложение для создания и редактирования информации о встречах сотрудников

Написано для Node.js 8 и использует библиотеки:
* express
* sequelize
* graphql

## Задание
Код содержит ошибки разной степени критичности. Некоторых из них стилистические, а некоторые даже не позволят вам запустить приложение. Вам необходимо найти и исправить их.

Пункты для самопроверки:
1. Приложение должно успешно запускаться
2. Должно открываться GraphQL IDE - http://localhost:3000/graphql/
3. Все запросы на получение или изменения данных через graphql должны работать корректно. Все возможные запросы можно посмотреть в вкладке Docs в GraphQL IDE или в схеме (typeDefs.js)
4. Не должно быть лишнего кода
5. Все должно быть в едином codestyle

## Запуск
```
npm i
npm run dev
```

Для сброса данных в базе:
```
npm run reset-db
```

## Решение
Исправленный рабочий проект лежит в этом репозитории
Исходный проект можно найти [Здесь](https://github.com/yandex-shri-msk-2018/entrance-task-1)
### 1.Ошибка при запуске програмы
При запуске программы в консоль выводится следующая ошибка:
![Ошибка запуска](https://github.com/dimegusew/Yandex-Entrance-task-1/raw/master/images/1error.png)


Ошибка указывает, что при создании нового объекта от конструктора Sequelize не указан диалект базы данных.

Переходим в \node_modules\sequelize\lib\sequelize.js
Ищем в документе где вызывается конструктор
New Sequelize
Находим вызов конструктора Sequelize в \models
Видим что при инстансе нового объекта диалект указан

```
const sequelize = new Sequelize(null, null, {
  dialect: 'sqlite',
  storage: 'db.sqlite3',

operatorsAliases: { $and: Op.and },

  logging: false
});
```
Смотрим в \node_modules\sequelize\lib\sequelize.js
Видим, что Options идут третьим аргументом:

```
 // new Sequelize(database, username, password, { ... options })
```

Ставим перед {…Options} еще один null

```
const sequelize = new Sequelize(null, null, null {
  dialect: 'sqlite',
  storage: 'db.sqlite3',

operatorsAliases: { $and: Op.and },

  logging: false
});
```

Программа успешно запустилась

### 2.Ошибка при переходе по ссылке  http://localhost:3000/graphql/

При переходе по ссылке  http://localhost:3000/graphql/ в браузере высвечивается ошибка :
Cannot Get/graphQl/

Ошибка заключается в опечатке записи роута:
```
app.use('/graphgl', graphqlRoutes);
```
Заменим на:

```
app.use('/graphgl', graphqlRoutes);
```

Приложение запустилось


### 3.Ошибки при запросах и записи на сервер
При проверке всех запросов было выявлеено что запрос events:[Event] выдает ошибку

Переходим в \graphql\resolvers\query.js
Находим events

```
events (root, args, context) {
    return models.Event.findAll(argumets, context);
  },
```
Аргументом в events передает arg, а возвращает метод findAll c аргументом arguments
При изменении arguments на args ошибка пропадает

При проверке всех mutations было выявлено, что отсутствует запрос addUserToEvent указанный в Doc

Создаем Запрос:

```
  addUserToEvent (root, { id, userId }, context) {
    return models.Event.findById(id)
            .then(event => {
               event.addUsers(userId);
               return event
            });
  },
```

Запрос сработал

Но при запросе events users и room равны null

Используя средство для просмотра баз данных (в данном случае использовался sqlite browser https://sqliteonline.com/#
Понимаем, что запись прошла успешно, значит проблема не в записи, а в запросе из базы данных.

Находим что в \graphql\resolvers\index.js функции users и room ничего не возвращают.

```
	module.exports = function resolvers () {
  return {
    Query: query,

    Mutation: mutation,

    Event: {
      users (event) {
        event.getUsers();
      },
      room (event) {
        event.getRoom();
      }
    },

    Date: GraphQLDate
  };
};

```
Исправляем

```
module.exports = function resolvers() {
  return {
    Query: query,

    Mutation: mutation,

    Event: {
      users(event) {
        return event.getUsers();
      },
      room(event) {
        return event.getRoom();
      }
    },

    Date: GraphQLDate
  };
};

```
Все работает


**UPD** Во время работы с фронтендом обнаружилась еще одна неточность: функция обновления события не принимала айди юзеров и айди комнаты:

```
  updateEvent (root, { id, input }, context) {
    return models.Event.findById(id)
            .then(event => {
              return event.update(input);
            });
  },
```

Исправление:

```
  updateEvent (root, { id, input,roomId,usersIds }, context) {
    return models.Event.findById(id)
            .then(event => {
              event.setRoom(roomId);
              ;

              return event.setUsers(usersIds)
              .then(()=>event.update(input))

            });
  },

```
