# PALOALTO SERVICE

## Micro Serviço paloalto-service

### Fluxo - Service Create

```mermaid
flowchart TD
  Start([Start]) --> GetTx[GetTransactionID]
  GetTx --> ForService{Para cada service}
  ForService --> CheckSingleton[Checar Firewall]
  CheckSingleton -- não --> Err[Retornar core.ERROR]
  CheckSingleton --> Prepare[Montar payload]
  Prepare --> ExistsCall[client.Exists.payload]
  ExistsCall -- não --> CreateCall[client.Create.payload]
  CreateCall -- erro --> Err
  CreateCall --> Continue[Próximo service]
  ExistsCall -- existe --> Continue
  Continue --> ForService
  ForService --> EndServices[Portas Processadas]
  EndServices --> ReturnOk[Retornar COMPLETED]
  Err --> End([End])
  ReturnOk --> End
```

## Payload no Micro Serviço

```json
{
  "Name":"TCP-63000",
  "Path":"service",
  "Port": { 
    "Port":"63000"
  }
}
```

### End-Point API PaloAlto

> /restapi/v10.2/Objects/Services

### Payload API PaloAlto

```json
{
  "entry": {
    "@name": "TCP-63000",
    "description": "TCP-63000",
    "protocol": {
      "tcp": {
        "port": "63000",
      }
    }
  }
}
```