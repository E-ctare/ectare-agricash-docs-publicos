# Manual da Integração Agricash
## Importação de Portadores

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

## 4 - API de importação

Para realizar a importação, o Agricash solicitará os dados mínimos para a criação de limite de fundos para o portador. Nisso, um fundo com limite de financiamento deve existir entre o emissor e a instituição financiadora. Tendo esse pré-requisito sido alcançado, o usuário da importação precisará ter em mãos o CNPJ do emissor e o CNPJ do fundo utilizado no momento da criação de tal limite.

### 4.1 Explicação do payload

Favor enviar os CNPJs, CPFs, RG e CEPs sem pontuação ou máscara.

Os dados do corpo da requisição são descritos nos seguintes ***objetos estruturais***:  
```
{
    "cnpjEmissor": string : obrigatorio | sem pontuação
    "cnpjFundo": string : obrigatorio | sem pontuação
    "dados": object : obrigatorio | {
            "portador": object : obrigatorio | {
                "nome": string : obrigatorio 
                "documento": string : obrigatorio | sem mascara
                "telefone": string : obrigatorio | sem mascara Ex.: 35999999999
                "email": string : obrigatorio
                "profissao": string : obrigatorio
                "estadoCivil": string : obrigatorio | [Solteiro(a), Casado(a), União Estável, Viúvo(a), Divorciado(a)] 
                "regimeBens": string : obrigatorio caso não solteiro | [comunhao_universal, separacao_total, comunhao_parcial]
                "rg": string : obrigatorio
                "nacionalidade": string : obrigatorio
                "dataNascimento": string : obrigatorio | Ex.: 2000-12-31
                "possuiCertificadoDigital": boolean : obrigatorio | Determina o tipo da assinatura
                "garantiaPreferencial": obrigatorio | penhor ou nenhuma
                "tipoTaxaCprPreferencial": obrigatorio | prefixada ou posfixada
                "endereco": object : obrigatorio | {
                    "logradouro": string : obrigatorio
                    "numero": string : opcional
                    "complemento": string : opcional
                    "CEP": string : obrigatorio | sem mascara - usado para obter a cidade internamente
                    "bairro": string : opcional
                },
                "documentacao": list : opcional | [
                    {
                        "nome": string : obrigatorio | arquivo.png
                        "tipo": string : obrigatorio
                        "url": string : obrigatorio se não for base64
                        "bytes": string : obrigatorio se não for url 
                    }
            },
            "propriedadesRurais": list : obrigatorio | [
                {
                    "nome": string : obrigatorio
                    "areaTotal": number : obrigatorio | Ex.: 199.90 | Em hectares
                    "areaCultivada": number : obrigatorio | Ex.: 199.90 | Em hectares
                    "endereco": object : obrigatorio | {
                        "logradouro": string : obrigatorio
                        "numero": string : opcional
                        "complemento": string : opcional
                        "CEP": string : obrigatorio | sem mascara - usado para obter a cidade internamente
                    },
                    "proprietarios": list : obrigatorio [
                        {
                            "nome": string : obrigatorio
                            "documento": string : obrigatorio
                            "rg": string : obrigatorio
                            "inscricaoEstadual": string : opcional
                            "estadoCivil": string : obrigatorio | [Solteiro(a), Casado(a), União Estável, Viúvo(a), Divorciado(a)]
                            "nacionalidade": string : obrigatorio
                            "profissao": string : obrigatorio
                            "tipo": string : obrigatorio | [Proprietario, Arrendatario]
                            "qualificacaoEspecial": string : opcional
                            "obsQualificacaoEspecial": string : opcional
                            "endereco": object : obrigatorio | {
                                "logradouro": string : obrigatorio
                                "numero": "string : opcional
                                "complemento": string : opcional
                                "CEP": string : obrigatorio | sem mascara - usado para obter a cidade internamente
                                "bairro": string : opcional
                            }
                        }
                    ],
                    "documentacao": list : opcional | [
                        {
                            "nome": string : obrigatorio | arquivo.png
                            "tipo": string : obrigatorio | contrato_arrendamento
                            "url": string : obrigatorio se não for base64
                            "bytes": string : obrigatorio se não for url 
                        }
                    ],
                    "matriculas": list : obrigatorio | [
                        {
                            "matricula": string : obrigatorio
                            "cartorio": string : obrigatorio
                            "areaTotal": number : obrigatorio | Ex.: 299.18 | Em hectares
                            "areaCultivada": number : obrigatorio | Ex.: 30.99
                            "culturas": list : obrigatorio | [
                                string : obrigatorio | [corn, soy, bovine, coffee, milk, watermelon, wheat, onion, caupiBean, sunflower]
                            ],
                            "arrendado": boolean : obrigatorio,
                            "numeroLivro": string : opcional,
                            "folhaRegistro": string : opcional,
                            "documentacao": list : opcional | [
                                {
                                    "nome": string : obrigatorio | arquivo.png
                                    "tipo": string : obrigatorio
                                    "url": string : obrigatorio se não for base64
                                    "bytes": string : obrigatorio se não for url 
                                }
                            ]
                        }
                    ]
                }
            ],
            "limiteFinanciamento": {
                "valor": number : obrigatorio | Ex.: 9192839.69
                "taxa": number : obrigatorio | Ex.: 0.5
                "dataExpiracao": string : obrigatorio | Ex.: 2000-12-31
            }
        }
}
```

