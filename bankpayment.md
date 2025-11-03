# BANKPAYMENT

## Visão Geral

O processo de pagamento bancário envolve a geração de GNRE (Guia Nacional de Recolhimento de Tributos Estaduais) e posterior envio para pagamento junto ao banco. Este documento explica o fluxo completo e como monitorar cada etapa.

## Query para Busca de Registros

```sql
select cs.obcnee -> 'obAddress' ->> 'cdState' as cdstate, 
       cs.cdcoshipment, 
       ccs.sttaxsend, 
       ccs.sttaxprocess, 
       ccs.stpayment, 
       ccs.stpaymentprocess, 
       ccs.dserror, 
       ccs.cdreceipt,
       ccs.idbankpaymentgroup, 
       ccs.idcotaxcontrolshipment, 
       cs.obcnee ->> 'cdDocument', 
       ccs.dtupdated, 
       ccs.dsscancode, 
       bpg.idbankpaymentgroup,
       ccs.*, 
       bpg.*
from smart.process p
join smart.processvolume pv on pv.idprocess = p.idprocess 
join smart.coshipment cs on cs.idprocessvolume = pv.idprocessvolume 
join smart.cotaxcontrolshipment ccs on ccs.idcotaxcontrolshipment = cs.idcoshipment 
join smart.cotaxcontrol cc on cc.idcotaxcontrol = ccs.idcotaxcontrol 
left join smart.bankpaymentgroup bpg on bpg.idbankpaymentgroup = ccs.idbankpaymentgroup 
where p.cdprocess in ('PHX-IC23120003') 
  and cc.tpintegration = 'NOBORDIST'  -- ou 'SMART' para integração Smart
  and ccs.vltax > 0 
  and tptaxpayment = 'BARCODE_AND_PAYMENT' 
  and ccs.stpayment is null;  -- Remover este filtro se o pagamento já foi iniciado
```

### Observações sobre a Query:

- **tpintegration**: 
  - `'NOBORDIST'` - Integração NoBorderDist
  - `'SMART'` - Integração Smart (comentar o NOBORDIST e usar este)
- **stpayment is null**: Se já começou o pagamento ou foi enviado para o banco, remover este filtro, pois o `stpayment` estará preenchido com `ERROR` ou `PENDING`, dependendo da situação

## Status de Geração de GNRE

Os campos `sttaxsend` e `sttaxprocess` são status relacionados à geração de GNRE na Sefaz.

### Status Possíveis:

#### 1. Ambos PENDING
```
sttaxsend = 'PENDING'
sttaxprocess = 'PENDING'
```
- **Significado**: Está tentando enviar para a Sefaz para gerar a GNRE
- **Job responsável**: 
  - `gnresendProd` 
  - `gnreSendCotaxProd`

#### 2. Enviado, aguardando resultado
```
sttaxsend = 'SUCCESS'
sttaxprocess = 'AWAITING'
```
- **Significado**: Foi enviado para a Sefaz, mas ainda precisa buscar o resultado via `cdreference` para receber o código de barras
- **Job responsável**: 
  - `gnreCheckProcessingProd`
  - `gnreCheckProcessingCotaxProd`

#### 3. GNRE gerada com sucesso
```
sttaxsend = 'SUCCESS'
sttaxprocess = 'SUCCESS'
```
- **Significado**: GNRE gerada com sucesso, código de barras disponível
- **Próximo passo**: Iniciar processo de pagamento

## Status de Pagamento Bancário

Após a GNRE ser gerada (`sttaxsend = 'SUCCESS'` e `sttaxprocess = 'SUCCESS'`), inicia-se a parte de pagamentos.

Os campos relacionados são: `stpayment` e `stpaymentprocess`.

### Status Inicial (Disponível para Pagamento)

```
stpayment is null
stpaymentprocess is null
```

- **Significado**: Está disponível para pagamento, ainda não foi enviado nem está na fila para ir ao banco
- **Ação**: Pode ser enviado para pagamento

### Enviado para Pagamento

