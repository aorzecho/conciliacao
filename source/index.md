---
title: Integração com serviço de conciliação

language_tabs:
  - xml: Exemplo de conciliação

search: true
---

# Conciliação Stone

A Conciliação Stone é uma ferramenta que disponibiliza diariamente aos estabelecimentos, a demonstração das transações realizadas e suas respectivas informações financeiras, aluguel de POS, ocorridos em D-1. Esta ferramenta permite o acompanhamento desde a captura até o pagamento/desconto de cada uma das transações e lançamentos realizados.

Com a Conciliação Stone, o lojista consegue visualizar de forma bastante clara, o valor líquido e bruto de cada parcela, o valor líquido e bruto de cancelamento parcial e total, quais parcelas da transação foram antecipadas e qual foi o custo da antecipação de cada parcela, chargebacks a serem descontados, etc. 

Tudo isto num mesmo lugar, sem a necessidade de tratar vários arquivos com layouts diferentes. Com a Conciliação Stone, o cliente obtém o arquivo de conciliação diretamente conosco, de maneira segura, através de um Webservice que retorna o arquivo de conciliação.

# O serviço de conciliação

O webservice de conciliação consiste em uma requisição HTTP autenticada, utilizando uma camada segura - SSL/TLS, para a obtenção de um arquivo com todos os eventos ocorridos no dia anterior.

Esse arquivo contém todas as transações que foram capturadas no dia anterior à data de geração do arquivo - *D-1*, ou seja, todas as transações capturadas no dia anterior ao dia da geração do arquivo. **É importante ressaltar que o arquivo contém apenas as transações que foram capturadas e não as que somente foram autorizadas, uma vez que esse tipo de transação não gera movimento financeiro**.

O arquivo adota o modelo Previsão-Liquidação, diferente do que é praticado no mercado. Isso quer dizer que no Arquivo de Conciliação da Stone, estão presentes informações sobre a previsão de pagamento das transações e dos ajustes (agenda); e informações sobre a liquidação das transações e dos ajustes (extrato). **É importante ressaltar que em prol da maior transparência, as transações canceladas não são tratadas como ajustes**.

## Obtendo o arquivo

```
curl \
-H "Authorization: affiliation-key" \
-H "Accept-Encoding: gzip" \
"https://conciliation.stone.com.br/conciliation-file/yyyymmdd"
```

As requisições devem ser enviadas para o serviço de conciliação utilizando o método **GET** para o endpoint `https://conciliation.stone.com.br/conciliation-file/{dataReferencia}`, onde `{dataReferencia}` é a data em que as transações foram capturadas, no formato yyyymmdd.

Por se tratar de informações sigilosas para a empresa, tanto a requisição quanto a resposta trafegam em uma camada segura criptografada e as requisições precisam, necessariamente, estar autenticadas. Essa autenticação consiste no envio de um campo de cabeçalho HTTP `Authorization` contendo a chave de afiliação da loja `AffiliationKey`.

É possível, ainda, que o volume de operações e eventos seja relativamente grande. Para agilizar a transferência do arquivo, é possível solicitá-lo de forma compactada. Caso seja necessário que o arquivo esteja compactado, o sistema conciliador deve incluir o campo de cabeçalho HTTP `Accept-Encoding: gzip` na requisição ao serviço.

### Arquivo de conciliação de exemplo

Você pode fazer o download de um arquivo de conciliação com um exemplo mais completo através do seguinte link: [Stone.123456789.20141103.xml](/attachment/Stone.123456789.20141103.xml)

