## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/matheuspalanowski/Teste/edit/master/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/matheuspalanowski/Teste/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.

# Introdução
Olá, tudo bem?

Esse é um resumo sobre como a API da Mundipagg funciona.

## Estrutura da API
A API possui uma estrutura que deve ser entendida para facilitar o desenvolvimento.

Sendo assim precisamos entender alguns conceitos antes de começar a integrar na API.
Primeiro será necessário você saber que a API solicita um autenticação por questões de segurança, sendo assim sempre será necessário enviar os dados de autenticação nas chamadas.

A API da Mundipagg tem por finalidade realizar a gestão de pagamentos e para isso possui uma estrutura de Pedido e cobranças que devemos entender a seguir...

A primeira hierarquia que existe é a de Pedidos, eles são a primeira camada da API.
Todo pedido possui Comprador, Item, Endereços e Cobranças.

Sendo assim a relação:
- Pedido 
  - Item (relação N Itens x 1 Pedido)
  - Comprador (relação 1 Comprador x 1 Pedido)
  - Endereço (relação 1 Endereço x 1 Pedido)
  - Cobrança (relação N Cobranças x 1 Pedido)

Sobre a relação de cobrança é importante saber que cada cobrança corresponde a um pagamento realizado.
Sendo assim a relação:
- Cobrança
  - Comprador (relação 1 Comprador x 1 Cobrança)
  - Fatura (relação 1 fatura x 1 Cobrança)
  - Meio de pagamento (relação 1 Meio de pagamento x 1 Cobrança)

Existe uma relação importante entre a empresa e as lojas que podem utilizar a API.
É importante saber que as requisições realizadas via API são a nível de lojas e não de empresas.
Lembrando que a Mundipagg permite que o cliente realize a gestão de seu negócio, incluindo sua carteira de clientes.

A relação de hierarquia pode ser observada abaixo:
- Account (Empresa)
  - Merchant (Loja) (Relação N Merchants X 1 Account)
    - Customer (Comprador) (Relação N Compradores X 1 Merchant) ou (Relação N Compradores X 1 Account e N Merchant)
      - Card (Cartão) (Relação N Cartões X 1 Comprador)
      - Address (Endereço) (Relação N Endereços X 1 Comprador)

# Autenticação

Autenticação é o método que a API permite que um terceiro se conecte ao sistema.

## Como obter sua Chave de API

Antes de começar, você precisa obter suas chaves de API.
Para isso, siga os seguintes passos:
1 - Acesse este link e crie sua Loja e seu Usuário em nosso Dash,
2 - Após acessar o Dash, navegue até a área de Configurações e resgate suas chaves.

### Tipos de chaves
Nós disponibilizaremos 02 chaves para que você possa realizar testes:
* Exemplo de Chave Secreta de Sandbox: sk_test_tra6ezsW3BtPPXQa
* Exemplo de Chave Pública de Sandbox: pk_test_gaa5xzfz7CfPPZAv

## Autorização Basic Auth
Para se autenticar conosco você deve enviar a Chave de API no cabeçalho Authorization, seguindo o padrão da HTTP Basic Authentication.

~~~Node.js
var fs = require('fs');
const request = require("request");
var body = JSON.parse(fs.readFileSync('body.json', 'utf8'));

var options = {                 
    method: 'POST',             
    uri: 'https://api.mundipagg.com/core/v1/orders',                    
    headers: {               
      'Authorization': 'Basic ' + new Buffer("sk_test_tra6ezsW3BtPPXQa:").toString('base64'),
      'Content-Type': 'application/json'              
    },
    json : body
};    

request(options, function(error, response, body) {  
    console.log(response.body);
});
~~~


## Autorização Bearer Token para Gerenciamento de Wallets
Para o gerenciamento de Wallets você deverá autenticar-se nos servidores da Mundipagg através de um Access Token, que é um token temporário que permitirá o gerenciamento de cartões, pedido, cobranças e muitos outros recursos, sem que dados sensíveis trafeguem em seus servidores. Saiba mais sobre Wallets.

Para se autenticar conosco através do Access Token você deve enviá-lo no cabeçalho Authorization, seguindo o padrão da HTTP Bearer Token.

~~~Node.js
var fs = require('fs');
const request = require("request");
var body = JSON.parse(fs.readFileSync('body_card.json', 'utf8'));

