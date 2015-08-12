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

- layout v1

```
curl \
-H "Authorization: affiliation-key" \
-H "Accept-Encoding: gzip" \
"https://conciliation.stone.com.br/conciliation-file/yyyymmdd"
```

- layout v2

```
curl \
-H "Authorization: affiliation-key" \
-H "Accept-Encoding: gzip" \
"https://conciliation.stone.com.br/conciliation-file/v2/yyyymmdd"
```

As requisições devem ser enviadas para o serviço de conciliação utilizando o método **GET** para o endpoint `https://conciliation.stone.com.br/conciliation-file/{dataReferencia}` ou `https://conciliation.stone.com.br/conciliation-file/v{numeroversao}/{dataReferencia}`, onde `{dataReferencia}` é a data em que as transações foram capturadas, no formato yyyymmdd e {numeroversao} o número da versão do layout desejado.

Por se tratar de informações sigilosas para a empresa, tanto a requisição quanto a resposta trafegam em uma camada segura criptografada e as requisições precisam, necessariamente, estar autenticadas. Essa autenticação consiste no envio de um campo de cabeçalho HTTP `Authorization` contendo a chave de afiliação da loja `AffiliationKey`.



É possível, ainda, que o volume de operações e eventos seja relativamente grande. Para agilizar a transferência do arquivo, é possível solicitá-lo de forma compactada. Caso seja necessário que o arquivo esteja compactado, o sistema conciliador deve incluir o campo de cabeçalho HTTP `Accept-Encoding: gzip` na requisição ao serviço.

# Layout v2

### Header

Nó filho de Conciliation que contém as informações referentes à loja e ao arquivo, como identificação do lojista, data de geração do arquivo, etc.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| GenerationDateTime | Data/Hora | 14 | Data/Hora de geração do arquivo (Formato: aaaammddHHmmss)|
| StoneCode | Num | 9 | Código identificador da loja |
| LayoutVersion | Num | 3 | Versão do Layout do arquivo |
| FileId | Num | 26 | Código identificador do arquivo |
| ReferenceDate |Date|8| Data a que se refere o arquivo|

### FinancialTransactions

Nó filho de Conciliation, container de Transaction, que contém as transações capturadas e ou canceladas ocorridas no dia de referência do arquivo.

| Elemento | Descrição |
| -------- | --------- |
| Transaction | Contém informações referentes à transação, como valor total da transação, número de parcelas da transação, etc. |

### Transaction

Nó filho de FinancialTransactions ou de FinancialTransactionPayments que contém as informações referentes à transação, como valor total da transação, número de parcelas da transação, etc.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| Events| Container | ## | Contém contadores dos eventos da transação no dia de referência do arquivo.|
| AcquirerTransactionKey | Num | 14 | Identificador único da transação (NSU) gerado pela adquirente|
| InitiatorTransactionKey | Alfa | 128 | Código recebido pelo sistema cliente |
| AuthorizationDateTime| Data/Hora | 14 | Data/Hora da autorização (Formato: aaaammddHHmmss) |
| CaptureLocalDateTime | Data/Hora | 14 | Data/Hora da captura (Formato: aaaammddHHmmss) no horario local da adquirente|
| AccountType | Alfa | 4 | Tipo de conta utilizada na transação crédito, débito, etc... |
| InstallmentType | Alfa | 60 | Tipo de parcelamento utilizado na transação lojista/emissor |
| NumberOfInstallments | Num | 4 | Número de parcelas |
| AuthorizedAmount | Float | 20 | Valor autorizado |
| CapturedAmount | Float | 20 | Valor capturado |
| CanceledAmount | Float | 20 | Total cancelado |
| AuthorizationCurrencyCode | Num | 4 | Código da moeda |
| IssuerAuthorizationCode | Num | 6 | Código da autorização fornecido pelo emissor. |
| BrandId | Num | 2 | Identificador da bandeira do cartão |
| CardNumber | Alfa | 19 | Número do cartão (truncado) |
| Poi | Container | ## | Contém dados do Ponto de Interação (terminal) que realizou a transação|
| Cancellations | Container | ## | Contém informações referentes aos cancelamentos, como data de desconto do cancelamento e valor total do cancelamento. |
| Installments| Container | ## | Contém as parcelas da transação. |

### Events

Nó filho de Transaction, contém contadores dos eventos de uma transação no dia de referência do arquivo.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| CancellationCharges | Num | 9 | Número de descontos de cancelamento|
| Cancellations | Num | 9 | Número de cancelamentos|
| Captures | Num | 9 | Número de capturas|
| ChargebackRefunds | Num | 9 | Número de estornos de chargeback |
| Chargebacks | Num | 9 | Número de chargebacks |
| Payments | Num | 9 | Número de pagamentos |

