---
title: "Documentação Consolidada — Fluxos de Rede no firewall fisico"
summary: "Este README reúne em um único lugar os fluxos de tratamentos de recursos de rede usados neste projeto: NAT, Load Balancer, VPN e Public Address e demais itens. Inclui diagramas (Mermaid), templates genéricos (YAML/CLI) e checklists de validação."
---

# Documentação Consolidada — Fluxos de Rede

Este README reúne em um único lugar os fluxos de tratamentos de recursos de rede usados neste projeto: NAT, Load Balancer, VPN e Public Address e demais itens. Inclui diagramas (Mermaid), templates genéricos (YAML/CLI) e checklists de validação.

---

## Índice

- [Visão Geral e Convenções](#visão-geral-e-convenções)
- [TAG](#tag)
---

## Visão Geral e Convenções

- Formato: Markdown com blocos Mermaid para diagramas.
- Estrutura de fluxo padrão: criação, edição e remoção.
- Exemplos: templates genéricos de chamadas a API do PaloAlto.

---

## TAG

**Objetivo:** Criação e remoção de Tag no Paloalto. Esse recurso está dentro do fluxo de criação de Organization. v1.4.paloalto.tag.create

### Fluxo

```mermaid
flowchart TB
  v1_1_nsxt_tier1_create["v1.1.nsxt.tier1.create"] -->|next| v1_2_nsxt_locale_service_create["v1.2.nsxt.locale.service.create"]
  v1_1_nsxt_tier1_create["v1.1.nsxt.tier1.create"] -->|error| v1_5_nsxt_tier1_create_error_delete["v1.5.nsxt.tier1.create.error.delete"]
  v1_2_nsxt_locale_service_create["v1.2.nsxt.locale.service.create"] -->|next| v1_3_nsxt_policy_create["v1.3.nsxt.policy.create"]
  v1_2_nsxt_locale_service_create["v1.2.nsxt.locale.service.create"] -->|error| v1_4_nsxt_locale_service_create_error_delete["v1.4.nsxt.locale.service.create.error.delete"]
  v1_3_nsxt_policy_create["v1.3.nsxt.policy.create"] -->|next| v1_4_paloalto_tag_create["v1.4.paloalto.tag.create"]
  v1_3_nsxt_policy_create["v1.3.nsxt.policy.create"] -->|error| v1_3_nsxt_policy_create_error_delete["v1.3.nsxt.policy.create.error.delete"]
  v1_4_paloalto_tag_create["v1.4.paloalto.tag.create"] -->|next| v1_5_vsphere_create_folder["v1.5.vsphere.create.folder"]
  v1_4_paloalto_tag_create["v1.4.paloalto.tag.create"] -->|error| v1_2_paloalto_tag_create_error_delete["v1.2.paloalto.tag.create.error.delete"]
  v1_5_vsphere_create_folder["v1.5.vsphere.create.folder"] -->|next| v1_organization_create["v1.organization.create"]
  v1_5_vsphere_create_folder["v1.5.vsphere.create.folder"] -->|error| v1_1_vsphere_create_folder_error_delete["v1.1.vsphere.create.folder.error.delete"]
  v1_1_vsphere_create_folder_error_delete["v1.1.vsphere.create.folder.error.delete"] -->|next| v1_2_paloalto_tag_create_error_delete["v1.2.paloalto.tag.create.error.delete"]
  v1_1_vsphere_create_folder_error_delete["v1.1.vsphere.create.folder.error.delete"] -->|error| v1_organization_create["v1.organization.create"]
  v1_2_paloalto_tag_create_error_delete["v1.2.paloalto.tag.create.error.delete"] -->|next| v1_3_nsxt_policy_create_error_delete["v1.3.nsxt.policy.create.error.delete"]
  v1_2_paloalto_tag_create_error_delete["v1.2.paloalto.tag.create.error.delete"] -->|error| v1_organization_create["v1.organization.create"]
  v1_3_nsxt_policy_create_error_delete["v1.3.nsxt.policy.create.error.delete"] -->|next| v1_4_nsxt_locale_service_create_error_delete["v1.4.nsxt.locale.service.create.error.delete"]
  v1_3_nsxt_policy_create_error_delete["v1.3.nsxt.policy.create.error.delete"] -->|error| v1_organization_create["v1.organization.create"]
  v1_4_nsxt_locale_service_create_error_delete["v1.4.nsxt.locale.service.create.error.delete"] -->|next| v1_5_nsxt_tier1_create_error_delete["v1.5.nsxt.tier1.create.error.delete"]
  v1_4_nsxt_locale_service_create_error_delete["v1.4.nsxt.locale.service.create.error.delete"] -->|error| v1_organization_create["v1.organization.create"]
  v1_5_nsxt_tier1_create_error_delete["v1.5.nsxt.tier1.create.error.delete"] -->|next| v1_organization_create["v1.organization.create"]
  v1_5_nsxt_tier1_create_error_delete["v1.5.nsxt.tier1.create.error.delete"] -->|error| v1_organization_create["v1.organization.create"]

```

## Micro Serviço paloalto-tag - create

### Fluxo

```mermaid
flowchart LR
  Start([Start])
  ForEach["Loop: para cada tag"]
  Check{Existe Paloalto?}
  CreateClient["Iniciar client PaloAlto"]
  GetTag["Get tag por nome"]
  Exists{Tag existe?}
  CreateTag["Criar tag"]
  SetUpdate["update = true"]
  Next["Próximo tag"]
  ReturnOK["Retorna COMPLETED (update true/false)"]
  Error["Retorna ERROR"]

  Start --> ForEach --> Check
  Check -- Não --> Error
  Check -- Sim --> CreateClient --> GetTag
  GetTag --> Exists
  Exists -- Sim --> Next
  Exists -- Não --> CreateTag --> SetUpdate --> Next
  Next --> ForEach
  ForEach -->|todos processados| ReturnOK
```

### Payload no Micro Serviço

```json
{
  "PaloaltoTag": [
    {
      "ID": 10937,
      "CreatedAt": "2025-12-11T17:47:11Z",
      "UpdatedAt": "2025-12-11T17:47:11Z",
      "DeletedAt": null,
      "OrganizationID": 10175,
      "Name": "TFEMP5_CAU7LX_ate",
      "Identifier": "fisico"
    }
  ]
}
```

### End-Point API PaloAlto

> /restapi/v10.2/Objects/Tags

### Payload API PaloAlto

```json
{
  "tag": {
    "name": "TFEMP5_CAU7LX_ate",
    "color": "red",
    "comments": "TFEMP5_CAU7LX_ate"
  }
}
```



 