var access_token = '4wdOK29W6RvyYbgPv63VPMYopk4Jy752dqbQxmm3qOgKy8dbexRvXQaGDZE0AYws88s7z64lWJnM1P7Vk9jYL2No5wp89jg1leNnG0WwXZOKDaLER3MXq07jQLokJn5m'

var customer_id = 'cus_9El4qnTEKFKQoV7r'

console.log(body);

var options = {                 
    method: 'POST',             
    uri: 'https://api.mundipagg.com/core/v1//customers/'+customer_id+'/cards',                    
    headers: {               
      'Authorization': 'Bearer ' + access_token,
      'Content-Type': 'application/json'              
    },
    json : body
};    

  request(options, function(error, response, body) {  
    console.log(options.headers['Authorization']);
    console.log(response.body);
});
~~~

### Access Token
O objeto access_token é um token temporário que pode ser utilizado para autenticação nos servidores da Mundipagg. Com ele é possível gerenciar os cartões da Wallet do cliente sem trafegar os dados do cartão em seus servidores. Saiba mais sobre Access Token acessando nossa documentação. Este objeto possui os seguintes atributos:
|Atributos|	Tipo|	Descrição|
---|---|---
|id|string|Código do Access Token. Formato: at_XXXXXXXXXXXXXXXX
|code|string|Access Token.
|status|enum|Status do Access Token. Valores possíveis: active ou deleted.
|created_at|datetime|Data de criação do Access Token.
|customer|object|Dados do cliente. Saiba mais sobre clientes
|expires_in|integer|Tempo, em minutos, para expiração do token.
|deleted_at|datetime|Data de exclusão do Access Token.

#### Métodos permitidos com AccessToken:
Método|Descrição
---|---
GetCustomer| Obter um cliente


### Criar Access Token
Este recurso permite a criação de um access_token associado a um cliente através de seu customer_id.

>Endpoint: https://api.mundipagg.com/core/v1/customers/customer_id/access-tokens

~~~Json
{
  "expires_in": 30
}
~~~
~~~C#
using MundiAPI.PCL;
using MundiAPI.PCL.Models;
using System.Web.Mvc;

namespace MundipaggTest.Controllers {

    public class HomeController : Controller {

    public ActionResult Index() 

    // Secret key fornecida pela Mundipagg
    string basicAuthUserName = "sk_test_4tdVXpseumRmqbo";
    // Senha em branco. Passando apenas a secret key
    string basicAuthPassword = "";

    var client = new MundiAPIClient(basicAuthUserName, basicAuthPassword);

    string customerId = "cus_6l5dMWZ0hkHZ4XnE";

    var request = new CreateAccessTokenRequest {
        ExpiresIn = 1
    };

    var response = client.Customers.CreateAccessToken(customerId, request);

    return View();
    }
}
}
~~~

#### Campos obrigatórios:
Campo| Descrição
---|---
customer_id| Código do cliente. Formato: cus_XXXXXXXXXXXXXXXX.

### Obter Access Token
Através dos identificadores do access token (access_token_id) e de seu cliente associado (customer_id) é possível obter os dados do token gerado.

>Endpoint: https://api.mundipagg.com/core/v1/customers/customer_id/access-tokens/access_token_id

~~~C#
using MundiAPI.PCL;
using System.Web.Mvc;

namespace MundipaggTest.Controllers {

    public class HomeController : Controller {

    public ActionResult Index() {

    // Secret key fornecida pela Mundipagg
    string basicAuthUserName = "sk_test_4tdVXpseumRmqbo";
    // Senha em branco. Passando apenas a secret key
    string basicAuthPassword = "";

    var client = new MundiAPIClient(basicAuthUserName, basicAuthPassword);

    string customerId = "cus_6l5dMWZ0hkHZ4XnE";
    string accessTokenId = "at_Z6AwO09CKvcZQXln";

    var response = client.Customers.GetAccessToken(customerId, accessTokenId);

    return View();
    }
}
}
~~~

#### Campos obrigatórios:
Campo| Descrição
---|---
customer_id| Código do cliente. Formato: cus_XXXXXXXXXXXXXXXX.
access_token_id| Código do Access Token. Formato: at_XXXXXXXXXXXXXXXX.

### Cancelar Access Token
Com o verbo HTTP DELETE, através dos identificadores do access token (access_token_id) e de seu cliente associado (customer_id) é possível excluir um token anteriormente gerado.

