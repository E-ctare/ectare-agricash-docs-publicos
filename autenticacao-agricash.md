# Manual da Integração Agricash
## Autenticação

## 1 - Introdução

Para que o acesso às APIs sejam feitos de forma segura será necessário que o usuário autentique suas requisição com um token. Será fornecido a informação necessária para que o token seja gerado, bem como o endpoint de API que permita essa autenticação.

## 2 - Ambientes

O Agricash conta com dois ambientes principais. São eles:

produção:  
~~~
https://api.agricash.ectare.com.br/v2  
~~~
 
desenvolvimento:  
~~~
https://api.dev.agricash.ectare.com.br/v2
~~~

## 3 - Autenticação

Para obter um token, o usuário deve estar portando seu **clientId** e seu **clientSecret**. Esses dados são informados no momento da contratação dos serviços. Será também necessário informar a API a qual deseja-se conectar. 

### 3.1 - API de Autenticação

Realizar requisição POST de acordo com o CURL disponibilizado abaixo:

~~~curl
curl --request POST \
  --url {{AMBIENTE}}/ectare-gerenciador-autenticacao/oauth/token \
  --header 'content-type: application/json' \
  --data '{
  "client_id": "{{SEU_CLIENT_ID}}",
  "client_secret": "{{SEU_CLIENT_SECRET}}",
  "audience": "{{SUA_API}}"
}'
~~~

O retorno desta requisição será, em caso de sucesso, o seguinte objeto:
~~~json
{
    "access_token":"eyJhbGciOiJIUzI1N...",
    "scope":"user:read",
    "issued_at":1753971859,
    "expires_in":3600,
    "expires_at":1753971859
}
~~~

Caso a requisição falhe, erros específicos serão retornados contendo a mensagem de erro
~~~json
{
    "mensagem":"mensagem do erro"
}
~~~