### Poi

Contém dados do Ponto de Interação (terminal) que realizou a transação.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| PoiType | Num | 4 | Tipo do ponto de interação (e-commerce, pos, mobile, etc).|
| SerialNumber | Num | 32 | Número de série do terminal, se existir |

### Cancellations

Contém uma lista de cancelamentos

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| Cancellation | Container | ## | Representa um cancelamento de uma transação |

### Cancellation

Representa um cancelamento de uma transação.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| OperationKey | Alfa | 32 | identificador único da operação de cancelamento|
| CancellationDateTime | Data/Hora | 14 |Data hora do cancelamento (Formato: aaaammddHHmmss)|
| ReturnedAmount | Float | 20 | Valor revertido e devolvido ao portador do cartão |
| Billing | Collection | ## | Lista de cobranças relativa ao cancelamento |

### Billing

Cobrança relativa ao cancelamento.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| ChargedAmount | Float | 20 | Valor de desconto do cancelamento (descontado do lojista)|
| PrevisionChargeDate | Data/Hora | 8 | Data prevista para cobrança (aparece quando o elemento de cobrança está sob o nó de agenda [FinancialTransactions]) (Formato: aaaammdd) |
| ChargeDate | Data/Hora | 8 | Data de cobrança (aparece no nó [FinancialTransactionsAccounts]) (Formato: aaaammdd)|

### Installments

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
|Installment| Container | ## | Representa uma parcela de uma transação. |

## Installment

Representa uma parcela de uma transação.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| InstallmentNumber | Num | 2 | Número da parcela |
| GrossAmount | Float | 20 | Valor bruto da parcela |
| NetAmount | Float | 20 | Valor liquido da parcela |
| PaymentDate | Data/Hora | 8 | Data de pagemento da parcela. (Formato: aaaammdd)|
| PrevisionPaymentDate | Data/Hora | 8 | Previsão da data de pagamento (aparece quando o elemento de cobrança está sob o nó de agenda [FinancialTransactions]) (Formato: aaaammdd)|
| AdvanceRateAmount | Float | 20 | Valor cobrado pela antecipação de recebível |
| AdvancedReceivableOriginalPaymentDate | Data/Hora | 8 | Data original de pagamento do recebível adiantado |
| Chargeback | Container | ## | Contém chargebacks relativos a parcela |
| ChargebackRefund | Container | ## | Contém estornos de chargeback relativos a parcela |
| PaymentId | Num | 9 | Referência do elemento de pagamento ([Payments]) no qual a liquidação dessa parcela foi incluida |

### Chargeback

Nó filho de [Installment] que contém informações sobre o chargeback, como data de desconto, Id do chargeback, data em que ocorreu o chargeback, etc.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| Id | Num | 10 | Identificador do chargeback |
| Amount | Float | 20 | Valor do chargeback |
| Date | Data | 8 | Data em que ocorreu o chargeback. (Formato: aaaammdd) |
| ChargeDate | Data | 8 | Data em que o chargeback será descontado. (Formato: aaaammdd) |
| ReasonCode | Num | 8 | Código de motivo do chargeback informado pelo banco emissor |

### ChargebackRefund

Nó filho de [Installment] que contém informações sobre a reapresentação do chargeback, como a data em que ocorreu a reapresentação, data de pagamento, etc.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| Id | Num | 10 | Identificador da reapresentação do chargeback |
| Amount | Float | 20 | Valor da reapresentação do chargeback |
| Date | Data | 8 | Data em que ocorreu a reapresentação do chargeback. (Formato: aaaammdd) |
| PaymentDate | Data | 8 | Data em que a reapresentação o chargeback será creditada. (Formato: aaaammdd) |
| ReasonCode | Num | 8 | Código de motivo do chargeback informado pelo banco emissor |

### FinancialEvents

Nó filho de Conciliation, previsão de eventos financeiros lançados no dia de referência do arquivo (ajustes financeiros, aluguel de pos, etc...).

| Elemento | Descrição |
| -------- | --------- |
| Event | Contém informações sobre o evento ocorrido |

### Event

Nó filho de [FinancialEvents] ou de [FinacialEventPayments], descreve os detalhes do evento ocorrido.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| EventId | Num | 10 | Código identificador do evento |
| PaymentId | Num | 9 | Referência do elemento de pagamento ([Payments]) no qual a liquidação dessa parcela foi incluida, aparece no nó [FinacialEventPayments]|
| Description | Alfa | 60 | Descrição do evento |
| Type | Num | 2 | Tipo do evento |
| PaymentDate | Data | 8 | Data em que o evento será pago. (Formato: aaaammdd) |
| PrevisionPaymentDate | Data | 8 | Previsão da data de cobrança (aparece sob o nó FinancialEvents) |
| Amount | Float | 20 | Valor do evento |

