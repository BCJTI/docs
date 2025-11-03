# TRACKING360

## Visão Geral

O sistema de Tracking360 rastreia processos automaticamente para empresas que possuem a feature habilitada. O registro é criado automaticamente quando um processo é salvo, através do `afterSave` da tabela `process`.

## Feature de Tracking360

### ID da Feature

```
idfeature: 7d57086f-84ca-4923-955c-5580987f2d54
```

### Funcionamento

- **Todas as empresas** que possuem esta feature habilitada terão seus processos inseridos na tabela `tracking360`
- A inserção ocorre automaticamente via `afterSave` da tabela `process`

## Processamento de Registros

### Condições para Processamento

Um processo será processado quando:

- `dttrackinguntil` estiver com uma **data posterior à data atual**
- `dtnext` estiver **preenchido**
- `dtfinish` estiver **null**

**Regra**: O processo será processado no **primeiro dia após** o `dtnext`.

### Exemplo

```sql
-- Processo será processado após 2024-12-16
dttrackinguntil: 2024-12-20
dtnext: 2024-12-15
dtfinish: null
```

## Registros com Erros (Dados Insuficientes)

Quando falta informação para processar um registro, o sistema cria o registro com valores padrão que indicam que **não será processado**:

### Campos Preenchidos

- `dttrackinguntil`: `2005-02-15 01:00:00-02`
- `fltracking`: `false`
- `dserror`: **Preenchido com o erro** explicando por que não será processado

### Quando Isso Acontece

- Dados incompletos no processo
- Informações obrigatórias ausentes
- Validações falhadas

## Exceções - Processos Permitidos Manualmente

Existe uma **exceção aos processos** onde empresas têm a `idfeature`, mas alguns processos específicos foram solicitados pelo **Felipe Lopes** para burlar a necessidade da feature.

### Lista de Processos Permitidos (Hardcodeada)

Esses processos estão escritos de forma **HARDCODE** no arquivo `load/livetracking.go` e foram **inseridos manualmente** na tabela `tracking360`:

```go
idsProcessesPermitted := []uuid.UUID{
    uuid.MustParse("cf916767-f3d9-4f1f-bae8-7044a291a52d"), // Wind
    uuid.MustParse("9d12ddab-e706-43f4-8833-c2a0d1d68f1c"), // Wind
    uuid.MustParse("60499481-5fe4-4a77-98fa-69172bd26ca5"), // EVL-ES25080002  Landis
    uuid.MustParse("14978120-69f6-43b8-b15b-90c6b3b3a2b5"), // EVL-ES25090005  Gelopar
    uuid.MustParse("3ea918d4-e922-495c-abc4-2b803613f390"), // EVL-ES25090011  Janda
    uuid.MustParse("a7fadbd8-8631-4405-a4d5-59b1ebe77bf0"), // EVL-IA25100087
    uuid.MustParse("25a0ea58-6026-41ef-811c-1d9549daf323"), // EVL-IA25100093
    uuid.MustParse("19e350d3-ec59-4ac7-9215-0f1b1ad69538"), // EVL-IS25080006  da givi
    uuid.MustParse("2a1557af-0f09-42cb-84b3-9e78c05276ea"), // EVL-IS25080052  da vessel
    uuid.MustParse("4a53eb1f-4ff9-4585-9eb7-4488cdbf6f4d"), // EVL-IS25090035  da tagia
}
```

### Observações

- Esses processos **não precisam** ter a feature habilitada
- Foram inseridos **manualmente** na tabela `tracking360`
- A lista está no arquivo `load/livetracking.go`
- **Solicitado por**: Felipe Lopes

## Estrutura de Campos Importantes

### dttrackinguntil
- **Data limite** para rastreamento
- Se anterior à data atual e com `dtfinish` null, não será processado
- Valor padrão para erros: `2005-02-15 01:00:00-02`

### fltracking
- **Flag de rastreamento** (true/false)
- Indica se o processo deve ser rastreado
- `false` quando há erro nos dados

### dtnext
- **Próxima data** de processamento
- Deve estar preenchido para processar
- Processamento ocorre no primeiro dia após esta data

### dtfinish
- **Data de finalização** do rastreamento
- Se `null`, o processo ainda pode ser processado
- Se preenchido, indica que o rastreamento foi finalizado

### dserror
- **Descrição do erro**
- Preenchido quando há problema que impede o processamento
- Contém detalhes sobre por que não será processado

## Fluxo Resumido

1. **Criação do Processo**
   - Processo é salvo na tabela `process`
   - `afterSave` é disparado
   - Verifica se empresa tem `idfeature: 7d57086f-84ca-4923-955c-5580987f2d54`
   - OU se processo está na lista de exceções

2. **Inserção na Tabela tracking360**
   - Registro é criado na tabela `tracking360`
   - Se dados incompletos: `dttrackinguntil = 2005-02-15`, `fltracking = false`, `dserror` preenchido
   - Caso contrário: dados normais preenchidos

3. **Verificação de Processamento**
   - Sistema verifica: `dttrackinguntil > hoje` AND `dtnext` preenchido AND `dtfinish = null`
   - Se condições atendidas: processa no primeiro dia após `dtnext`

4. **Finalização**
   - Quando rastreamento termina: `dtfinish` é preenchido
   - Processo não será mais processado automaticamente

## Localização do Código

- **Arquivo principal**: `load/livetracking.go`
- **Lista de exceções**: Hardcodeada no mesmo arquivo
- **Trigger**: `afterSave` da tabela `process`

