# MERCANTE

## Visão Geral

O processo MERCANTE é composto por duas etapas principais (Step2 e Step7) que trabalham em conjunto para buscar e atualizar dados de mercadores no sistema.

## Step2 - Busca e Inserção Inicial

O **Step2** realiza a busca no site da Mercante por data. **Importante**: este passo não tem vínculo com o Smart.

### Funcionalidades:

- Busca todos os mercadores que o certificado tem acesso no site da Mercante
- Insere ou atualiza registros na tabela `smart.merchant`
- Ao final, realiza o link com o processo caso encontre `cdcarrierdocument`

### Observações:

- A busca é baseada em data
- Não depende do sistema Smart para realizar a busca inicial
- Os dados são sincronizados para a tabela `smart.merchant`

## Step7 - Complementação de Dados

O **Step7** completa a atualização dos dados do mercador.

### Funcionalidades:

- Busca os dados restantes do mercador
- Atualiza a tabela `merchant` deixando-a bem populada
- Gera o PDF do documento

### Observações:

- Depende do Step2 ter sido executado previamente
- Enriquece os dados já inseridos/atualizados pelo Step2

## Diferença entre Office na tabela Merchant e Office do Processo

### Office na tabela `merchant`:
- É o **office onde o mercador foi encontrado** pelo certificado no site da Mercante
- Reflete a origem/organização no sistema Mercante

### Office do processo:
- É o **office onde o processo está inserido** no Smart
- Pode ser diferente do office encontrado na Mercante, dependendo de como o processo foi criado no Smart

## Como Usar

### Executar Step2

```json
{
    "name": "Job MerchantStep2",
    "type": "go",
    "request": "launch",
    "mode": "auto",
    "program": "${workspaceFolder}/etl/job/main.go",
    "args": [
        "-env",
        "prod",
        "-flow",
        "mercstep2",
        "-loglevel",
        "4",
        "-parallel",
        "1",
        "-idorganization",
        "9821c7e2-ea4e-49ee-ad39-8f210f1004c6"
    ]
}
```

### Executar Step7

Para executar o Step7, basta alterar o parâmetro `-flow` de `mercstep2` para `mercstep7`:

```json
{
    "name": "Job MerchantStep7",
    "type": "go",
    "request": "launch",
    "mode": "auto",
    "program": "${workspaceFolder}/etl/job/main.go",
    "args": [
        "-env",
        "prod",
        "-flow",
        "mercstep7",
        "-loglevel",
        "4",
        "-parallel",
        "1",
        "-idorganization",
        "9821c7e2-ea4e-49ee-ad39-8f210f1004c6"
    ]
}
```

## Parâmetros

- `-env`: Ambiente de execução (ex: `prod`)
- `-flow`: Fluxo a ser executado (`mercstep2` ou `mercstep7`)
- `-loglevel`: Nível de log (ex: `4`)
- `-parallel`: Número de execuções paralelas (ex: `1`)
- `-idorganization`: ID da organização a ser processada

