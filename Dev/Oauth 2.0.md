https://youtu.be/OZ8s7CP3kXQ

Authorization Code Flow
1. Пользователь запрашивает логин через Oauth
2. Приложение (Auth Client) перенаправляет 302 пользователя на Auth Server, где ему надо будет пройти авторизацию (передает: client_id, redirect_uri, scope, response_type=code)
3. Auth Server запрашивает credentials пользователя
4. Auth Server перенаправляет 302 пользователя по redirect_uri на Auth Client (authorize_code, scope, state)
5. Auth Client делает запрос на Auth Server (client_id, client_secret, authorize_code, redirect_uri)
6. Auth Server возвращает 200 (access_token, refresh_token)

Implicit Flow
1. Пользователь запрашивает логин через Oauth
2. Приложение (Auth Client) перенаправляет 302 пользователя на Auth Server, где ему надо будет пройти авторизацию (передает: client_id, redirect_uri, scope, response_type=code)
3. Auth Server запрашивает credentials пользователя
4. Auth Server возвращает 200 (access_token, refresh_token)

Auth Server ()

client_id, client_secret
user, pass

http://localhost:9000/authorize?client_id=qwerty


val startDt =  
  ZonedDateTime.ofInstant(Instant.ofEpochMilli(readRequest.message.unixTimestampMs), readEntry.timezone)