>Endpoint: https://api.mundipagg.com/core/v1/customers/customer_id/access-tokens/access_token_id

~~~C#
using MundiAPI.PCL;
using System.Web.Mvc;

namespace MundipaggTest.Controllers {

    public class HomeController : Controller {

    public ActionResult Index() {

    // Secret key fornecida pela Mundipagg
    string basicAuthUserName = "sk_test_4tdVXpseumRmqbo";
    // Senha em branco. Passando apenas a secret key
    string basicAuthPassword = "";

    var client = new MundiAPIClient(basicAuthUserName, basicAuthPassword);

    string customerId = "cus_6l5dMWZ0hkHZ4XnE";
    string accessTokenId = "at_Z6AwO09CKvcZQXln";

    var response = client.Customers.DeleteAccessToken(customerId, accessTokenId);

    return View();
    }
}
}
~~~

#### Campos obrigatórios:
Campo| Descrição
---|---
customer_id| Código do cliente. Formato: cus_XXXXXXXXXXXXXXXX.
access_token_id| Código do Access Token. Formato: at_XXXXXXXXXXXXXXXX.

# Wallet
A API Mundipagg fornece o recurso Wallet, que permite o armazenamento de cartões associados a um comprador, viabilizando a realização de compras com um clique, por exemplo. Saiba mais sobre Wallet vs. Carteira de Clientes.

Relação:
- Acount
  - Comprador (Relação N Compradores X N Accounts)
    - Cartão (Relação N Cartões X 1 Comprador)
    - Endereço (Relação N Endereços X 1 Comprador)

## Comprador
O objeto customer possibilita a criação e gerenciamento de uma Carteira de Clientes. Um customer possui os seguintes atributos:


### Criar um comprador 

>Endpoint: https://api.mundipagg.com/core/v1/customers

~~~Json
{
    "name": "Tony Stark",
    "email": "tonystarkk@avengers.com",
    "code": "MY_CUSTOMER_001",
    "document": "123456789",
    "type": "individual",
    "gender": "male",
    "address": {
    	"line_1": "375, Av. General Justo, Centro",
    	"line_2": "8º andar",
    	"zip_code": "20021130",
    	"city": "Rio de Janeiro",
    	"state": "RJ",
    	"country": "BR"
    },
    "birthdate": "05/03/1984",    
    "phones": {
      "home_phone": {
        "country_code": "55",
        "area_code": "21",
        "number": "000000000"
      },
      "mobile_phone": {
        "country_code": "55",
        "area_code": "21",
        "number": "000000000"
      }
    },
    "metadata": {
      "company": "Avengers"
    }
}
~~~

#### Campos obrigatórios:
Campo| Descrição
---|---
name|
type|

>email é a chave forte

### Obter um comprador 

>Endpoint: https://api.mundipagg.com/core/v1/customers/customer_id

~~~C#
using MundiAPI.PCL;
using System.Web.Mvc;

namespace MundipaggTest.Controllers {

    public class HomeController : Controller {

public ActionResult Index() {

    // Secret key fornecida pela Mundipagg
    string basicAuthUserName = "sk_test_4tdVXpseumRmqbo";
    // Senha em branco. Passando apenas a secret key
    string basicAuthPassword = "";

    var client = new MundiAPIClient(basicAuthUserName, basicAuthPassword);
    string customerId = "cus_6l5dMWZ0hkHZ4XnE";

    var response = client.Customers.GetCustomer(customerId);

    return View();
}
}
}
~~~

#### Campos obrigatórios:
Campo| Descrição
---|---
customer_id|

> Access Token permite acessa esse método

### Editar um comprador 
>Endpoint: https://api.mundipagg.com/core/v1/customers/customer_id

~~~Json
{
  "name": "Peter Parker",
  "email": "parker@avengers.com",
  "gender": "male"
}
~~~

#### Campos passíveis de edição:
Campo| Descrição
---|---
name|
email|
phones|
document|
code|
type|
gender|
address|
birthdate|
metadata|

>email é a chave forte

## Cartão
### Criar um cartão
### Obter um cartão
### Editar um cartão
### Excluir um cartão
### Criar CardToken

## Endereço
### Criar um Endereço
### Obter um Endereço
### Editar um Endereço
### Excluir um Endereço

# Pagamentos
Para realizar pagamentos utilizamos a API da Mundipagg é necessário criar um pedido.