```xml
<?xml version="1.0" encoding="utf-8"?>
<Conciliation>
    <Header>
        <GenerationDateTime>20141217200910</GenerationDateTime>
        <StoneCode>1234567</StoneCode>
        <MerchantName>Uma empresa de exemplo.</MerchantName>
        <MerchantTaxId>11.111.111/0001-24</MerchantTaxId>
        <LayoutVersion>1</LayoutVersion>
    </Header>
    <FinancialTransactions>
        <Transaction>
            <AuthorizationDateTime>20141216035050</AuthorizationDateTime>
            <OrderReference>abdcefa068354aaa</OrderReference>
            <StoneId>12345678901234</StoneId>
            <AuthorizationCode>T12345</AuthorizationCode>
            <CaptureDateTime>20141216015054</CaptureDateTime>
            <Brand>2</Brand>
            <CardNumber>526778******1234</CardNumber>
            <SalePlanType>282</SalePlanType>
            <ProductType>2</ProductType>
            <NumberOfInstallments>2</NumberOfInstallments>
            <CaptureMethod>Ecommerce</CaptureMethod>
            <SerialNumber />
            <CapturedAmount>2.400000</CapturedAmount>
            <AuthorizedAmount>2.400000</AuthorizedAmount>
            <AuthorizedCurrencyCode>986</AuthorizedCurrencyCode>
            <Installments>
                <Installment>
                    <InstallmentNumber>1</InstallmentNumber>
                    <GrossAmount>20.000000</GrossAmount>
                    <NetAmount>19.560000</NetAmount>
                    <PaymentDate>20150115</PaymentDate>
                </Installment>
                <Installment>
                    <InstallmentNumber>4</InstallmentNumber>
                    <GrossAmount>20.000000</GrossAmount>
                    <NetAmount>19.560000</NetAmount>
                    <PaymentDate>20150415</PaymentDate>
                </Installment>
            </Installments>
        </Transaction>
    </FinancialTransactions>
    <FinancialTransactionsPayments>
        <Transaction>
            <AuthorizationDateTime>20140818145634</AuthorizationDateTime>
            <OrderReference />
            <StoneId>12345678901234</StoneId>
            <AuthorizationCode>081237</AuthorizationCode>
            <CaptureDateTime>20140818115634</CaptureDateTime>
            <BankId />
            <BankBrach />
            <BankAccount />
            <Installments>
                <Installment>
                    <InstallmentNumber>4</InstallmentNumber>
                    <GrossAmount>100.000000</GrossAmount>
                    <NetAmount>97.800000</NetAmount>
                    <PaymentDate>20141216</PaymentDate>
					<Chargeback>
				        <Id>132456</Id>
					    <Amount>100.000000</Amount>
					    <Date>20141216</Date>
					    <PaymentDate>20141217</PaymentDate>
				    </Chargeback>
                </Installment>
            </Installments>
        </Transaction>
    </FinancialTransactionsPayments>
	<FinancialEvents>
	    <Event>
		    <EventId>321</EventId>
			<Description>ALUGUEL DE POS</Description>
			<Type>1</Type>
			<PaymentDate>20140503</PaymentDate>
			<Amount>80.000000</Amount>
		</Event>
	</FinancialEvents>
	<FinancialEventPayments>
	    <Event>
		    <BankId>237</BankId>
			<BankBrach>111</BankBrach>
			<BankAccount>111111-11</BankAccount>
			<EventId>321</EventId>
			<Description>ALUGUEL DE POS</Description>
			<Type>1</Type>
			<PaymentDate>20140503</PaymentDate>
			<Amount>80.000000</Amount>
		</Event>
	</FinancialEventPayments>
    <Trailer>
        <CapturedTransactionsQuantity>16</CapturedTransactionsQuantity>
        <CanceledTransactionsQuantity>0</CanceledTransactionsQuantity>
        <PaidTransactionsQuantity>35</PaidTransactionsQuantity>
    </Trailer>
</Conciliation>
```

### Nó Header

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| GenerationDateTime | Data/Hora | 14 | Data/Hora de geração do arquivo (Formato: aaaammddHHmmss)|
| StoneCode | Num | 9 | Código identificador da loja |
| MerchantName | Alfa | 60 | Razão Social |
| MerchantTaxId | Alfa | 25 | CNPJ da loja |
| LayoutVersion | Num | 3 | Versão do Layout do arquivo |
| Field | Num | 26 | Código identificador do arquivo |

