# Manual da Integração Agricash
## Autenticação

## 1 - Introdução

Para que o acesso às APIs sejam feitos de forma segura será necessário que o usuário autentique suas requisição com um token. Será fornecido a informação necessária para que o token seja gerado, bem como o endpoint de API que permita essa autenticação.
O serviço de importação se divide em duas etapas. Envia-se um payload JSON contendo os dados do cadastro junto com a o CNPJ do emissor e o CNPJ do fundo do qual será gerado o limite. Este objeto contém os dados do portador, também referenciado como produtor ou devedor, bem como os dados de sua propriedade rural, proprietários ou arrendatários desta propriedade, matrículas e os valores para o limite.
As informações de propriedades rurais e matrículas são coletadas de forma que seja composto um objeto onde cada propriedade rural possui uma lista de matrículas. Cada propriedade deve ter, então, no mínimo uma matrícula. Os proprietários são configurados para a propriedade rural e a titularidade é especificada através dos tipos, podendo ser proprietário ou arrendatário.
Os dados importados ***serão tomados como fonte da verdade***, ***substituindo*** os dados que estão no Agricash.

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