### FinancialTransactionsAccounts

Nó filho de Conciliation, container de Transaction, que contém as transações pagas ou descontadas no dia referência do arquivo.

| Elemento | Descrição |
| -------- | --------- |
| Transaction | Contém informações referentes à transação, como valor total da transação, número de parcelas da transação, etc. |

### FinancialEventAccounts

Nó filho de Conciliation, eventos financeiros pagos ou descontados no dia de referência do arquivo (ajustes financeiros, aluguel de pos, etc...).

| Elemento | Descrição |
| -------- | --------- |
| Event | Contém informações sobre o evento ocorrido |

### Payments

Nó filho de Conciliation, contém informações dos pagamentos efetuados relativos as transações e eventos financeiros do arquivo.

| Elemento | Descrição |
| -------- | --------- |
| Payment | Pagamento efetuado para o lojista. |

### Payment

Nó filho de Payments, representa um pagamento efetuado para o lojista.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| Id | Num | 9 | Identificador do pagamento. |
| TotalAmount | Float | 20 | Valor total depositado na conta do lojista.|
| FavoredBankAccount | Container | ## | Informações bancarias da conta favorecida pelo pagamento |

### FavoredBankAccount

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| BankCode | Num | 4 | Código do banco favorecido |
| BankBranch | Alfa | 10 | Número da agencia favorecida |
| BankAccountNumber | Num | 12 | Número da conta favorecida |

### Trailer

Contadores e totalizadores do arquivo.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| CapturedTransactionsQuantity | Num | 9 | Número de transações capturadas |
| CanceledTransactionsQuantity | Num | 9 | Número de transações canceladas |
| PaidInstallmentsQuantity | Num | 9 | Número de transações pagas |
| ChargedCancellationsQuantity | Num | 9 | Número de cancelamentos descontados |
| ChargebacksQuantity | Num | 9 | Número de chargebacks |
| ChargebacksRefundQuantity | Num | 9 | Número de estornos de chargeback |
| ChargedChargebacksQuantity | Num | 9 | Número de chargebacks descontados |
| PaidChargebacksRefundQuantity | Num | 9 | Número de estornos de chargebacks pagos |
| PaidEventsQuantity  | Num | 9 | Número de eventos financaeiros pagos |
| ChargedEventsQuantity  | Num | 9 | Número de eventos financaeiros descontados |

# Layout v1


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
		<ReferenceDate>20150607</ReferenceDate>
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
            <CapturedAmount>240.000000</CapturedAmount>
            <AuthorizedAmount>240.000000</AuthorizedAmount>
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
					<ChargeBack>
				        <Id>132456</Id>
					    <Amount>100.000000</Amount>
					    <Date>20141216</Date>
					    <PaymentDate>20141217</PaymentDate>
				    </ChargeBack>
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
| ReferenceDate |Date|8| Data a que se refere o arquivo|

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

## Tabela de referência dos elementos

### ChargeBack

