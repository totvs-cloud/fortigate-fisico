## NATs

**Objetivo:** Configurar regras de NAT (Network Address Translation) para gerenciar o tráfego de rede entre diferentes zonas ou interfaces.

### Fluxo - Criação de NAT

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

## Serviços envolvidos

- [v1.1.paloalto.service.create](paloalto-service.md#fluxo---service-create)
- [v1.2.paloalto.host.create](paloalto-host.md#fluxo---host-create)
- [v1.3.paloalto.rule.create](paloalto-rule.md#fluxo---rule-create)
- [v1.4.paloalto.rule.edit](#)

---

### Fluxo - Edição de NAT

```mermaid
graph TD
  v1_0_restrictions_find["v1.0.restrictions.find"] -->|next| v1_1_paloalto_service_create["v1.1.paloalto.service.create"]
  v1_0_restrictions_find["v1.0.restrictions.find"] -->|error| v1_nat_edit["v1.nat.edit"]
  v1_1_paloalto_service_create["v1.1.paloalto.service.create"] -->|next| v1_2_paloalto_host_delete["v1.2.paloalto.host.delete"]
  v1_1_paloalto_service_create["v1.1.paloalto.service.create"] -->|error| v1_nat_edit["v1.nat.edit"]
  v1_2_paloalto_host_delete["v1.2.paloalto.host.delete"] -->|next| v1_3_paloalto_host_create["v1.3.paloalto.host.create"]
  v1_2_paloalto_host_delete["v1.2.paloalto.host.delete"] -->|error| v1_nat_edit["v1.nat.edit"]
  v1_3_paloalto_host_create["v1.3.paloalto.host.create"] -->|next| v1_4_paloalto_rule_delete["v1.4.paloalto.rule.delete"]
  v1_3_paloalto_host_create["v1.3.paloalto.host.create"] -->|error| v1_nat_edit["v1.nat.edit"]
  v1_4_paloalto_rule_delete["v1.4.paloalto.rule.delete"] -->|next| v1_5_paloalto_rule_create["v1.5.paloalto.rule.create"]
  v1_4_paloalto_rule_delete["v1.4.paloalto.rule.delete"] -->|error| v1_nat_edit["v1.nat.edit"]
  v1_5_paloalto_rule_create["v1.5.paloalto.rule.create"] -->|next| v1_6_paloalto_rule_edit["v1.6.paloalto.rule.edit"]
  v1_5_paloalto_rule_create["v1.5.paloalto.rule.create"] -->|error| v1_nat_edit["v1.nat.edit"]
  v1_6_paloalto_rule_edit["v1.6.paloalto.rule.edit"] -->|next| v1_7_paloalto_nat_delete["v1.7.paloalto.nat.delete"]
  v1_6_paloalto_rule_edit["v1.6.paloalto.rule.edit"] -->|error| v1_nat_edit["v1.nat.edit"]
  v1_7_paloalto_nat_delete["v1.7.paloalto.nat.delete"] -->|next| v1_8_paloalto_nat_create["v1.8.paloalto.nat.create"]
  v1_7_paloalto_nat_delete["v1.7.paloalto.nat.delete"] -->|error| v1_nat_edit["v1.nat.edit"]
  v1_8_paloalto_nat_create["v1.8.paloalto.nat.create"] -->|next| v1_9_paloalto_nat_edit["v1.9.paloalto.nat.edit"]
  v1_8_paloalto_nat_create["v1.8.paloalto.nat.create"] -->|error| v1_nat_edit["v1.nat.edit"]
  v1_9_paloalto_nat_edit["v1.9.paloalto.nat.edit"] -->|next| v1_10_fortinet_address_create["v1.10.fortinet.address.create"]
  v1_9_paloalto_nat_edit["v1.9.paloalto.nat.edit"] -->|error| v1_nat_edit["v1.nat.edit"]
  v1_10_fortinet_address_create["v1.10.fortinet.address.create"] -->|next| v1_11_fortinet_vip_delete["v1.11.fortinet.vip.delete"]
  v1_10_fortinet_address_create["v1.10.fortinet.address.create"] -->|error| v1_nat_edit["v1.nat.edit"]
  v1_11_fortinet_vip_delete["v1.11.fortinet.vip.delete"] -->|next| v1_12_fortinet_vip_create["v1.12.fortinet.vip.create"]
  v1_11_fortinet_vip_delete["v1.11.fortinet.vip.delete"] -->|error| v1_nat_edit["v1.nat.edit"]
  v1_12_fortinet_vip_create["v1.12.fortinet.vip.create"] -->|next| v1_13_fortinet_snat_edit["v1.13.fortinet.snat.edit"]
  v1_12_fortinet_vip_create["v1.12.fortinet.vip.create"] -->|error| v1_nat_edit["v1.nat.edit"]
  v1_13_fortinet_snat_edit["v1.13.fortinet.snat.edit"] -->|next| v1_14_fortinet_policy_delete["v1.14.fortinet.policy.delete"]
  v1_13_fortinet_snat_edit["v1.13.fortinet.snat.edit"] -->|error| v1_nat_edit["v1.nat.edit"]
  v1_14_fortinet_policy_delete["v1.14.fortinet.policy.delete"] -->|next| v1_15_fortinet_service_delete["v1.15.fortinet.service.delete"]
  v1_14_fortinet_policy_delete["v1.14.fortinet.policy.delete"] -->|error| v1_nat_edit["v1.nat.edit"]
  v1_15_fortinet_service_delete["v1.15.fortinet.service.delete"] -->|next| v1_16_fortinet_service_create["v1.16.fortinet.service.create"]
  v1_15_fortinet_service_delete["v1.15.fortinet.service.delete"] -->|error| v1_nat_edit["v1.nat.edit"]
  v1_16_fortinet_service_create["v1.16.fortinet.service.create"] -->|next| v1_17_fortinet_policy_create["v1.17.fortinet.policy.create"]
  v1_16_fortinet_service_create["v1.16.fortinet.service.create"] -->|error| v1_nat_edit["v1.nat.edit"]
  v1_17_fortinet_policy_create["v1.17.fortinet.policy.create"] -->|next| v1_18_fortinet_address_delete["v1.18.fortinet.address.delete"]
  v1_17_fortinet_policy_create["v1.17.fortinet.policy.create"] -->|error| v1_nat_edit["v1.nat.edit"]
  v1_18_fortinet_address_delete["v1.18.fortinet.address.delete"] -->|next| v1_nat_edit["v1.nat.edit"]
  v1_18_fortinet_address_delete["v1.18.fortinet.address.delete"] -->|error| v1_nat_edit["v1.nat.edit"]
```

### Fluxo - Remoção de NAT

```mermaid
graph TD
  v1_1_paloalto_rule_delete["v1.1.paloalto.rule.delete"] -->|next| v1_2_paloalto_rule_edit["v1.2.paloalto.rule.edit"]
  v1_1_paloalto_rule_delete["v1.1.paloalto.rule.delete"] -->|error| v1_nat_delete["v1.nat.delete"]
  v1_2_paloalto_rule_edit["v1.2.paloalto.rule.edit"] -->|next| v1_3_paloalto_nat_delete["v1.3.paloalto.nat.delete"]
  v1_2_paloalto_rule_edit["v1.2.paloalto.rule.edit"] -->|error| v1_nat_delete["v1.nat.delete"]
  v1_3_paloalto_nat_delete["v1.3.paloalto.nat.delete"] -->|next| v1_4_paloalto_nat_edit["v1.4.paloalto.nat.edit"]
  v1_3_paloalto_nat_delete["v1.3.paloalto.nat.delete"] -->|error| v1_nat_delete["v1.nat.delete"]
  v1_4_paloalto_nat_edit["v1.4.paloalto.nat.edit"] -->|next| v1_5_paloalto_service_delete["v1.5.paloalto.service.delete"]
  v1_4_paloalto_nat_edit["v1.4.paloalto.nat.edit"] -->|error| v1_nat_delete["v1.nat.delete"]
  v1_5_paloalto_service_delete["v1.5.paloalto.service.delete"] -->|next| v1_6_paloalto_host_delete["v1.6.paloalto.host.delete"]
  v1_5_paloalto_service_delete["v1.5.paloalto.service.delete"] -->|error| v1_nat_delete["v1.nat.delete"]
  v1_6_paloalto_host_delete["v1.6.paloalto.host.delete"] -->|next| v1_7_fortinet_policy_delete["v1.8.fortinet.vip.delete"]
  v1_6_paloalto_host_delete["v1.6.paloalto.host.delete"] -->|error| v1_nat_delete["v1.nat.delete"]
  v1_7_fortinet_policy_delete["v1.7.fortinet.policy.delete"] -->|next| v1_8_fortinet_vip_delete["v1.8.fortinet.vip.delete"]
  v1_7_fortinet_policy_delete["v1.7.fortinet.policy.delete"] -->|error| v1_nat_delete["v1.nat.delete"]
  v1_8_fortinet_vip_delete["v1.8.fortinet.vip.delete"] -->|next| v1_9_fortinet_snat_edit["v1.9.fortinet.snat.edit"]
  v1_8_fortinet_vip_delete["v1.8.fortinet.vip.delete"] -->|error| v1_nat_delete["v1.nat.delete"]
  v1_9_fortinet_snat_edit["v1.9.fortinet.snat.edit"] -->|next| v1_10_fortinet_snat_delete["v1.10.fortinet.snat.delete"]
  v1_9_fortinet_snat_edit["v1.9.fortinet.snat.edit"] -->|error| v1_nat_delete["v1.nat.delete"]
  v1_10_fortinet_snat_delete["v1.10.fortinet.snat.delete"] -->|next| v1_11_fortinet_address_delete["v1.11.fortinet.address.delete"]
  v1_10_fortinet_snat_delete["v1.10.fortinet.snat.delete"] -->|error| v1_nat_delete["v1.nat.delete"]
  v1_11_fortinet_address_delete["v1.11.fortinet.address.delete"] -->|next| v1_12_fortinet_service_delete["v1.12.fortinet.service.delete"]
  v1_11_fortinet_address_delete["v1.11.fortinet.address.delete"] -->|error| v1_nat_delete["v1.nat.delete"]
  v1_12_fortinet_service_delete["v1.12.fortinet.service.delete"] -->|next| v1_nat_delete["v1.nat.delete"]
  v1_12_fortinet_service_delete["v1.12.fortinet.service.delete"] -->|error| v1_nat_delete["v1.nat.delete"]
