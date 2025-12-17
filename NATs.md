## NATs

**Objetivo:** Configurar regras de NAT (Network Address Translation) para gerenciar o tráfego de rede entre diferentes zonas ou interfaces.

### Fluxo

```mermaid
flowchart TB
  v1_0_restrictions_find["v1.0.restrictions.find"] -->|next| v1_1_paloalto_service_create["v1.1.paloalto.service.create"]
  v1_0_restrictions_find["v1.0.restrictions.find"] -->|error| v1_nat_create["v1.nat.create"]
  v1_1_paloalto_service_create["v1.1.paloalto.service.create"] -->|next| v1_2_paloalto_host_create["v1.2.paloalto.host.create"]
  v1_1_paloalto_service_create["v1.1.paloalto.service.create"] -->|error| v1_nat_create["v1.nat.create"]
  v1_2_paloalto_host_create["v1.2.paloalto.host.create"] -->|next| v1_3_paloalto_rule_create["v1.3.paloalto.rule.create"]
  v1_2_paloalto_host_create["v1.2.paloalto.host.create"] -->|error| v1_8_paloalto_host_create_error_delete["v1.8.paloalto.host.create.error.delete"]
  v1_3_paloalto_rule_create["v1.3.paloalto.rule.create"] -->|next| v1_4_paloalto_rule_edit["v1.4.paloalto.rule.edit"]
  v1_3_paloalto_rule_create["v1.3.paloalto.rule.create"] -->|error| v1_7_paloalto_rule_create_error_delete["v1.7.paloalto.rule.create.error.delete"]
  v1_4_paloalto_rule_edit["v1.4.paloalto.rule.edit"] -->|next| v1_5_paloalto_nat_create["v1.5.paloalto.nat.create"]
  v1_4_paloalto_rule_edit["v1.4.paloalto.rule.edit"] -->|error| v1_6_paloalto_rule_edit_error_edit["v1.6.paloalto.rule.edit.error.edit"]
  v1_5_paloalto_nat_create["v1.5.paloalto.nat.create"] -->|next| v1_6_paloalto_nat_edit["v1.6.paloalto.nat.edit"]
  v1_5_paloalto_nat_create["v1.5.paloalto.nat.create"] -->|error| v1_5_paloalto_nat_create_error_delete["v1.5.paloalto.nat.create.error.delete"]
  v1_6_paloalto_nat_edit["v1.6.paloalto.nat.edit"] -->|next| v1_7_fortinet_address_create["v1.7.fortinet.address.create"]
  v1_6_paloalto_nat_edit["v1.6.paloalto.nat.edit"] -->|error| v1_4_paloalto_nat_edit_error_edit["v1.4.paloalto.nat.edit.error.edit"]
  v1_7_fortinet_address_create["v1.7.fortinet.address.create"] -->|next| v1_8_fortinet_ippool_create["v1.8.fortinet.ippool.create"]
  v1_7_fortinet_address_create["v1.7.fortinet.address.create"] -->|error| v1_3_fortinet_address_create_error_delete["v1.3.fortinet.address.create.error.delete"]
  v1_8_fortinet_ippool_create["v1.8.fortinet.ippool.create"] -->|next| v1_9_fortinet_service_create["v1.9.fortinet.service.create"]
  v1_8_fortinet_ippool_create["v1.8.fortinet.ippool.create"] -->|error| v1_5_paloalto_nat_create_error_delete["v1.5.paloalto.nat.create.error.delete"]
  v1_9_fortinet_service_create["v1.9.fortinet.service.create"] -->|next| v1_10_fortinet_vip_create["v1.10.fortinet.vip.create"]
  v1_9_fortinet_service_create["v1.9.fortinet.service.create"] -->|error| v1_3_fortinet_address_create_error_delete["v1.3.fortinet.address.create.error.delete"]
  v1_10_fortinet_vip_create["v1.10.fortinet.vip.create"] -->|next| v1_11_fortinet_policy_create["v1.11.fortinet.policy.create"]
  v1_10_fortinet_vip_create["v1.10.fortinet.vip.create"] -->|error| v1_2_fortinet_vip_create_error_delete["v1.2.fortinet.vip.create.error.delete"]
  v1_11_fortinet_snat_edit["v1.11.fortinet.snat.edit"] -->|next| v1_nat_create["v1.nat.create"]
  v1_11_fortinet_snat_edit["v1.11.fortinet.snat.edit"] -->|error| v1_1_fortinet_snat_create_error_delete["v1.1.fortinet.snat.create.error.delete"]
  v1_11_fortinet_policy_create["v1.11.fortinet.policy.create"] -->|next| v1_12_fortinet_policy_edit["v1.12.fortinet.policy.edit"]
  v1_11_fortinet_policy_create["v1.11.fortinet.policy.create"] -->|error| v1_3_fortinet_policy_create_error_delete["v1.3.fortinet.policy.create.error.delete"]
  v1_12_fortinet_policy_edit["v1.12.fortinet.policy.edit"] -->|next| v1_13_fortinet_snat_create["v1.13.fortinet.snat.create"]
  v1_12_fortinet_policy_edit["v1.12.fortinet.policy.edit"] -->|error| v1_3_fortinet_policy_create_error_delete["v1.3.fortinet.policy.create.error.delete"]
  v1_13_fortinet_snat_create["v1.13.fortinet.snat.create"] -->|next| v1_11_fortinet_snat_edit["v1.11.fortinet.snat.edit"]
  v1_13_fortinet_snat_create["v1.13.fortinet.snat.create"] -->|error| v1_1_fortinet_snat_create_error_delete["v1.1.fortinet.snat.create.error.delete"]
  v1_1_fortinet_snat_create_error_delete["v1.1.fortinet.snat.create.error.delete"] -->|next| v1_2_fortinet_vip_create_error_delete["v1.2.fortinet.vip.create.error.delete"]
  v1_1_fortinet_snat_create_error_delete["v1.1.fortinet.snat.create.error.delete"] -->|error| v1_nat_create["v1.nat.create"]
  v1_2_fortinet_vip_create_error_delete["v1.2.fortinet.vip.create.error.delete"] -->|next| v1_3_fortinet_address_create_error_delete["v1.3.fortinet.address.create.error.delete"]
  v1_2_fortinet_vip_create_error_delete["v1.2.fortinet.vip.create.error.delete"] -->|error| v1_nat_create["v1.nat.create"]
  v1_3_fortinet_address_create_error_delete["v1.3.fortinet.address.create.error.delete"] -->|next| v1_5_paloalto_nat_create_error_delete["v1.5.paloalto.nat.create.error.delete"]
  v1_3_fortinet_address_create_error_delete["v1.3.fortinet.address.create.error.delete"] -->|error| v1_nat_create["v1.nat.create"]
  v1_3_fortinet_policy_create_error_delete["v1.3.fortinet.policy.create.error.delete"] -->|next| v1_2_fortinet_vip_create_error_delete["v1.2.fortinet.vip.create.error.delete"]
  v1_3_fortinet_policy_create_error_delete["v1.3.fortinet.policy.create.error.delete"] -->|error| v1_nat_create["v1.nat.create"]
  v1_4_paloalto_nat_edit_error_edit["v1.4.paloalto.nat.edit.error.edit"] -->|next| v1_5_paloalto_nat_create_error_delete["v1.5.paloalto.nat.create.error.delete"]
  v1_4_paloalto_nat_edit_error_edit["v1.4.paloalto.nat.edit.error.edit"] -->|error| v1_nat_create["v1.nat.create"]
  v1_5_paloalto_nat_create_error_delete["v1.5.paloalto.nat.create.error.delete"] -->|next| v1_6_paloalto_rule_edit_error_edit["v1.6.paloalto.rule.edit.error.edit"]
  v1_5_paloalto_nat_create_error_delete["v1.5.paloalto.nat.create.error.delete"] -->|error| v1_nat_create["v1.nat.create"]
  v1_6_paloalto_rule_edit_error_edit["v1.6.paloalto.rule.edit.error.edit"] -->|next| v1_7_paloalto_rule_create_error_delete["v1.7.paloalto.rule.create.error.delete"]
  v1_6_paloalto_rule_edit_error_edit["v1.6.paloalto.rule.edit.error.edit"] -->|error| v1_nat_create["v1.nat.create"]
  v1_7_paloalto_rule_create_error_delete["v1.7.paloalto.rule.create.error.delete"] -->|next| v1_8_paloalto_host_create_error_delete["v1.8.paloalto.host.create.error.delete"]
  v1_7_paloalto_rule_create_error_delete["v1.7.paloalto.rule.create.error.delete"] -->|error| v1_nat_create["v1.nat.create"]
  v1_8_paloalto_host_create_error_delete["v1.8.paloalto.host.create.error.delete"] -->|next| v1_nat_create["v1.nat.create"]
  v1_8_paloalto_host_create_error_delete["v1.8.paloalto.host.create.error.delete"] -->|error| v1_nat_create["v1.nat.create"]
```

## Micro Serviço paloalto-service - create

### Fluxo

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