Nó filho de [Installment](#installment) que contém informações sobre o chargeback, como data do pagemento, Id do chargeback, data em que ocorreu o chargeback, etc.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| Id | Num | 10 | Identificador do chargeback |
| Amount | Float | 20 | Valor do chargeback |
| Date | Data | 8 | Data em que ocorreu o chargeback. (Formato: aaaammdd) |
| PaymentDate | Data | 8 | Data em que o chargeback será descontado. (Formato: aaaammdd) |

### ChargeBackRefund

Nó filho de [Installment](#installment) que contém informações sobre a reapresentação do chargeback, como a data em que ocorreu a reapresentação, data de pagamento, etc.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| Id | Num | 10 | Identificador da reapresentação do chargeback |
| Amount | Float | 20 | Valor da reapresentação do chargeback |
| Date | Data | 8 | Data em que ocorreu a reapresentação do chargeback. (Formato: aaaammdd) |
| PaymentDate | Data | 8 | Data em que a reapresentação o chargeback será creditada. (Formato: aaaammdd) |

### Conciliation

| Elemento | Descrição |
| -------- | --------- |
| [Header](#header) | Contém informações referentes à loja e ao arquivo, como razão social, CNPJ, data de geração do arquivo, etc.|
| [FinancialTransactions](#financialtransactions) | Contém as transações capturadas no dia, inclusive as transações que foram capturadas e canceladas no mesmo dia.|
| [FinancialEvents](#financialevents) | Contém os eventos que ocorreram no dia, como (Ajuste MDR, aluguel de POS, etc)|
| [FinancialTransactionPayments](#financialtransactionpayments) | Contém as transações que foram pagas no dia. Contém também os descontos (referentes à cancelamentos) realizados no dia.|
| [FinacialEventPayments](#finacialeventpayments) | Contém os eventos que foram descontos ou creditados no dia, como (Ajuste MDR, aluguel de POS, etc)|
| [Messages](#messages) | Contém as observações referentes às operações do dia.|
| [Trailer](#trailer) | Contém os totalizadores do arquivo, como total de transações capturadas, total líquido de transações pagas, etc.|

### Event

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

### FinancialEvents

Nó filho de [Conciliation](#conciliation) que contém os eventos que ocorreram no dia, como (Ajuste MDR, aluguel de POS, etc)

| Elemento | Descrição |
| -------- | --------- |
| [Event](#event) | Contém informações sobre o evento ocorrido |

### FinacialEventPayments

Nó filho de [Conciliation](#conciliation) que contém os eventos que foram descontos ou creditados no dia, como (Ajuste MDR, aluguel de POS, etc).

| Elemento | Descrição |
| -------- | --------- |
| [Event](#event) | Contém informações sobre o evento ocorrido |

### FinancialTransactions

Nó filho de [Conciliation](#conciliation) que contém as transações capturadas no dia, inclusive as transações que foram capturadas e canceladas no mesmo dia.

| Elemento | Descrição |
| -------- | --------- |
| [Transaction](#transaction) | Contém informações referentes à transação, como valor total da transação, número de parcelas da transação, etc. |

### FinancialTransactionPayments

Nó filho de [Conciliation](#conciliation) que contém as transações que foram pagas no dia. Contém também os descontos, referentes à cancelamentos, realizados no dia.

| Elemento | Descrição |
| -------- | --------- |
| [Transaction](#transaction) | Contém informações referentes à transação, como valor total da transação, número de parcelas da transação, etc. |

### Header

Nó filho de [Conciliation](#conciliation) que contém as informações referentes à loja e ao arquivo, como razão social, CNPJ, data de geração do arquivo, etc.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| GenerationDateTime | Data/Hora | 14 | Data/Hora de geração do arquivo (Formato: aaaammddHHmmss)|
| StoneCode | Num | 9 | Código identificador da loja |
| MerchantName | Alfa | 60 | Razão Social |
| MerchantTaxId | Alfa | 25 | CNPJ da loja |
| LayoutVersion | Num | 3 | Versão do Layout do arquivo |
| Field | Num | 26 | Código identificador do arquivo |


### Installment

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

### Installments

Nó filho de [Transaction](#transaction) que contém as parcelas da transação.

| Elemento | Descrição |
| -------- | --------- |
| [Installment](#installment) | Contém informações referentes a uma parcela específica, como valor bruto e valor líquido, número da parcela etc. |

### Message

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

### Refund

Nó filho de [Transaction](#transaction) que contém informações referentes ao cancelamento, como data de desconto do cancelamento e valor total do cancelamento.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| RefundAmount | Float | 20 | Valor do cancelamento (pode ser igual ou maior que o valor capturado) |
| RefundPaymentDate | Data | 8 | Data em que o cancelamento/estorno será descontado. (Formato: aaaammdd) |
| RefundDate | Data | 8 | Data em que ocorreu o cancelamento. (Formato: aaaammdd) |
| RefundGrossAmount | Float | 20 | Valor bruto do cancelamento|

### Trailer

Nó filho de [Conciliation](#conciliation) que contém os totalizadores do arquivo, como total de transações capturadas, total líquido de transações pagas, etc.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| CapturedTransactionsQuantity | Num | 10 | Quantidade total de transações capturadas |
| RefundedTransactionsQuantity | Num | 10 | Quantidade total de transações canceladas |
| PaidTransactionsQuantity | Num | 10 | Quantidade total de transações pagas |

### Transaction

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

## Apêndice

### CaptureMethod

|Valor|Descrição|
|-----|---------|
|1|POS|
|2|MICRO POS|
|3|TEF|
|4|ECOMMERCE|

### Brand

|Valor|Descrição|
|-----|---------|
|1|Visa|
|2|MasterCard|

### ProductType

|Valor|Descrição|
|-----|---------|
|1|Debit|
|2|Credit|

### SalePlanType

|Valor|Descrição|
|-----|---------|
|1|à vista lojista|
|2|parcelado lojista|

### Eventid

|Valor|Descrição|
|-----|---------|
|22|PosRent (Aluguel de POS)|
|23|InternalTransfer (Transferência Interna)|
|27|FinancialAdjustment (Ajuste financeiro)|
