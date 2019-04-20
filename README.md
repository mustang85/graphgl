# graphgl
Это язык подобен языку SQL
Используется для взаимодействия с сервером и получения данных.

Среда выполнения: (graphgl сервер)
Написали запрос, отправили его на сервер -> 
GraphQl (реализован в рамках сервера) он принимает его
- разбивает его на составные части, чтобы понять что мы от него хотим
- правильно ли мы сформировали запрос
-> далее он может получить данные:
- с сервера
- диска
- локального хранилища
после чего обратно отправляет данные клиенту

GraphQl и REST
Главное в REST, это EndPoint, это определенный url (https://codedojo.ru/api/posts) 
куда мы отпралвляем запросы
posts - название сущности, что мы хотим получить Get методом
posts/1 - получить 1 пост, может быть индитификатор (String | Number)
https://codedojo.ru/api/posts/1/comments - комментарии определенного поста
https://codedojo.ru/api/users/1/posts/1/comments - получить комментарии 1 поста
https://codedojo.ru/api/posts/1?fields=id,title - получить поля id, title 1 поста
в пост также могут входить данные:
- url
- description
- author
и много чего что не нужно в данный момент
https://codedojo.ru/api/posts/1?fields=id,title - сервер поймет этот запрос и вернет
{
    "id": 1,
    "title": "Знакомство с GraphQL"
}

https://codedojo.ru/api/posts/1?fields=id,title,author - если потребуется автор поста
{
    "id": 1,
    "title": "Знакомство с GraphQL",
    "author": 42 (Идентификатор поста)
}
чтобы получить автора, делаем запрос на https://codedojo.ru/api/authors/42 получаем
{
    "id": 42,
    "firstname": "Oleg",
    "lastname": "Polyakov"
}
далее на клиенте формируем объект, после чего используем его в приложении
{
    "id": 1,
    "title": "Знакомство с GraphQL",
    "author": {
        "id": 42,
        "firstname": "Oleg",
        "lastname": "Polyakov"
    }
} тоесть пришлось выполнить 2 запроса чтобы получить финальный объект, все зависет от
того как реализовано на сервере


Посмотрим как такой же запрос будет выглядить в GraphQL:
query { -> запрос
    posts(id: 1) { -> название запроса (критерий, а не все)
        id -> далее набор полей которые хотим получить
        title
        author {
            firstname
            lastname
        }
    }
}

API как правило возвращает объект полный по максимуму, чтобы удовлетворить те или иные 
вариации на клиенте
К примеру есть пользователь, у него много полей, а для программы в данный момент нужно его имя и город 
и нужно вывести на экран 500 таких пользователей
- как минимум передаются лишние данные с запросом
- вложенность в объектах

GET запрос GraphQL
fetch('https://codedojo.ru/graphgl?query={posts(id: 1){...}}')

POST
fetch('https://codedojo.ru/graphgl', {
    method: 'POST',
    headers: {
        'Content-Type:' 'application/json'
    },
    body: JSON.stringify({
        query: `
            {
                posts(id: 1) {
                    id
                    title
                    author {
                        firstname
                        lastname
                    }
                }
            }
        `
    })
})

В GraphQL конечная точка одна, отличается только query запросом

Термины используемые в GraphQL 
Query - запрос
query {
    posts {
        id
        title
        author {
            firstname
            lastname
        }
        comments {
            text
        }
    }
}

Fields 
posts, id, author, firstname, text, etc..

Type
Перед тем как использовать или отправлять запросы на сервер, серверу необходимо 
объяснить что такое POST, объявляем тип данных POST
type Post {
    id: ID! Простые данные (по сути здесь тоже строка)
    title: String! Простые данные
    content: String Простые данные
    author: Author! Составные данные
    status: Status! Составные данные
    comments: [Comment]! Составные данные (вида массив)
}

type Author {
    id: ID!
    firstname: String!
    lastname: String!
}

! этот символ говорит GraphQl что свойство обязательно

Вместо создания нового типа, мы создаем энумерацию (уникальное значение)
enum Status {
    DRAFT
    PUBLISHED
    ARCHIVED
}

DRAFT, PUBLISHED, ARCHIVED будут выглядить как строки в сохраняемом объекте

type Comment {
    id: ID!
    title: String!
    videoId: String
}
