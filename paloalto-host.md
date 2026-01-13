# PALOALTO HOST

## Micro Serviço paloalto-host

### Fluxo - Host Create

```mermaid
flowchart TD
  A([Start]) --> B[for each host]
  B --> C{Singleton?}
  C -- No --> E[ERROR]
  C -- Yes --> D[build Client]
  D --> F[new client]
  F --> G{client ok?}
  G -- No --> E
  G -- Yes --> H["get host"]
  H --> I{"code ok? (0 or 5)"}
  I -- No --> E
  I -- Yes --> J[build payload]
  J --> K{host exists?}
  K -- Yes --> O[mark rollback data]
  K -- No --> M["create host (retry)"]
  M --> N{create ok?}
  N -- No --> E
  N -- Yes --> O
  O --> P[update=true]
  P --> R{more hosts?}
  R -- Yes --> B
  R -- No --> S([return COMPLETED])
``` 

## Payload no Micro Serviço - paloalto-host
  
```json
{
  "Name": "TFDNBM_CV5RGT_ate_HST-189.126.152.92",
  "Vsys": "vsys1",
  "Address": "189.126.152.92/32",
  "Identifier": "fisico"
}
```

### End-Point API PaloAlto - Address Object

> /restapi/v10.2/Objects/Addresses

### Payload API PaloAlto - Address Object

```json
{
  "entry": {
    "@name": "TFDNBM_CV5RGT_ate_HST-189.126.152.92",
    "ip-netmask": "189.126.152.92/32"
  }
}
```

### Fluxo - Host Delete
```mermaid
flowchart TD
    A([Início]) --> B{PaloaltoDeleteHost?}
    
    %% Decisão inicial da Flag
    B -- Não --> M([Sucesso])
    B -- Sim --> C(Loop)
    
    %% Início do Loop e Validação de Config
    C --> D{EnvSingletons?}
    D -- Não --> N([Erro: Fim])
    D -- Sim --> E[Inicializar<br/>PaloAlto Client]
    
    %% Criação do Client
    E --> F{Client criado<br/>sem erro?}
    F -- Não --> N
    F -- Sim --> G[API GET: Verificar Host<br/>]
    
    %% GET e Validação
    G --> H{Erro do GET}
    H -- Não --> N
    H -- Sim --> I{Host existe?}
    
    %% Decisão de Deletar
    I -- Não --> L{Mais Hosts<br/>na lista?}
    I -- Sim --> J[API DELETE: Remover Host]
    
    %% DELETE e Validação
    J --> K{Erro do DELETE}
    K -- Não --> N
    K -- Sim --> L
    
    %% Controle do Loop
    L -- Sim --> C
    L -- Não --> M
``` 


## Payload no Micro Serviço - paloalto-host (delete)
  
```json
{
  "Name": "TFDNBM_CV5RGT_ate_HST-189.126.152.92",
  "Vsys": "vsys1",
  "Address": "189.126.152.92/32",
  "Identifier": "fisico"
}
```

### End-Point API PaloAlto - Address Object (delete)

> /restapi/v10.2/Objects/Addresses?location=vsys&vsys=vsys2&name=TFDNBM_CV5RGT_ate_HST-189.126.152.92