### Nó Transaction

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| AuthorizationDateTime| Data/Hora | 14 | Data/Hora da autorização (Formato: aaaammddHHmmss) |
| OrderReference | Alfa | 128 | Código recebido pelo sistema cliente |
| StoneId | Num | 14 | Identificador único da transação (NSU) |
| CardNumber | Alfa | 19 | Número do cartão (truncado) |
| Brand | Num | 2 | Bandeira do cartão |
| AuthorizationCode | Num | 6 | Código da autorização fornecido pelo emissor. |
| AuthorizedAmount | Float | 20 | Valor autorizado |
| AuthorizedCurrencyCode | Num | 4 | Código da moeda |
| CapturedAmount | Float | 20 | Valor capturado |
| CaptureDateTime | Data/Hora | 14 | Data/Hora da captura (Formato: aaaammddHHmmss) |
| SalePlanType | Alfa | 60 | Plano de venda |
| ProductType | Alfa | 4 | Tipo do produto |
| NumberOfInstallments | Num | 4 | Número de parcelas |
| CaptureMethod | Alfa | 2 | Meio de captura |
| SerialNumber | Alfa | 50 | Número serial do POS (vazio caso o meio de captura seja ecommerce) |
| BankId | Num | 3 | Código do banco |
| BankBranch | Alfa | 6 | Agência bancária |
| BankAccount | Alfa | 11 | Conta bancária |
| Installments | Container | ## | Contém as parcelas da transação. |

### Nó Installment

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| InstallmentNumber | Num | 4 | Número da parcela |
| GrossAmount | Float | 20 | Valor bruto da parcela |
| NetAmount | Float | 20 | Valor líquido da parcela |
| PaymentDate | Data | 8 | Data de pagamento da parcela (Formato: aaaammdd) |
| AdvanceAmount | Float | 20 | Valor cobrado pela antecipação do recebível |
| OriginalPaymentDate | Data | 8 | Data original de pagamento da parcela (Formato: aaaammdd) |

### Nó Trailer

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| CapturedTransactionsQuantity | Num | 10 | Quantidade total de transações capturadas |
| RefundedTransactionsQuantity | Num | 10 | Quantidade total de transações canceladas |
| PaidTransactionsQuantity | Num | 10 | Quantidade total de transações pagas |

# Tabela de referência dos elementos

## ChargeBack

