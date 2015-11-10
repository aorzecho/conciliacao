---
title: Conciliação Stone

search: true
---

# Conciliação Stone

A Conciliação Stone é uma ferramenta que disponibiliza diariamente aos estabelecimentos, a demonstração das transações realizadas e suas respectivas informações financeiras, aluguel de POS, ocorridos no dia referenciado. Esta ferramenta permite o acompanhamento desde a captura até o pagamento/desconto de cada uma das transações e lançamentos realizados.

Com a Conciliação Stone, o lojista consegue visualizar de forma bastante clara, o valor líquido e bruto de cada parcela, o valor líquido e bruto de cancelamento parcial e total, quais parcelas da transação foram antecipadas e qual foi o custo da antecipação de cada parcela, chargebacks a serem descontados, etc.

Tudo isto num mesmo lugar, sem a necessidade de tratar vários arquivos com layouts diferentes. Com a Conciliação Stone, o cliente obtém o arquivo de conciliação diretamente conosco, de maneira segura, através de um Webservice que retorna o arquivo de conciliação.

# O serviço de conciliação

O webservice de conciliação consiste em uma requisição HTTP autenticada, utilizando uma camada segura - SSL/TLS, para a obtenção de um arquivo com todos os eventos ocorridos no dia requisitado.

Esse arquivo contém todas as transações que foram capturadas no dia requisitado, ou seja, todas as transações capturadas no dia para o qual a conciliação está sendo gerada. **É importante ressaltar que o arquivo contém apenas as transações que foram capturadas e não as que somente foram autorizadas, uma vez que esse tipo de transação não gera movimento financeiro**.

O arquivo adota o modelo Previsão-Liquidação, diferente do que é praticado no mercado. Isso quer dizer que no Arquivo de Conciliação da Stone, estão presentes informações sobre a previsão de pagamento das transações e dos ajustes (agenda); e informações sobre a liquidação das transações e dos ajustes (extrato). **É importante ressaltar que em prol da maior transparência, as transações canceladas não são tratadas como ajustes**.

## Obtendo o arquivo

- Layout v1

```shell
curl \
-H "Authorization: affiliation-key" \
-H "Accept-Encoding: gzip" \
"https://conciliation.stone.com.br/conciliation-file/yyyyMMdd"
```

- Layout v2

```shell
curl \
-H "Authorization: affiliation-key" \
-H "Accept-Encoding: gzip" \
"https://conciliation.stone.com.br/conciliation-file/v2/yyyyMMdd"
```


As requisições devem ser enviadas para o serviço de conciliação utilizando o método **GET** para o endpoint `https://conciliation.stone.com.br/conciliation-file/{dataReferencia}` ou `https://conciliation.stone.com.br/conciliation-file/v{numeroversao}/{dataReferencia}`, onde `{dataReferencia}` é a data em que as transações foram capturadas, no formato yyyyMMdd e {numeroversao} o número da versão do layout desejado.

Por se tratar de informações sigilosas para a empresa, tanto a requisição quanto a resposta trafegam em uma camada segura criptografada e as requisições precisam, necessariamente, estar autenticadas. Essa autenticação consiste no envio de um campo de cabeçalho HTTP `Authorization` contendo a chave de afiliação da loja `AffiliationKey`.



É possível, ainda, que o volume de operações e eventos seja relativamente grande. Para agilizar a transferência do arquivo, é possível solicitá-lo de forma compactada. Caso seja necessário que o arquivo esteja compactado, o sistema conciliador deve incluir o campo de cabeçalho HTTP `Accept-Encoding: gzip` na requisição ao serviço.

<!--
# Layout V1

> Nota:

>A versão 1 do arquivo de conciliação não será mais atualizada, o serviço ainda pode ser consumido mas encorajamos que adapte sua consulta à versão 2, já que as novas <em>features</em> serão aplicadas apenas ao Layout v2 .

## O Arquivo
### Conciliation

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| Header   | Container | ###### | Contém as informações do lojista e do arquivo |
| FinancialTransactions | Container | ###### | Contém as transações que aconteceram com o lojista no dia requisitado |
| FinancialEvents | Container | ###### | Contém os eventos financeiros lançados para o lojista no dia requisitado |
| FinancialTransactionsPayments | Container | ###### | Contém as transações que foram pagas/cobradas ao lojista no dia requisitado |
| FinancialEventPayments | Container | ###### | Contém os eventos que foram pagos/cobrados ao lojista no dia requisistado |
| Trailer | Container | ###### | Contém os contadores do arquivo |

### Header

Nó filho de Conciliation que contém as informações referentes à loja e ao arquivo, como identificação do lojista, data de geração do arquivo, etc.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| GenerationDateTime | Datetime | 14 | Datetime de geração do arquivo (Formato: aaaammddHHmmss)|
| StoneCode | Num | 9 | Código identificador da loja |
| LayoutVersion | Num | 3 | Versão do Layout do arquivo |
| FileId | Num | 26 | Código identificador do arquivo |
| ReferenceDate |Date|8| Data a que se refere o arquivo|

### FinancialTransactions

Nó filho de Conciliation, container de Transaction, que contém as transações capturadas e ou canceladas ocorridas no dia de referência do arquivo.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| Transaction | Container | ###### |  Contém informações referentes à transação, como valor total da transação, número de parcelas da transação, etc. |