Quando existe `idbankpaymentgroup` preenchido, significa que foi "mandado para pagamento".

**Exceção**: Se a integração for `NOBORDIST`, o pagamento é feito via GET direto pelo `cdreference`. É necessário visualizar os logs no `requestresponse` para entender o que aconteceu.

#### Status na Tabela bankpaymentgroup

```sql
bpg.stprocess = 'PENDING'
bpg.stpayment = 'PENDING'
bpg.dsnotes is null
```

- **Significado**: 
  - `stprocess` e `stpayment` estão como `PENDING` na tabela `bankpaymentgroup`
  - Está aguardando para ser enviado ao banco
- **Job responsável**: `BankPaymentLoteProd` (flow: `bankinterpaymentlote`)

### Erros

Se ocorrer algum erro, tanto na tabela `cotaxcontrol`/`cotaxcontrolshipment` quanto na `bankpaymentgroup`, deve haver uma mensagem no campo `dserror`.

## Enviado ao Banco

Após o envio ao banco, o status ficará como **"aguardando pagamento"**.

### Jobs de Busca de Pagamento

Após enviar ao banco, os seguintes jobs são executados para buscar o status do pagamento:

1. **BankInterPaymentLoteProd** 
   - Flow: `banksearchlote`
   - **Descrição**: Buscar lote Inter (BB não precisa)

2. **BankInterSearchBoletoProd**
   - Flow: `bankinterpaymentboleto`
   - **Descrição**: Buscar boleto individual

3. **BankInterPaymentCotaxProd**
   - Flow: `bankinterpaymentboletocotax`
   - **Descrição**: Buscar pagamento via cotax

### Condições para Buscar Pagamento

- **stpayment** precisa estar como `PENDING`

### Busca por Boleto

A busca é feita pelo **número do boleto**.

**Observação para BB (Banco do Brasil)**: 
- A busca é feita pelo `CdReference`, que é o ID que o banco retorna ao fazer a inserção do boleto

## Logs e Depuração

Os logs de todas as requisições ficam na tabela `requestresponse`.

### Consulta de Logs

```sql
select * 
from smart.requestresponse 
where '5f34a480-33e8-4106-bc83-bc7155ca357e' = any(idsrecord);
```

Onde `idsrecord` é a chave primária de:
- `bankpaymentgroup` (idbankpaymentgroup)
- `cotaxcontrolshipment` (idcotaxcontrolshipment)
- `cotaxcontrol` (idcotaxcontrol)

### Quando Consultar Logs

- Para entender erros durante o processo
- Para verificar requisições enviadas/recebidas
- Para integração NOBORDIST (onde não há uso de bankpaymentgroup)

## Fluxo Resumido

1. **Geração GNRE**
   - `sttaxsend = PENDING` e `sttaxprocess = PENDING` → Enviando para Sefaz
   - `sttaxsend = SUCCESS` e `sttaxprocess = AWAITING` → Aguardando resultado
   - `sttaxsend = SUCCESS` e `sttaxprocess = SUCCESS` → GNRE gerada ✅

2. **Preparação para Pagamento**
   - `stpayment = null` → Disponível para pagamento
   - Preenchimento de `idbankpaymentgroup` → Enviado para grupo de pagamento

3. **Envio ao Banco**
   - `stprocess = PENDING` e `stpayment = PENDING` (na bankpaymentgroup) → Aguardando envio
   - Jobs executam o envio → Status muda para "aguardando pagamento"

4. **Busca de Pagamento**
   - Jobs executam busca periódica pelo número do boleto (ou `CdReference` no BB)
   - Atualização de status conforme resultado do banco

## Tabelas Principais

- `smart.process` - Processos
- `smart.processvolume` - Volumes do processo
- `smart.coshipment` - Conhecimentos de embarque
- `smart.cotaxcontrolshipment` - Controle de impostos por conhecimento
- `smart.cotaxcontrol` - Controle de impostos
- `smart.bankpaymentgroup` - Grupos de pagamento bancário
- `smart.requestresponse` - Logs de requisições