### 4.2 Dados constantes

O Agricash conta com a padronização de alguns campos para que os dados sigam uma estrutura. Solicitamos que os dados abaixo descritos sejam informados de acordo com as opções destinadas a ele:

#### 4.2.1 Tipos de documentos do produtor (portador):

    certidao_casamento  
    certidao_casamento_terceiro
    certidao_nascimento
    cnh_frente
    cnh_verso
    comprovante_endereco
    comprovante_endereco_terceiro
    dados_bancarios
    dados_bancarios_terceiro
    declaracao_uniao_estavel
    documento_pessoal_terceiro
    ie
    nota_fiscal_venda
    rg_frente
    rg_verso
    termo_aceite
    balancete
    cartao_cnpj
    certidao_casamento_socio
    comprovante_endereco_socio
    contrato_social
    dre
    documento_pessoal_socio
    estatuto_social
    faturamento
    recibo_ir
    declaracao_ir
    balanco
    quadro_safra
    alteracao_contratual
    declaracao_ir_socio
    recibo_ir_socio
    certificado_comercializacao_agrotoxico
    cnd

#### 4.2.2 Tipos de documentos para propriedades rurais

    analise_esg
    car
    ccir
    demarcacao_area
    itr
    quadro_safra_assinado
    certificado_florestal

#### 4.2.3 Tipos de documentos para matrículas das propriedades

    carta_anuencia
    contrato_arrendamento
    matricula_imovel

#### 4.2.4 Tipos de estados civis

    Solteiro(a)
    Casado(a)
    Divorciado(a)
    União Estável
    Separado(a)
    Viúvo(a)

#### 4.2.5 Tipos de regime de bens

    comunhao_universal
    separacao_total
    comunhao_parcial

#### 4.2.6 Tipos de titulares de propriedades rurais

Obs.: Sem acento, capitalizado

    Proprietario
    Arrendatario    

### 4.3 Realizando a requisição

A requisição deve ser do tipo POST, configurando o cabeçalho de Autenticação com o token gerado nos passos anteriores.

Exemplo com CURL:
~~~curl
curl --request POST \
  --url {{AMBIENTE}}/devedores/integracoes/importacao \
  --header 'authorization: eyJhbGciOiJIUzI1N...' \
  --header 'content-type: application/json' \
  --data '{}'
~~~