### Transaction

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| AuthorizationDateTime| Datetime | 14 | Datetime da autorização (Formato: aaaammddHHmmss) |
| OrderReference | Alfa | 128 | Código recebido pelo sistema cliente |
| StoneId | Num | 14 | Identificador único da transação (NSU) |
| AuthorizationCode | Num | 6 | Código da autorização fornecido pelo emissor. |
| CaptureDateTime | Datetime | 14 | Datetime da captura (Formato: aaaammddHHmmss) |
| Brand | Num | 2 | Bandeira do cartão |
| CardNumber | Alfa | 19 | Número do cartão (truncado) |
| SalePlanType | Alfa | 60 | Plano de venda |
| ProductType | Alfa | 4 | Tipo do produto |
| NumberOfInstallments | Num | 4 | Número de parcelas |
| CaptureMethod | Num | 4 | Meio de captura |
| SerialNumber | Alfa | 50 | Número serial do POS (vazio caso o meio de captura seja ecommerce) |
| CapturedAmount | Float | 20 | Valor capturado |
| AuthorizedAmount | Float | 20 | Valor autorizado |
| AuthorizedCurrencyCode | Num | 4 | Código da moeda |
| Refund | Container | ###### | Elemento que representa um cancelamento dessa transação* |
| Installments | Container | ###### | Contém as parcelas da transação. |

>\* Campo que só aparece se a transação tiver tido cancelamento

### Refund

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| RefundGrossAmount | Float | 20 | Valor bruto do cancelamento |
| RefundAmount | Float | 20 | Valor cobrado do lojista pelo cancelamento |
| RefundPaymentDate | Date | 14 | Dia em que será/foi cobrado o cancelamento Formato:(aaaaMMdd)|
| RefundDate | Date | 14 | Dia em que ocorreu o cancelamento |

### Installments

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
|Installment| Container | ###### | Representa uma parcela de uma transação. |

### <ir id = 'installment'>Installment</ir>

Representa uma parcela de uma transação.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| InstallmentNumber | Num | 2 | Número da parcela |
| GrossAmount | Float | 20 | Valor bruto da parcela |
| NetAmount | Float | 20 | Valor liquido da parcela |
| AdvanceRateAmount | Float | 20 | Valor cobrado pela antecipação de recebível* |
| AdvancedReceivableOriginalPaymentDate | Datetime | 8 | Data original de pagamento do recebível adiantado* |
| Chargeback | Container | ###### | Contém chargebacks relativos a parcela** |
| ChargebackRefund | Container | ###### | Contém estornos de chargeback relativos a parcela\*** |
|PaymentDate | Datetime | 8 | Dia em que a transação será/foi paga ao lojista  |
| MdrNetAmount | Float | 20 | Valor líquido sem a taxa de antecipação \* |

> \* Campo que aparece apenas se houve antecipação da parcela
>
> \*\* Campo que aparece somente se houve chargeback da parcela
>
> \*\*\* Campo que aparece apenas se houve reapresentação de chargeback

### ChargeBack

Nó filho de Installment que contém informações sobre o chargeback, como data do pagemento, Id do chargeback, data em que ocorreu o chargeback, etc.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| Id | Num | 10 | Identificador do chargeback |
| Amount | Float | 20 | Valor do chargeback |
| Date | Data | 8 | Data em que ocorreu o chargeback. (Formato: aaaammdd) |
| PaymentDate | Data | 8 | Data em que o chargeback será descontado. (Formato: aaaammdd) |
| ReasonCode | Num | 8 | Código do motivo do chargeback informado pelo banco emissor |

### ChargeBackRefund

Nó filho de Installment que contém informações sobre a reapresentação do chargeback, como a data em que ocorreu a reapresentação, data de pagamento, etc.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| Id | Num | 10 | Identificador da reapresentação do chargeback |
| Amount | Float | 20 | Valor da reapresentação do chargeback |
| Date | Data | 8 | Data em que ocorreu a reapresentação do chargeback. (Formato: aaaammdd) |
| PaymentDate | Data | 8 | Data em que a reapresentação o chargeback será creditada. (Formato: aaaammdd) |
| ReasonCode | Num | 8 | Código do motivo do chargeback informado pelo banco emissor |

### FinancialTransactionPayments

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| Transaction | Container | ###### |  Contém informações referentes à transação, como valor total da transação, número de parcelas da transação, etc. |

### Transaction

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| AuthorizationDateTime| Datetime | 14 | Datetime da autorização (Formato: aaaammddHHmmss) |
| OrderReference | Alfa | 128 | Código recebido pelo sistema cliente |
| StoneId | Num | 14 | Identificador único da transação (NSU) |
| AuthorizationCode | Num | 6 | Código da autorização fornecido pelo emissor. |
| CaptureDateTime | Datetime | 14 | Datetime da captura (Formato: aaaammddHHmmss) |
| Brand | Num | 2 | Bandeira do cartão |
| CardNumber | Alfa | 19 | Número do cartão (truncado) |
| SalePlanType | Alfa | 60 | Plano de venda |
| ProductType | Alfa | 4 | Tipo do produto |
| NumberOfInstallments | Num | 4 | Número de parcelas |
| CaptureMethod | Num | 4 | Meio de captura |
| SerialNumber | Alfa | 50 | Número serial do POS (vazio caso o meio de captura seja ecommerce) |
| CapturedAmount | Float | 20 | Valor capturado |
| AuthorizedAmount | Float | 20 | Valor autorizado |
| AuthorizedCurrencyCode | Num | 4 | Código da moeda |
| BankId | Num | 4 | Código de identificação do banco do beneficiado |
| BankBranch | Num | 4 | Agência Bancária do beneficiado |
| BankAccount | Num | 12 | Conta bancária do beneficiado |  
| Refund | Container | ###### | Elemento que representa um cancelamento dessa transação* |
| Installments | Container | ###### | Contém as parcelas da transação. |

>\* Campo que só aparece se a transação tiver tido cancelamento

### Installments

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
|Installment| Container | ###### | Representa uma parcela de uma transação. |

### Installment

