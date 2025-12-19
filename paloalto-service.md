# PALOALTO SERVICE

## Micro Serviço paloalto-service

### Fluxo - Service Create

```mermaid
flowchart TD
  A([Start]) --> B[GetTransactionID]
  B --> C{Para cada service}
  C --> D[Checar Firewall]
  D -- não --> E[Retornar core.ERROR]
  D --> F[Montar payload]
  F --> G[client.Exists.payload]
  G -- não --> H[client.Create.payload]
  H -- erro --> E
  H --> I[Próximo service]

  G -- existe --> I
  I --> C
  C --> J[Portas Processadas]
  J --> K[Retornar COMPLETED]
  E --> L([End])
  K --> L
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

### Fluxo - Service Delete

```mermaid
flowchart TD
  A([Start]) --> B[GetTransactionID]
  B --> C{Para cada service}
  C --> D[Checar Firewall]
  D -- não --> E[Retornar core.ERROR]
  D --> F[Montar payload]
  F --> G[client.Exists.payload]
  G -- não --> I[Próximo service]
  G -- existe --> H[Está em uso?]
  H -- sim --> N[Continue]
  N --> I[Próximo service]
  H -- não --> J[client.Delete.payload]
  J -- erro --> E
  J --> I[Próximo service]
  I --> C
  C --> K[Portas Processadas]
  K --> L[Retornar COMPLETED]
  E --> M([End])
  L --> M
```

## Payload no Micro Serviço

```json
{
  "Name":"TCP-63000",
}
```

### End-Point API PaloAlto

> /restapi/v10.2/Objects/Services

### Payload API PaloAlto

```json
{
  "entry": {
    "@name": "TCP-63000"
  }
}
```