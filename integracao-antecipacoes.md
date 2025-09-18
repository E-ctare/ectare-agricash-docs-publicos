# Manual da Integração Agricash
## Registro de antecipações

## Autenticação

Para autenticar-se com as APIs da e-ctare, favor seguir os passos descritos [aqui](./autenticacao-agricash.md)

## 1 - API de registros

Para realizar um registro de uma coleta, o serviço deve ser acessado enviando o payload contendo os dados necessários para cada situação.

### 1.1 - Explicação do payload

O sistema está preparado para lidar com duas situações: coleta de produto ou recebimento de uma nota fiscal. O serviço é o mesmo mas o payload precisa das informações de cada contexto.

```
{
    "cnpjEmissor": string : obrigatorio | sem pontuação
    "cpfCnpjPortador": string : obrigatorio | sem pontuação
    "nomePortador" : obrigatorio
    "telefonePortador" : obrigatorio | sem pontuacao ou mascara | ex.: 16999999999
    "tipo": string : obrigatorio | [coleta] ou [nota]
    "dataColeta": string : obrigatorio se [coleta] | padrão dd/mm/yyyy
    "descricao": string : obrigatorio se [coleta] | leite; algodão; arroz etc.
    "quantidade": number : obrigatorio se [coleta] | ex.: 200.10; 400; 2.50 etc.
    "medida": string : obrigatorio se [coleta] | litros; kg; sacas; pacotes etc.
    "valorUnidade": number : obrigatorio se [coleta] | valor estimado de cada medida coletada
    "adiantamento": opcional; ignorado se [nota] | para o valor mínimo do limite
    "notaFiscal": objeto | obrigatorio se [nota] | {
        "base64": string | obrigatorio se url vazio | o arquivo em codificação base64
        "url": string : obrigatorio se base64 vazio | url do arquivo para ser baixado
    }
}
```

### 1.2 - Compreensão dos dados

*cnpjEmissor* é o cnpj da cooperativa

*cpfCnpjPortador* é o documento do produtor ou fornecedor da nota

*tipo* é o tipo da origem do registro. Pode ser uma coleta de produto ou uma nota fiscal de um fornecedor

*dataColeta* é a data, no formato brasileiro (dd/mm/yyyy), em que foi feita a coleta do produto

*descricao* é o tipo de produto ou qualquer outra informação que ajude a identificar o conteúdo da coleta

*quantidade* é um valor que, aliado à medida, forma a informação de quantos litros, kilos, pacotes foram coletados

*medida* é a informação sobre a medida do que foi coletado. Para leite, usar litros ou L, por exemplo

*valorUnidade* é o valor em reais que cada unidade, daquela medida, vale

*adiantamento* é o valor que configura o sistema de adiantamentos (mínimo)

*notaFiscal* é um objeto para quando a origem da integração for do tipo nota. Este objeto possui: *url* e *base64*. O arquivo será baixado caso a url seja válida. Caso contrário, o base64 será utilizado para obter o arquivo da nota fiscal. Caso os dois estejam preenchidos, a prioridade é dada ao base64.


## 1.3 - Exemplo de payload - Contexto de coleta

```json
{
    "cnpjEmissor": "68531428000102",
    "cpfCnpjPortador":"84913750070",
    "nomePortador":"Cristiano Silva",
    "telefonePortador":"35912345678",
    "tipo":"coleta",
    "dataColeta":"19/05/2025",
    "descricao": "leite",
    "quantidade": 120,
    "medida":"litro",
    "valorUnidade":0.86,
    "adiantamento":16000.50
}
```

## 1.4 - Exemplo de payload - Contexto de nota fiscal

a) Passando a nota como base64

```json
{
    "cnpjEmissor": "06171175000148",
    "cpfCnpjPortador":"51340919000165",
    "nomePortador": "Cristiano Silva",
    "telefonePortador": "35912345678",
    "tipo":"nota",
    "notaFiscal":{
        "base64":"iVBORw0KGgoAAAANSUhEUgAAAQAAAAEACA",
    }
}
```

b) Passando a nota como url

```json
{
    "cnpjEmissor": "06171175000148",
    "cpfCnpjPortador":"51340919000165",
    "nomePortador": "Cristiano Silva",
    "telefonePortador": "35912345678",
    "tipo":"nota",
    "notaFiscal":{
        "url":"https://imagem-da-nota-fiscal/arquivo.xml",
    }
}
```

## 2 - CURL do serviço

Como informado na sessão de autenticação, a e-ctare conta com dois ambientes: desenvolvimento e produção.
Para facilitar a compreensão dos enpoints, vamos abreviar seus nomes para {{AMBIENTE}}, seja DEV ou PROD.

Também será necessário um token de acesso para utilizar as APIs. Do mesmo modo, aqui nos referimos ao parâmetro como {{TOKEN}}

### 2.1 - Request

Exemplo com CURL:
~~~curl
curl --request POST \
  --url {{AMBIENTE}}/antecipacoes/integracoes/registros \
  --header 'authorization: {{TOKEN}}' \
  --header 'content-type: application/json' \
  --data '{}'
~~~

### 2.2 - Response

201 - Created


Erros 
~~~
    404 - Not found
    Caso os dados enviados não encontrem referencias na nossa base
    Caso o enpoint não seja encontrado
~~~
~~~
    400 - Bad request
    Caso os dados enviados sejam inválidos ou incompletos
~~~
~~~
    500 - Internal server error
    Caso a integração falhe por indisponibilidade dos nossos serviços
~~~