# Pedido
Para realizar pedidos de cartão de crédito ou boleto, você deve criar um objeto order. Este objeto tem os seguintes atributos:

## Criar um Pedido 

Para ciar um Pedido alguns dados são necessários, são eles:
* Comprador
* Itens
* Pagamento

Vale ressaltar que o valor do Pedido é computado com base nos itens infromados

> Endpoint: https://api.mundipagg.com/core/v1/orders/

~~~C#
   // Neste exemplo estamos utilizando A sdk C# MundiAPI.PCL
     // Secret key fornecida pela Mundipagg
     string basicAuthUserName = "sk_test_4tdVXpseumRmqbo";
     // Senha em branco. Passando apenas a secret key
     string basicAuthPassword = "";
     var client = new MundiAPIClient (basicAuthUserName, basicAuthPassword);

     var items = new List<CreateOrderItemRequest> {
         new CreateOrderItemRequest {
         Amount = 2990,
         Description = "Chaveiro do Tesseract",
         Quantity = 1
         }
     };
     var customer = new CreateCustomerRequest {
         Name = "Tony Stark",
         Email = "tonystark@avengers.com",
     };

     var payments = new List<CreatePaymentRequest> {
         new CreatePaymentRequest {
         PaymentMethod = "credit_card",
         CreditCard = new CreateCreditCardPaymentRequest {
         Card = new CreateCardRequest {
         HolderName = "Tony Stark",
         Number = "342793631858229",
         ExpMonth = 1,
         ExpYear = 18,
         Cvv = "3531",
         }
         }
         }
     };
     var request = new CreateOrderRequest {
         Items = items,
         Customer = customer,
         Payments = payments
     };

     var response = client.Orders.CreateOrder (request);
~~~

Ao criar um pedido uma cobrança é automaticamente criada, vinculando os dados do meio de pagamento informado.

## Status de um Pedido
É possível criar um pedido aberto ou fechado.

### Pedido Aberto
Permite a inclusão de novas cobranças e alteração de status, possibilitando total gestão por parte do cliente.

### Incluir cobrança no pedido
Enquanto um pedido estiver aberto, é possível adicionar novas cobranças utilizando o order_id na criação de uma cobrança.
Para incluir cobranças em um pedido aberto alguns dados são necessários, são eles:
* order_id
* amount
* payment

### Incluir Item no pedido
Com a criação de um pedido aberto, é possível que itens sejam gerenciados. O objeto item do pedido tem os seguintes atributos:

Para incluir um item em um pedido aberto alguns dados são necessários, são eles:
* order_id
* code - No caso de serviços de análise de fraude
* description
* quantity

> Endpoint: https://api.mundipagg.com/core/v1/orders/order_id/items

~~~C#
// Neste exemplo estamos utilizando A sdk C# MundiAPI.PCL
// Secret key fornecida pela Mundipagg
string basicAuthUserName = "sk_test_4tdVXpseumRmqbo";
// Senha em branco. Passando apenas a secret key
string basicAuthPassword = "";
var client = new MundiAPIClient(basicAuthUserName, basicAuthPassword);

var item = new CreateOrderItemRequest {
    Amount = 10,
    Category = "Mark 1 Gloves",
    Description = "Gloves",
    Quantity = 1,
};
var request = client.Orders.CreateOrderItem("or_JQ35ED8izte8BN9k", item);
~~~

### Editar Item no pedido
### Deletar um Item no pedido ou todos
### Consultar um item do pedido

### Pedido Fechado

## Meios de pagamento de um pedido
### Cartão de crédito
#### Criar um pedido com cartão de crédito
### Cartão de débito
#### Criar um pedido com cartão de débito
### Cartão Voucher
#### Criar um pedido com Voucher
### Cartão de Private Label
#### Criar um pedido com cartão Private Label
### Boleto
#### Criar um pedido com boleto
### Transferência
#### Criar um pedido com transferência
### SafetyPay
#### Criar um pedido com SafetyPay
### ApplePay
#### Criar um pedido com ApplePay
### GooglePay
#### Criar um pedido com GooglePay
### SamsungPay
#### Criar um pedido com SamsungPay
### Cash
#### Criar um pedido com Cash
### Checkout
Atributos do meio de pagamento checkout:
* expires_in
* billing_address_editable
* customer_editable
* accepted_payment_methods
* success_url
* accepted_multi_payment_methods
* default_payment_method
* Payment