Nó filho de [Installment](#installment) que contém informações sobre o chargeback, como data do pagemento, Id do chargeback, data em que ocorreu o chargeback, etc.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| Id | Num | 10 | Identificador do chargeback |
| Amount | Float | 20 | Valor do chargeback |
| Date | Data | 8 | Data em que ocorreu o chargeback. (Formato: aaaammdd) |
| PaymentDate | Data | 8 | Data em que o chargeback será descontado. (Formato: aaaammdd) |

## ChargeBackRefund

Nó filho de [Installment](#installment) que contém informações sobre a reapresentação do chargeback, como a data em que ocorreu a reapresentação, data de pagamento, etc.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| Id | Num | 10 | Identificador da reapresentação do chargeback |
| Amount | Float | 20 | Valor da reapresentação do chargeback |
| Date | Data | 8 | Data em que ocorreu a reapresentação do chargeback. (Formato: aaaammdd) |
| PaymentDate | Data | 8 | Data em que a reapresentação o chargeback será creditada. (Formato: aaaammdd) |

## Conciliation

| Elemento | Descrição |
| -------- | --------- |
| [Header](#header) | Contém informações referentes à loja e ao arquivo, como razão social, CNPJ, data de geração do arquivo, etc.|
| [FinancialTransactions](#financialtransactions) | Contém as transações capturadas no dia, inclusive as transações que foram capturadas e canceladas no mesmo dia.|
| [FinancialEvents](#financialevents) | Contém os eventos que ocorreram no dia, como (Ajuste MDR, aluguel de POS, etc)|
| [FinancialTransactionPayments](#financialtransactionpayments) | Contém as transações que foram pagas no dia. Contém também os descontos (referentes à cancelamentos) realizados no dia.|
| [FinacialEventPayments](#finacialeventpayments) | Contém os eventos que foram descontos ou creditados no dia, como (Ajuste MDR, aluguel de POS, etc)|
| [Messages](#messages) | Contém as observações referentes às operações do dia.|
| [Trailer](#trailer) | Contém os totalizadores do arquivo, como total de transações capturadas, total líquido de transações pagas, etc.|

## Event

Nó filho de [FinancialEvents](#financialevents) ou de [FinacialEventPayments](#finacialeventpayments), descreve os detalhes do evento ocorrido.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| EventId | Num | 10 | Código identificador do evento |
| Description | Alfa | 60 | Descrição do evento |
| Type | Num | 2 | Tipo do evento |
| PaymentDate | Data | 8 | Data em que o evento será pago. (Formato: aaaammdd) |
| Amount | Float | 20 | Valor do evento |
| BankId | Num | 3 | Código do banco |
| BankBranch | Alfa | 6 | Agência bancária |
| BankAccount | Alfa | 11 | Conta bancária |

## FinancialEvents

Nó filho de [Conciliation](#conciliation) que contém os eventos que ocorreram no dia, como (Ajuste MDR, aluguel de POS, etc)

| Elemento | Descrição |
| -------- | --------- |
| [Event](#event) | Contém informações sobre o evento ocorrido |

## FinacialEventPayments

Nó filho de [Conciliation](#conciliation) que contém os eventos que foram descontos ou creditados no dia, como (Ajuste MDR, aluguel de POS, etc).

| Elemento | Descrição |
| -------- | --------- |
| [Event](#event) | Contém informações sobre o evento ocorrido |

## FinancialTransactions

Nó filho de [Conciliation](#conciliation) que contém as transações capturadas no dia, inclusive as transações que foram capturadas e canceladas no mesmo dia.

| Elemento | Descrição |
| -------- | --------- |
| [Transaction](#transaction) | Contém informações referentes à transação, como valor total da transação, número de parcelas da transação, etc. |

## FinancialTransactionPayments

Nó filho de [Conciliation](#conciliation) que contém as transações que foram pagas no dia. Contém também os descontos, referentes à cancelamentos, realizados no dia.

| Elemento | Descrição |
| -------- | --------- |
| [Transaction](#transaction) | Contém informações referentes à transação, como valor total da transação, número de parcelas da transação, etc. |

## Header

Nó filho de [Conciliation](#conciliation) que contém as informações referentes à loja e ao arquivo, como razão social, CNPJ, data de geração do arquivo, etc.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| GenerationDateTime | Data/Hora | 14 | Data/Hora de geração do arquivo (Formato: aaaammddHHmmss)|
| StoneCode | Num | 9 | Código identificador da loja |
| MerchantName | Alfa | 60 | Razão Social |
| MerchantTaxId | Alfa | 25 | CNPJ da loja |
| LayoutVersion | Num | 3 | Versão do Layout do arquivo |
| Field | Num | 26 | Código identificador do arquivo |


## Installment

Nó filho de [Installments](#installments) que contém informações referentes a uma parcela específica, como valor bruto e valor líquido, número da parcela etc.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| InstallmentNumber | Num | 4 | Número da parcela |
| GrossAmount | Float | 20 | Valor bruto da parcela |
| NetAmount | Float | 20 | Valor líquido da parcela |
| PaymentDate | Data | 8 | Data de pagamento da parcela (Formato: aaaammdd) |
| AdvanceAmount | Float | 20 | Valor cobrado pela antecipação do recebível |
| OriginalPaymentDate | Data | 8 | Data original de pagamento da parcela (Formato: aaaammdd) |
| [ChargeBack](#chargeback) | Container | ## | Contém informações sobre o chargeback, como data do pagemento, Id do chargeback, data em que ocorreu o chargeback, etc. |
| [ChargeBackRefund](#chargebackrefund) | Container | ## | Contém informações sobre a reapresentação do chargeback, como a data em que ocorreu a reapresentação, data de pagamento, etc. |

## Installments

Nó filho de [Transaction](#transaction) que contém as parcelas da transação.

| Elemento | Descrição |
| -------- | --------- |
| [Installment](#installment) | Contém informações referentes a uma parcela específica, como valor bruto e valor líquido, número da parcela etc. |

## Message

Nó filho de [Messages](#messages) que contém as observações referentes às operações do dia.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| MessageId | Num | 9 | Código identificador do mensagem |
| Value | Alfa | 200 | Descrição da mensagem |
| Type | Num | 4 | Tipo da mensagem |

## Messages

Nó filho de [Conciliation](#conciliation) que contém as observações referentes às operações do dia.

| Elemento | Descrição |
| -------- | --------- |
| [Message](#message) | Contém a informação referente à observação referente às operações do dia. |

## Refund

Nó filho de [Transaction](#transaction) que contém informações referentes ao cancelamento, como data de desconto do cancelamento e valor total do cancelamento.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| RefundAmount | Float | 20 | Valor do cancelamento (pode ser igual ou maior que o valor capturado) |
| RefundPaymentDate | Data | 8 | Data em que o cancelamento/estorno será descontado. (Formato: aaaammdd) |
| RefundDate | Data | 8 | Data em que ocorreu o cancelamento. (Formato: aaaammdd) |

## Trailer

Nó filho de [Conciliation](#conciliation) que contém os totalizadores do arquivo, como total de transações capturadas, total líquido de transações pagas, etc.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| CapturedTransactionsQuantity | Num | 10 | Quantidade total de transações capturadas |
| RefundedTransactionsQuantity | Num | 10 | Quantidade total de transações canceladas |
| PaidTransactionsQuantity | Num | 10 | Quantidade total de transações pagas |

## Transaction

Nó filho de [FinancialTransactions](#financialtransactions) ou de [FinancialTransactionPayments](#financialtransactionpayments) que contém as informações referentes à transação, como valor total da transação, número de parcelas da transação, etc.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| AuthorizationDateTime| Data/Hora | 14 | Data/Hora da autorização (Formato: aaaammddHHmmss) |
| OrderReference | Alfa | 128 | Código recebido pelo sistema cliente |
| StoneId | Num | 14 | Identificador único da transação (NSU) |
| CardNumber | Alfa | 19 | Número do cartão (truncado) |
| Brand | Num | 2 | Bandeira do cartão |
| AuthorizationCode | Num | 6 | Código da autorização fornecido pelo emissor. |
| AuthorizedAmount | Float | 20 | Valor autorizado |
| AuthorizedCurrencyCode | Num | 4 | Código da moeda |
| CapturedAmount | Float | 20 | Valor capturado |
| CaptureDateTime | Data/Hora | 14 | Data/Hora da captura (Formato: aaaammddHHmmss) |
| SalePlanType | Alfa | 60 | Plano de venda |
| ProductType | Alfa | 4 | Tipo do produto |
| NumberOfInstallments | Num | 4 | Número de parcelas |
| CaptureMethod | Alfa | 2 | Meio de captura |
| SerialNumber | Alfa | 50 | Número serial do POS (vazio caso o meio de captura seja ecommerce) |
| BankId | Num | 3 | Código do banco |
| BankBranch | Alfa | 6 | Agência bancária |
| BankAccount | Alfa | 11 | Conta bancária |
| [Installments](#installments) | Container | ## | Contém as parcelas da transação. |
| [Refund](#refund) | Container | ## | Contém informações referentes ao cancelamento, como data de desconto do cancelamento e valor total do cancelamento. |