Exemplo de um payload da requisição:
~~~json
{
    "cnpjEmissor": "XXXXXXXXXXXXXX",
    "cnpjFundo": "XXXXXXXXXXXXXX",
    "dados": {
        "portador": {
            "nome": "Nome do Portador ou Produtor",
            "documento": "XXXXXXXXXXX",
            "telefone": "00000000000",
            "email": "email@email.com.br",
            "profissao": "profissao do portador",
            "estadoCivil": "Solteiro(a)",
            "regimeBens": "comunhao_parcial",
            "rg": "MMXXXXXXXX",
            "nacionalidade": "brasileiro",
            "dataNascimento": "2000-12-31",
            "possuiCertificadoDigital": false,
            "garantiaPreferencial": "penhor",
            "tipoTaxaCprPreferencial": "posfixada",
            "endereco": {
                "logradouro": "Nome da rua",
                "numero": "123",
                "complemento": "B",
                "CEP": "XXXXXXXX",
                "bairro": "Nome do Bairro"
            },
            "documentacao": [
                {
                    "nome": "cnh.jpg",
                    "tipo": "cnh",
                    "url": "https://url-do-arquivo.com.br/arquivo.jpg"
                },
                {
                    "nome": "comprovante_endereco.pdf",
                    "tipo": "comprovante_endereco",
                    "base64": "ZW5jb2RlZEZpbGVEYXRh"
                }
            ]
        },
        "propriedadesRurais": [
            {
                "nome": "Fazenda Santa Luzia",
                "areaTotal": 100.10,
                "areaCultivada": 89.2,
                "endereco": {
                    "logradouro": "Nome da estrada",
                    "numero": "123",
                    "complemento": "B",
                    "CEP": "XXXXXXXX"
                },
                "proprietarios": [
                    {
                        "nome": "Nome do proprietario",
                        "documento": "XXXXXXXXXXX",
                        "rg": "MMXXXXXXXX",
                        "inscricaoEstadual": "XXXXXXX",
                        "estadoCivil": "Solteiro(a)",
                        "nacionalidade": "brasileiro",
                        "profissao": "Profissão do proprietario",
                        "tipo": "Arrendatario ou Proprietario",
                        "qualificacaoEspecial": "",
                        "obsQualificacaoEspecial": "",
                        "endereco": {
                            "logradouro": "Nome da rua",
                            "numero": "123",
                            "complemento": "B",
                            "CEP": "XXXXXXXX",
                            "bairro": "Nome do Bairro"
                        }
                    }
                ],
                "documentacao": [
                    {
                        "nome": "itr.pdf",
                        "tipo": "itr",
                        "base64": "ZW5jb2RlZEZpbGVEYXRh"
                    },
                    {
                        "nome": "car.pdf",
                        "tipo": "car",
                        "url": "https://url-do-arquivo.com.br/arquivo.pdf"
                    }
                ],
                "matriculas": [
                    {
                        "matricula": "RG019230-M1299",
                        "cartorio": "Paraíso",
                        "areaTotal": 80,
                        "areaCultivada": 20,
                        "culturas": [
                            "corn"
                        ],
                        "arrendado": false,
                        "numeroLivro": "00000ABV",
                        "folhaRegistro": "688",
                        "documentacao": [
                            {
                                "nome": "car.pdf",
                                "tipo": "matricula_imovel",
                                "base64": "ZW5jb2RlZEZpbGVEYXRh"
                            },
                            {
                                "nome": "contrato_arrendamento.pdf",
                                "tipo": "contrato_arrendamento",
                                "url": "https://url-do-arquivo.com.br/arquivo.pdf"
                            }
                        ]
                    }
                ]
            }
        ],
        "limiteFinanciamento": {
            "valor": 123.45,
            "taxa": 0.5,
            "dataExpiracao": "2000-12-31"
        }
    }
}
~~~

### 4.4 Resposta da requisição

Em caso de sucesso, o status code **202** (Aceito) será informado junto com uma mensagem de aceitação dos dados. Isso pois o serviço tem duas frentes: parte assíncrona e parte síncrona.

A parte síncrona é a resposta imediata, devolvida logo no momento da requisição. Essa resposta informa ao usuário do serviço se os dados fornecidos estão corretos ***preliminarmente***. O payload de informações foi criado de forma que seja colhido o mínimo de informações possível para a admissão de um limite de financiamento em nossa estrutura. Nisso, qualquer um dos campos obrigatórios que falhem durante esta fase irá retornar um erro na requisição

Um exemplo dessa resposta de erro:
HTTP Error 400
~~~json
{
  "mensagem":"o CNPJ informado não é válido"
}
~~~
Após a etapa síncrona, o processo é tomado por uma rotina assíncrona, que irá cadastrar os portadores na base de dados do Agricash.