Sendo assim é necessário informar para a criação de um checkout quando ele deve expirar, se os dados de endereço e cliente serão editados, quais os tipos de pagamentos o checkout irá aceitar, se irá aceitar multimeios, qual URL de sucesso ele deve redirecionar o cliente e por fim os dados do meios de pagamento disponivél.

Vale ressaltar que o checkout só pode ser utilizado a nivel de pedido, visto que ele só irá criar uma cobrança quando um meio de pagamento for selecionado pelo consumidor.
Esse é um meio de pagamento/funcionalidade da api que depende da atuação do consumidor e por isso ele aguarda para a criação de cobrança.

Alguns pontos de observação:
Todo pedido criado com checkout é aberto, pois é necessário adicionar cobranças a esse pedido.
Se um checkout expirar o status do pedido será alterado para Canceled, porém não será fechado, permitindo a inclusão de cobranças ainda.

#### Criar um pedido com checkout
#### Criar um pedido com checkout e multimeios

## Pedidos com Multimeios
Neste caso você inclui mais de um objeto na coleção de payments. Desta maneira você pode realizar pedido com diversos cartões ou combinando diferentes meios de pagamento.

Dados obrigatórios:
* itens
* payments
  * Amount

> Como um pedido com multimeios está definindo mais de uma forma de pagamento é necessário que o cliente informe quando será cobrado em cada forma de pagamento

Cada Payment criado irá resultar em uma cobrança diferente relacionada ao pedido original.

## Pedidos com multicompradores
Neste caso você pode incluir um cliente (customer) dentro do nó de pagamento (payment). Caso um nó payment não possua um nó customer dentro, assumiremos que o pagador dele é o mesmo customer do pedido.

Dados obrigatórios:
* itens
* payments
  * Amount
  * Customer


> Como um pedido com multicompradores está definindo mais de uma forma de pagamento é necessário que o cliente informe quando será cobrado em cada forma de pagamento

Cada Payment criado irá resultar em uma cobrança diferente relacionada ao pedido original.

## Obter Pedido
## Editar Pedido
## Fechar Pedido

# Cobrança
Para realizar cobranças de cartão de crédito ou boleto, você deve criar um objeto charge. Este objeto tem os seguintes atributos:

### Capturar uma cobrança
### Cancelar uma cobrança
### Consultar uma cobrança
### Retentar uma cobrança
### Confirmar uma cobrança
### Editar uma cobrança
### Editar cartão de uma cobrança
Esse recurso só pode ser chamado quando o cartão a ser editado teve a transação não autorizada.
### Editar data de vencimento
### Editar meio de pagamento

# Assinatura
O objeto subscription é uma recorrência em si, que possibilita a cobrança do clientes com intervalos pré determinados, sem que seja necessário criar essa regra de negócio do seu lado. O objeto subscription contêm os seguintes atributos:

Atributos de uma assinatura:
* meio de pagamento
* Período
* Incidência
* Valor
* Modelo de cobrança
* Comprador
* Itens
* Desconto
* Incremento
* Plano

## Planos
### Criar Assinatura com planos
### item do plano

## item assinatura
### Usos de uma assinatura
## Desconto
## Incremento
## Criar Assinatura avulsa
## Fatura
## Ciclos


# MarketPlace

# Retornos da aplicação
A nossa API valida cada um dos campos enviados na requisição antes de prosseguir com a criação, consulta ou gerenciamento dos pedidos, transações e recursos.

Utilizamos os códigos de resposta convencionais do HTTP para indicar o sucesso ou a falha de uma requisição. Sendo assim, códigos 2xx indicam sucesso, 4xx indicam erros por algum dado informado incorretamente (por exemplo, algum campo obrigatório não enviado ou um cartão sem data de validade) e 5xx indicando erros nos servidores da Mundipagg.

Tabela dos HTTP Status Code:
|Código|	Status|	Definição| Cenário |
---|---|---|---
200|OK|Sucesso| 
400|Bad Request|Requisição inválida| 
401|Unauthorized|Chave de API inválida| 
404|Not Found|O recurso solicitado não existe| 
412|Precondition Failed|Parâmetros válidos mas a requisição falhou| 
422|Unprocessable Entity|Parâmetros inválidos| 
500|Internal Server Error|Ocorreu um erro interno| 