Representa uma parcela de uma transação. Semelhante ao elemento [Installment](#installment) descrito acima -->

# Layout V2

Diferente do layout 1 onde a apresentação é dinâmica e é possível observar o ciclo de vida de uma transação coletando o arquivo em diferentes momentos,
por exemplo uma transação realizada no dia 24-08 e cancelada no dia 19-10


  + Pedido de arquivo para o dia 24-08-2015 (Layout 1):

    ``` xml
    <Conciliation>
      <Header>
        <GenerationDateTime>20151020150721</GenerationDateTime>
        <StoneCode>123456789</StoneCode>
        <LayoutVersion>1</LayoutVersion>
        <FileId>000111</FileId>
        <ReferenceDate>20150824</ReferenceDate>
      </Header>
      <FinancialTransactions>
        <Transaction>
          <AuthorizationDateTime>20150824182825</AuthorizationDateTime>
          <OrderReference>1157795</OrderReference>
          <StoneId>33650014557171</StoneId>
          <AuthorizationCode>022535</AuthorizationCode>
          <CaptureDateTime>20150824152839</CaptureDateTime>
          <Brand>1</Brand>
          <CardNumber>123456******1122</CardNumber>
          <SalePlanType>2</SalePlanType>
          <ProductType>2</ProductType>
          <NumberOfInstallments>2</NumberOfInstallments>
          <SerialNumber/>
          <CapturedAmount>130.620000</CapturedAmount>
          <AuthorizedAmount>130.620000</AuthorizedAmount>
          <AuthorizedCurrencyCode>986</AuthorizedCurrencyCode>
          <Refund>
            <RefundGrossAmount>130.620000</RefundGrossAmount>
            <RefundAmount>127.628802</RefundAmount>
            <RefundPaymentDate>20150923</RefundPaymentDate>
            <RefundDate>20151019</RefundDate>
          </Refund>
          <Installments>
            <Installment>
              <InstallmentNumber>1</InstallmentNumber>
              <GrossAmount>65.310000</GrossAmount>
              <NetAmount>63.814401</NetAmount>
              <PaymentDate>20150923</PaymentDate>
            </Installment>
            <Installment>
              <InstallmentNumber>2</InstallmentNumber>
              <GrossAmount>65.310000</GrossAmount>
              <NetAmount>63.814401</NetAmount>
              <PaymentDate>20151020</PaymentDate>
            </Installment>
          </Installments>
        </Transaction>
        <Transaction>
          ...
    ```

  + Pedido do arquivo para o dia 19-10-2015 (Layout 1):

    ```xml
    <Conciliation>
      <Header>
        <GenerationDateTime>20151020150410</GenerationDateTime>
        <StoneCode>123456789</StoneCode>
        <LayoutVersion>1</LayoutVersion>
        <FileId>110011</FileId>
        <ReferenceDate>20151019</ReferenceDate>
      </Header>
      <FinancialTransactions>
        <Transaction>
          <AuthorizationDateTime>20150824182825</AuthorizationDateTime>
          <OrderReference>1157795</OrderReference>
          <StoneId>33650014557171</StoneId>
          <AuthorizationCode>022535</AuthorizationCode>
          <CaptureDateTime>20150824152839</CaptureDateTime>
          <Brand>1</Brand>
          <CardNumber>123456******1122</CardNumber>
          <SalePlanType>2</SalePlanType>
          <ProductType>2</ProductType>
          <NumberOfInstallments>2</NumberOfInstallments>
          <SerialNumber/>
          <CapturedAmount>130.620000</CapturedAmount>
          <AuthorizedAmount>130.620000</AuthorizedAmount>
          <AuthorizedCurrencyCode>986</AuthorizedCurrencyCode>
          <Refund>
            <RefundGrossAmount>130.620000</RefundGrossAmount>
            <RefundAmount>127.628802</RefundAmount>
            <RefundPaymentDate>20150923</RefundPaymentDate>
            <RefundDate>20151019</RefundDate>
          </Refund>
          <Installments>
            <Installment>
              <InstallmentNumber>1</InstallmentNumber>
              <GrossAmount>65.310000</GrossAmount>
              <NetAmount>63.814401</NetAmount>
              <PaymentDate>20150923</PaymentDate>
            </Installment>
            <Installment>
              <InstallmentNumber>2</InstallmentNumber>
              <GrossAmount>65.310000</GrossAmount>
              <NetAmount>63.814401</NetAmount>
              <PaymentDate>20151020</PaymentDate>
            </Installment>
          </Installments>
        </Transaction>
        <Transaction>
          ...

    ```

+ A transação com o StoneId = 33650014557171, aparece no arquivo do dia 24-08 e no arquivo do dia 19-10, como a transação foi cancelada no dia 19-10, o arquivo do dia 19-10 e o arquivo do dia 24-08 apresentam o campo Refund já que o modelo é dinâmico


 O layout 2 segue o modelo de apresentação estático , exibindo as transações como uma "foto" do momento da transação naquele dia, usando o exemplo anterior:


+ Pedido de arquivo para o dia 24-08-2015 (Layout 2):

  ```xml
  <Conciliation>
    <Header>
      <GenerationDateTime>20151013145131</GenerationDateTime>
      <StoneCode>123456789</StoneCode>
      <LayoutVersion>2</LayoutVersion>
      <FileId>500921</FileId>
      <ReferenceDate>20150824</ReferenceDate>
    </Header>
    <FinancialTransactions>
      <Transaction>
        <Events>
          <CancellationCharges>0</CancellationCharges>
          <Cancellations>0</Cancellations>
          <Captures>1</Captures>
          <ChargebackRefunds>0</ChargebackRefunds>
          <Chargebacks>0</Chargebacks>
          <Payments>0</Payments>
        </Events>
        <AcquirerTransactionKey>33650014557171</AcquirerTransactionKey>
        <InitiatorTransactionKey>1331635</InitiatorTransactionKey>
        <AuthorizationDateTime>20150920030409</AuthorizationDateTime>
        <CaptureLocalDateTime>20150920000415</CaptureLocalDateTime>
        <AccountType>2</AccountType>
        <InstallmentType>1</InstallmentType>
        <NumberOfInstallments>10</NumberOfInstallments>
        <AuthorizedAmount>130.620000</AuthorizedAmount>
        <CapturedAmount>130.620000</CapturedAmount>
        <AuthorizationCurrencyCode>986</AuthorizationCurrencyCode>
        <IssuerAuthorizationCode>040947</IssuerAuthorizationCode>
        <BrandId>1</BrandId>
        <CardNumber>422061******2743</CardNumber>
        <Poi>
          <PoiType>4</PoiType>
        </Poi>
        <Installments>
          <Installment>
            <InstallmentNumber>1</InstallmentNumber>
            <GrossAmount>65.310000</GrossAmount>
            <NetAmount>63.814401</NetAmount>
            <PrevisionPaymentDate>20150923</PrevisionPaymentDate>
          </Installment>
          <Installment>
            <InstallmentNumber>2</InstallmentNumber>
            <GrossAmount>65.310000</GrossAmount>
            <NetAmount>63.814401</NetAmount>
            <PrevisionPaymentDate>20151020</PrevisionPaymentDate>
          </Installment>
        </Installments>
      </Transaction>
      <Transaction>
        ...
  ```


+ Pedido do arquivo para o dia 19-10-2015 (Layout 2):

  ```xml
      <Conciliation>
        <Header>
          <GenerationDateTime>20151013145131</GenerationDateTime>
          <StoneCode>123456789</StoneCode>
          <LayoutVersion>2</LayoutVersion>
          <FileId>500921</FileId>
          <ReferenceDate>20151910</ReferenceDate>
        </Header>
        <FinancialTransactions>
          <Transaction>
            <Events>
              <CancellationCharges>0</CancellationCharges>
              <Cancellations>1</Cancellations>
              <Captures>0</Captures>
              <ChargebackRefunds>0</ChargebackRefunds>
              <Chargebacks>0</Chargebacks>
              <Payments>0</Payments>
            </Events>
            <AcquirerTransactionKey>33650014557171</AcquirerTransactionKey>
            <InitiatorTransactionKey>1157795</InitiatorTransactionKey>
            <AuthorizationDateTime>20150824155931</AuthorizationDateTime>
            <CaptureLocalDateTime>20150824125935</CaptureLocalDateTime>
            <Poi>
              <PoiType>4</PoiType>
            </Poi>
            <Cancellations>
              <Cancellation>
                <OperationKey>3635000017434024</OperationKey>
                <CancellationDateTime>20150910034340</CancellationDateTime>
                <ReturnedAmount>130.620000</ReturnedAmount>
                <Billing>
                  <ChargedAmount>127.628802</ChargedAmount>
                  <PrevisionChargeDate>20150911</PrevisionChargeDate>
                </Billing>
              </Cancellation>
            </Cancellations>
          </Transaction>
          <Transaction>
            ...
      ```

+ A transação com o StoneId = 33650014557171, aparece no arquivo do dia 24-08 e no arquivo do dia 19-10, como a transação foi cancelada no dia 19-10, o arquivo do dia 19-10 apresenta o campo Cancellations já que o modelo é estático

## O Fluxo

Quando uma transação é realizada, um nó *Transaction* referente a ela aparecerá sob *FinancialTransactions* no arquivo de conciliação desse dia com um evento *Captured* em *Event*. Se essa transação for modificada (ChargeBack, Cancelamento, Reapresentação de ChargeBack) ainda nesse dia um evento relacionado a essa modificação será incluido em *Event*.

Se a transação sofrer qualquer modificação em um dia diferente do dia da captura a transação aparecerá novamente sob *FinancialTransactions* desse dia com um evento relativo a modificação.

No dia em que essa transação for efetivamente paga ao lojista, um nó *Transaction* aparecerá sob *FinancialTransactionsAccounts*, e um nó *Payment* referente ao pagamento dessa transação em *Payments* no arquivo de conciliação desse dia.

O Fluxo é semelhante para os Eventos financeiros, porém esses ficam sob *FinancialEvents* e *FinancialEventsAccounts*

Portanto o arquivo de conciliação mostra tanto as transações pagas quanto as realizadas no dia referido
## O Arquivo

### Conciliation

Nó principal do arquivo de conciliação

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| Header   | Container | ###### | Contém as informações do lojista e do arquivo |
| FinancialTransactions | Container | ###### | Contém as transações que aconteceram com o lojista no dia requisitado |
| FinancialEvents | Container | ###### | Contém os eventos financeiros lançados para o lojista no dia requisitado |
| FinancialTransactionsAccounts | Container | ###### | Contém as transações que foram pagas/cobradas ao lojista no dia requisitado |
|FinancialEventAccounts | Container | ###### | Contém os eventos que foram pagos/cobrados ao lojista no dia requisistado |
| Payments | Container | ###### |  Contém informações dos pagamentos efetuados relativos as transações e eventos financeiros do arquivo. |
| Trailer | Container | ###### | Contém os totalizadores e contadores do arquivo |

### Header

Nó filho de Conciliation que contém as informações referentes à loja e ao arquivo, como identificação do lojista, data de geração do arquivo, etc.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| GenerationDateTime | Datetime | 14 | Datetime de geração do arquivo (Formato: aaaammddHHmmss)|
| StoneCode | Num | 9 | Código identificador da loja |
| LayoutVersion | Num | 3 | Versão do Layout do arquivo |
| FileId | Num | 26 | Código identificador do arquivo |
| ReferenceDate |Date|8| Data a que se refere o arquivo|

### FinancialTransactions

Nó filho de Conciliation, container de Transaction, que contém as transações capturadas e ou canceladas ocorridas no dia de referência do arquivo.

| Elemento | Descrição |
| -------- | --------- |
| Transaction | Contém informações referentes à transação, como valor total da transação, número de parcelas da transação, etc. |

### <t id="Transaction">Transaction</t>

Nó filho de FinancialTransactions que contém as informações referentes à transação, como valor total da transação, número de parcelas da transação, etc.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| Events| Container | ###### | Contém contadores dos eventos da transação no dia de referência do arquivo.|
| AcquirerTransactionKey | Num | 14 | Identificador único da transação (NSU) gerado pela adquirente|
| InitiatorTransactionKey | Alfa | 128 | Código recebido pelo sistema cliente |
| AuthorizationDateTime| Datetime | 14 | Datetime da autorização (Formato: aaaammddHHmmss) |
| CaptureLocalDateTime | Datetime | 14 | Datetime da captura (Formato: aaaammddHHmmss) no horario local da adquirente|
| [AccountType](#ProductType) | Alfa | 4 | Tipo de conta utilizada na transação crédito, débito, etc...* |
| [InstallmentType](#SalePlanType) | Alfa | 60 | Tipo de parcelamento utilizado na transação lojista/emissor* |
| NumberOfInstallments | Num | 4 | Número de parcelas* |
| AuthorizedAmount | Float | 20 | Valor autorizado* |
| CapturedAmount | Float | 20 | Valor capturado* |
| CanceledAmount | Float | 20 | Total cancelado* |
| AuthorizationCurrencyCode | Num | 4 | Código da moeda* |
| IssuerAuthorizationCode | Num | 6 | Código da autorização fornecido pelo emissor.* |
| [BrandId](#Brand) | Num | 2 | Identificador da bandeira do cartão* |
| CardNumber | Alfa | 19 | Número do cartão (truncado)* |
| Poi | Container | ###### | Contém dados do Ponto de Interação (terminal) que realizou a transação|
| Cancellations | Container | ###### | Contém informações referentes aos cancelamentos, como data de desconto do cancelamento e valor total do cancelamento.** |
| Installments| Container | ###### | Contém as parcelas da transação. |

> \* Elementos que aparecem apenas quando a transação é de captura
>
> ** Elemento que aparece apenas quando a transação é de cancelamento

### <b id="Events">Events</b>

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

<div class="page-break" />

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| [PoiType](#CaptureMethod) | Num | 4 | Tipo do ponto de interação (e-commerce, pos, mobile, etc).|
| SerialNumber | Num | 32 | Número de série do terminal, se existir |

### <cs id="Cancellations">Cancellations</cs>

Contém uma lista de cancelamentos

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| Cancellation | Container | ###### | Representa um cancelamento de uma transação |

### <canc id="Cancellation">Cancellation</canc>

Representa um cancelamento de uma transação.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| OperationKey | Alfa | 32 | identificador único da operação de cancelamento|
| CancellationDateTime | Datetime | 14 |Data hora do cancelamento (Formato: aaaammddHHmmss)|
| ReturnedAmount | Float | 20 | Valor revertido e devolvido ao portador do cartão |
| Billing | Collection | ###### | Lista de cobranças relativa ao cancelamento \* |

>\* Aparece apenas se a transação não tiver sido cancelada no mesmo dia da captura


### Billing

Cobrança relativa ao cancelamento.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| ChargedAmount | Float | 20 | Valor de desconto do cancelamento (descontado do lojista)|
| PrevisionChargeDate | Datetime | 8 | Data prevista para cobrança  (Formato: aaaammdd) |


### Installments

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
|Installment| Container | ###### | Representa uma parcela de uma transação. |

### Installment

Representa uma parcela de uma transação.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| InstallmentNumber | Num | 2 | Número da parcela |
| GrossAmount | Float | 20 | Valor bruto da parcela |
| NetAmount | Float | 20 | Valor liquido da parcela |
| PrevisionPaymentDate | Datetime | 8 | Previsão da data de pagamento (Formato: aaaammdd)|
| Chargeback | Container | ###### | Contém chargebacks relativos a parcela** |
| ChargebackRefund | Container | ###### | Contém estornos de chargeback relativos a parcela\*** |

> ** Elemento que só aparece quando houve Chargeback
>
> \*** Elemento que só aparece quando houve Chargeback e Reapresentação de Chargeback

### <c id="Chargeback">Chargeback</c>

Nó filho de Installment que contém informações sobre o chargeback, como data de desconto, Id do chargeback, data em que ocorreu o chargeback, etc.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| Id | Num | 10 | Identificador do chargeback |
| Amount | Float | 20 | Valor do chargeback |
| Date | Date | 8 | Data em que ocorreu o chargeback. (Formato: aaaammdd) |
| ChargeDate | Date | 8 | Data em que o chargeback será descontado. (Formato: aaaammdd) |
| ReasonCode | Num | 8 | Código de motivo do chargeback informado pelo banco emissor |

### <d id="ChargebackRef">ChargebackRefund</d>

Nó filho de Installment que contém informações sobre a reapresentação do chargeback, como a data em que ocorreu a reapresentação, data de pagamento, etc.

<div class="page-break" />

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| Id | Num | 10 | Identificador da reapresentação do chargeback |
| Amount | Float | 20 | Valor da reapresentação do chargeback |
| Date | Date | 8 | Data em que ocorreu a reapresentação do chargeback. (Formato: aaaammdd) |
| PaymentDate | Date | 8 | Data em que a reapresentação o chargeback será creditada. (Formato: aaaammdd) |
| ReasonCode | Num | 8 | Código de motivo do chargeback informado pelo banco emissor |

### FinancialEvents

Nó filho de Conciliation, previsão de eventos financeiros lançados no dia de referência do arquivo (ajustes financeiros, aluguel de pos, etc...).

| Elemento | Descrição |
| -------- | --------- |
| Event | Contém informações sobre o evento ocorrido |

### <e id ="Event">Event</e>

Nó filho de FinancialEvents , descreve os detalhes do evento ocorrido.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| EventId | Num | 10 | Código identificador do evento |
| Description | Alfa | 60 | Descrição do evento |
| Type | Num | 2 | Tipo do evento |
| PrevisionPaymentDate | Date | 8 | Previsão da data de cobrança |
| Amount | Float | 20 | Valor do evento |

### FinancialTransactionsAccounts

Nó filho de Conciliation, container de Transaction, que contém as transações pagas ou descontadas no dia referência do arquivo.

<div class="page-break" />

| Elemento | Descrição |
| -------- | --------- |
| Transaction | Contém informações referentes à transação, como valor total da transação, número de parcelas da transação, etc. |

### Transaction

Nó filho de FinancialTransactionPayments que contém as informações referentes à transação, como valor total da transação, número de parcelas da transação, etc.

[**Igual ao nó transaction de FinancialTransactions**](#Transaction)


### Events

Nó filho de Transaction, contém contadores dos eventos de uma transação no dia de referência do arquivo.

[**Igual ao nó Events em Transaction de FinancialTransactions**](#Events)

**Porém o único evento que aparece em uma transação de FinancialTransactionAccounts é Payment**

### Cancellations

Contém uma lista de cancelamentos

[**Igual ao nó Cancellations descrito acima**](#Cancellations)
### Cancellation

Representa um cancelamento de uma transação.

[** Igual ao nó Cancellation descrito acima**](#Cancellation)

### Billing

Cobrança relativa ao cancelamento.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| ChargedAmount | Float | 20 | Valor de desconto do cancelamento (descontado do lojista)|
| ChargeDate | Datetime | 8 | Data de cobrança (Formato: aaaammdd)|

### Installments

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
|Installment| Container | ###### | Representa uma parcela de uma transação. |

### Installment

Representa uma parcela de uma transação.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| InstallmentNumber | Num | 2 | Número da parcela |
| GrossAmount | Float | 20 | Valor bruto da parcela |
| NetAmount | Float | 20 | Valor liquido da parcela |
| PaymentDate | Datetime | 8 | Data de pagamento da parcela. (Formato: aaaammdd)|
| AdvanceRateAmount | Float | 20 | Valor cobrado pela antecipação de recebível* |
| AdvancedReceivableOriginalPaymentDate | Datetime | 8 | Data original de pagamento do recebível adiantado* |
| SuspendedByChargeback | Alfa | 4 | Marcador se o pagamento da parcela está suspensa por Chargeback\*\*\*\* |
| Chargeback | Container | ###### | Contém chargebacks relativos a parcela** |
| ChargebackRefund | Container | ###### | Contém estornos de chargeback relativos a parcela\*** |
| PaymentId | Num | 9 | Referência do elemento de pagamento (Payments) no qual a liquidação dessa parcela foi incluida |

> \* Elementos que só aparecem quando houver antecipação
>
> ** Elemento que só aparece quando houve Chargeback
>
> \*** Elemento que só aparece quando houve Chargeback e Reapresentação de Chargeback
>
>  \*\*\*\* Aparece apenas nas parcelas posteriores à parcela que sofreu chargeback

### Chargeback

Nó filho de Installment que contém informações sobre o chargeback, como data de desconto, Id do chargeback, data em que ocorreu o chargeback, etc.

[**Igual ao nó Chargeback descrito acima**](#Chargeback)

### ChargebackRefund

Nó filho de [Installment] que contém informações sobre a reapresentação do chargeback, como a data em que ocorreu a reapresentação, data de pagamento, etc.

[**Igual ao nó ChargebackRefund descrito acima**](#ChargebackRef)

### FinancialEventAccounts

Nó filho de Conciliation, eventos financeiros pagos ou descontados no dia de referência do arquivo (ajustes financeiros, aluguel de pos, etc...).

| Elemento | Descrição |
| -------- | --------- |
| Event | Contém informações sobre o evento ocorrido |

### Event
Nó filho de FinacialEventPayments ,descreve os detalhes do evento ocorrido.

Nó filho de [FinancialEvents] ou de [FinacialEventPayments], descreve os detalhes do evento ocorrido.

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| EventId | Num | 10 | Código identificador do evento |
| PaymentId | Num | 9 | Referência do elemento de pagamento ([Payments]) no qual a liquidação dessa parcela foi incluida, aparece no nó [FinacialEventPayments]|
| Description | Alfa | 60 | Descrição do evento |
| [Type](#type) | Num | 2 | Tipo do evento |
| PaymentDate | Date | 8 | Data em que o evento foi pago. (Formato: aaaammdd) |
| Amount | Float | 20 | Valor do evento |

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
| FavoredBankAccount | Container | ###### | Informações bancarias da conta favorecida pelo pagamento |

### FavoredBankAccount

| Elemento | Tipo | Tamanho | Descrição |
| -------- | ---- | ------- | --------- |
| BankCode | Num | 4 | Código do banco favorecido |
| BankBranch | Alfa | 10 | Número da agencia favorecida |
| BankAccountNumber | Num | 12 | Número da conta favorecida |

### Trailer

Contadores e totalizadores do arquivo.

| Elemento | Tipo | Tamanho | Descrição |
| :-------- | ---- | ------- | --------- |
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

## Exemplo de Layout v2

```xml
<Conciliation>
  <Header>
    <GenerationDateTime>20151013145131</GenerationDateTime>
    <StoneCode>123456789</StoneCode>
    <LayoutVersion>2</LayoutVersion>
    <FileId>020202</FileId>
    <ReferenceDate>20150920</ReferenceDate>
  </Header>
  <FinancialTransactions>
    <Transaction>
      <Events>
        <CancellationCharges>0</CancellationCharges>
        <Cancellations>1</Cancellations>
        <Captures>0</Captures>
        <ChargebackRefunds>0</ChargebackRefunds>
        <Chargebacks>0</Chargebacks>
        <Payments>0</Payments>
      </Events>
      <AcquirerTransactionKey>12345678912356</AcquirerTransactionKey>
      <InitiatorTransactionKey>1117737</InitiatorTransactionKey>
      <AuthorizationDateTime>20150818155931</AuthorizationDateTime>
      <CaptureLocalDateTime>20150818125935</CaptureLocalDateTime>
      <Poi>
        <PoiType>4</PoiType>
      </Poi>
      <Cancellations>
        <Cancellation>
          <OperationKey>3635000017434024</OperationKey>
          <CancellationDateTime>20150920034340</CancellationDateTime>
          <ReturnedAmount>20.000000</ReturnedAmount>
          <Billing>
            <ChargedAmount>19.602000</ChargedAmount>
            <PrevisionChargeDate>20150921</PrevisionChargeDate>
          </Billing>
        </Cancellation>
      </Cancellations>
      <Installments>
        <Installment>
          <InstallmentNumber>1</InstallmentNumber>
          <GrossAmount>20.000000</GrossAmount>
          <NetAmount>19.389317</NetAmount>
          <PrevisionPaymentDate>20150927</PrevisionPaymentDate>
          <AdvanceRateAmount>0.212683</AdvanceRateAmount>
          <AdvancedReceivableOriginalPaymentDate>20151017</AdvancedReceivableOriginalPaymentDate>
        </Installment>
      </Installments>
    </Transaction>
    <Transaction>
      <Events>
        <CancellationCharges>0</CancellationCharges>
        <Cancellations>0</Cancellations>
        <Captures>1</Captures>
        <ChargebackRefunds>0</ChargebackRefunds>
        <Chargebacks>0</Chargebacks>
        <Payments>0</Payments>
      </Events>
      <AcquirerTransactionKey>12345678912345</AcquirerTransactionKey>
      <InitiatorTransactionKey>1331632</InitiatorTransactionKey>
      <AuthorizationDateTime>20150920030009</AuthorizationDateTime>
      <CaptureLocalDateTime>20150920000010</CaptureLocalDateTime>
      <AccountType>2</AccountType>
      <InstallmentType>1</InstallmentType>
      <NumberOfInstallments>1</NumberOfInstallments>
      <AuthorizedAmount>50.000000</AuthorizedAmount>
      <CapturedAmount>50.000000</CapturedAmount>
      <AuthorizationCurrencyCode>986</AuthorizationCurrencyCode>
      <IssuerAuthorizationCode>094736</IssuerAuthorizationCode>
      <BrandId>2</BrandId>
      <CardNumber>132456******1122</CardNumber>
      <Poi>
        <PoiType>4</PoiType>
      </Poi>
      <Installments>
        <Installment>
          <InstallmentNumber>1</InstallmentNumber>
          <GrossAmount>50.000000</GrossAmount>
          <NetAmount>49.005000</NetAmount>
          <PrevisionPaymentDate>20151020</PrevisionPaymentDate>
        </Installment>
      </Installments>
    </Transaction>
    <Transaction>
      <Events>
        <CancellationCharges>0</CancellationCharges>
        <Cancellations>1</Cancellations>
        <Captures>1</Captures>
        <ChargebackRefunds>0</ChargebackRefunds>
        <Chargebacks>0</Chargebacks>
        <Payments>0</Payments>
      </Events>
      <AcquirerTransactionKey>36350017433715</AcquirerTransactionKey>
      <InitiatorTransactionKey>1331697</InitiatorTransactionKey>
      <AuthorizationDateTime>20150920033610</AuthorizationDateTime>
      <CaptureLocalDateTime>20150920003610</CaptureLocalDateTime>
      <AccountType>2</AccountType>
      <InstallmentType>1</InstallmentType>
      <NumberOfInstallments>1</NumberOfInstallments>
      <AuthorizedAmount>125.790000</AuthorizedAmount>
      <CapturedAmount>125.790000</CapturedAmount>
      <CanceledAmount>125.790000</CanceledAmount>
      <AuthorizationCurrencyCode>986</AuthorizationCurrencyCode>
      <IssuerAuthorizationCode>661137</IssuerAuthorizationCode>
      <BrandId>1</BrandId>
      <CardNumber>123456******1122</CardNumber>
      <Poi>
        <PoiType>4</PoiType>
      </Poi>
      <Cancellations>
        <Cancellation>
          <CancellationDateTime>20150920000000</CancellationDateTime>
          <ReturnedAmount>125.790000</ReturnedAmount>
        </Cancellation>
      </Cancellations>
      <Installments>
        <Installment>
          <InstallmentNumber>1</InstallmentNumber>
          <GrossAmount>125.790000</GrossAmount>
          <NetAmount />
        </Installment>
      </Installments>
    </Transaction>
  </FinancialTransactions>
  <FinancialEvents>
    <Event>
      <EventId>29869413</EventId>
      <Description>PosRent</Description>
      <Type>-22</Type>
      <PrevisionPaymentDate>20150923</PrevisionPaymentDate>
      <Amount>-590.000000</Amount>
    </Event>
  </FinancialEvents>
  <FinancialTransactionsAccounts>
    <Transaction>
      <Events>
        <CancellationCharges>0</CancellationCharges>
        <Cancellations>0</Cancellations>
        <Captures>0</Captures>
        <ChargebackRefunds>0</ChargebackRefunds>
        <Chargebacks>0</Chargebacks>
        <Payments>1</Payments>
      </Events>
      <AcquirerTransactionKey>31550012403598</AcquirerTransactionKey>
      <InitiatorTransactionKey>ad50f27deee549b2</InitiatorTransactionKey>
      <AuthorizationDateTime>20150803210946</AuthorizationDateTime>
      <CaptureLocalDateTime>20150803182445</CaptureLocalDateTime>
      <Poi>
        <PoiType>4</PoiType>
      </Poi>
      <Installments>
        <Installment>
          <InstallmentNumber>1</InstallmentNumber>
          <GrossAmount>123.440000</GrossAmount>
          <NetAmount>120.354375</NetAmount>
          <PaymentDate>20150920</PaymentDate>
          <PaymentId>109963</PaymentId>
        </Installment>
      </Installments>
    </Transaction>
    <Transaction>
      <Events>
        <CancellationCharges>0</CancellationCharges>
        <Cancellations>0</Cancellations>
        <Captures>0</Captures>
        <ChargebackRefunds>0</ChargebackRefunds>
        <Chargebacks>0</Chargebacks>
        <Payments>1</Payments>
      </Events>
      <AcquirerTransactionKey>31550012405762</AcquirerTransactionKey>
      <InitiatorTransactionKey>f172e42e9aa7446e</InitiatorTransactionKey>
      <AuthorizationDateTime>20150803212449</AuthorizationDateTime>
      <CaptureLocalDateTime>20150803183941</CaptureLocalDateTime>
      <Poi>
        <PoiType>4</PoiType>
      </Poi>
      <Installments>
        <Installment>
          <InstallmentNumber>1</InstallmentNumber>
          <GrossAmount>468.400000</GrossAmount>
          <NetAmount>457.533120</NetAmount>
          <PaymentDate>20150920</PaymentDate>
          <PaymentId>109963</PaymentId>
        </Installment>
      </Installments>
    </Transaction>
  </FinancialTransactionsAccounts>
  <FinancialEventAccounts>
    <Event>
      <EventId>38883564</EventId>
      <PaymentId>109963</PaymentId>
      <Description>FinancialAdjustment</Description>
      <Type>-27</Type>
      <PaymentDate>20150920</PaymentDate>
      <Amount>900.890000</Amount>
    </Event>
  </FinancialEventAccounts>
  <Payments>
    <Payment>
      <Id>109963</Id>
      <TotalAmount>1478.77</TotalAmount>
      <FavoredBankAccount>
        <BankCode>1</BankCode>
        <BankBranch>24111</BankBranch>
        <BankAccountNumber>0123456</BankAccountNumber>
      </FavoredBankAccount>
    </Payment>
  </Payments>
  <Trailer>
    <CapturedTransactionsQuantity>2</CapturedTransactionsQuantity>
    <CanceledTransactionsQuantity>3</CanceledTransactionsQuantity>
    <PaidInstallmentsQuantity>2</PaidInstallmentsQuantity>
    <ChargedCancellationsQuantity>0</ChargedCancellationsQuantity>
    <ChargebacksQuantity>0</ChargebacksQuantity>
    <ChargebacksRefundQuantity>0</ChargebacksRefundQuantity>
    <ChargedChargebacksQuantity>0</ChargedChargebacksQuantity>
    <PaidChargebacksRefundQuantity>0</PaidChargebacksRefundQuantity>
    <PaidEventsQuantity>1</PaidEventsQuantity>
    <ChargedEventsQuantity>0</ChargedEventsQuantity>
  </Trailer>
</Conciliation>
```

## Arquivos de Teste

Disponibilizamos 6 arquivos para o uso em testes, esses arquivos mostram o histórico de seis transações, são elas e seus respectivos históricos:

| AcquirerTransactionKey | Histórico |
| ---------------------- | --------- |
| 11111111111111 | Captura -> Pagamento |
| 22222222222222 | Captura -> Cancelamento -> Pagamento |
| 33333333333333 | Captura -> ChargeBack -> Pagamento |
| 44444444444444 | Captura -> Chargeback -> ChargebackRefund -> Pagamento |
| 55555555555555 | Captura -> Pagamento -> ChargeBack -> Pagamento -> ChargebackRefund -> Pagamento |
| 66666666666666 | Captura -> Pagamento -> Cancelamento -> Pagamento |

> Arquivos:
>
> [ Arquivo do dia 20151012](/attachment/HomologacaoV2/20151012.xml)
>
> [ Arquivo do dia 20151013](/attachment/HomologacaoV2/20151013.xml)
>
> [ Arquivo do dia 20151016](/attachment/HomologacaoV2/20151016.xml)
>
> [ Arquivo do dia 20151017](/attachment/HomologacaoV2/20151017.xml)
>
> [ Arquivo do dia 20151020](/attachment/HomologacaoV2/20151020.xml)
>
> [ Arquivo do dia 20151021](/attachment/HomologacaoV2/20151021.xml)

# Apêndice

### <cap id="CaptureMethod">CaptureMethod<sup>v1</sup> / POIType<sup>v2</sup></cap>

|Valor|Descrição|
|-----|---------|
|1|POS|
|2|MICRO POS|
|3|TEF|
|4|ECOMMERCE|

### <br id="Brand">Brand<sup>v1v2</sup></br>

|Valor|Descrição|
|-----|---------|
|1|Visa|
|2|MasterCard|

### <pr id="ProductType">ProductType<sup>v1</sup> / AccountType<sup>v2</sup></pr>

<div class="page-break" />

|Valor|Descrição|
|-----|---------|
|1|Debit|
|2|Credit|

### <spt id="SalePlanType"> SalePlanType<sup>v1</sup> /InstallmentType<sup>v2</sup></spt>

|Valor|Descrição|
|-----|---------|
|1|à vista lojista|
|2|parcelado lojista|
|3|parcelado emissor|

### <ty id="type">Type<sup>v1v2</sup></ty>

|Valor|Descrição|
|-----|---------|
|22|PosRent (Aluguel de POS)|
|23|InternalTransfer (Transferência Interna)|
|27|FinancialAdjustment (Ajuste financeiro)|
