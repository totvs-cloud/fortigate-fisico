# SDN-VPNS

**Objetivo:** Criar e gerenciar uma rede privada virtual (VPN) utilizando Software-Defined Networking (SDN) para melhorar a segurança, flexibilidade e gerenciamento da rede.

## Fluxo - SDN-VPN Create

```mermaid
graph TD
  A([Start]) --> v1_1_nsxt_vpn_service_create["v1.1.nsxt.vpn.service.create"]
  v1_1_nsxt_vpn_service_create["v1.1.nsxt.vpn.service.create"] -->|next| v1_2_nsxt_vpn_endpoint_create["v1.2.nsxt.vpn.endpoint.create"]
  v1_1_nsxt_vpn_service_create["v1.1.nsxt.vpn.service.create"] -->|error| v1_5_nsxt_vpn_service_create_error_delete["v1.5.nsxt.vpn.service.create.error.delete"]
  v1_2_nsxt_vpn_endpoint_create["v1.2.nsxt.vpn.endpoint.create"] -->|next| v1_3_nsxt_vpn_session_create["v1.3.nsxt.vpn.session.create"]
  v1_2_nsxt_vpn_endpoint_create["v1.2.nsxt.vpn.endpoint.create"] -->|error| v1_4_nsxt_vpn_endpoint_create_error_delete["v1.4.nsxt.vpn.endpoint.create.error.delete"]
  v1_3_nsxt_vpn_session_create["v1.3.nsxt.vpn.session.create"] -->|next| v1_4_paloalto_host_create["v1.4.paloalto.host.create"]
  v1_3_nsxt_vpn_session_create["v1.3.nsxt.vpn.session.create"] -->|error| v1_3_nsxt_vpn_session_create_error_delete["v1.3.nsxt.vpn.session.create.error.delete"]
  v1_4_paloalto_host_create["v1.4.paloalto.host.create"] -->|next| v1_5_paloalto_rule_create["v1.5.paloalto.rule.create"]
  v1_4_paloalto_host_create["v1.4.paloalto.host.create"] -->|error| v1_2_paloalto_host_create_error_delete["v1.2.paloalto.host.create.error.delete"]
  v1_5_paloalto_rule_create["v1.5.paloalto.rule.create"] -->|next| v1_6_paloalto_rule_edit["v1.6.paloalto.rule.edit"]
  v1_5_paloalto_rule_create["v1.5.paloalto.rule.create"] -->|error| v1_1_paloalto_rule_create_error_delete["v1.1.paloalto.rule.create.error.delete"]
  v1_6_paloalto_rule_edit["v1.6.paloalto.rule.edit"] -->|next| v1_7_fortinet_address_create["v1.7.fortinet.address.create"]
  v1_6_paloalto_rule_edit["v1.6.paloalto.rule.edit"] -->|error| v1_1_paloalto_rule_create_error_delete["v1.1.paloalto.rule.create.error.delete"]
  v1_7_fortinet_address_create["v1.7.fortinet.address.create"] -->|next| v1_8_fortinet_policy_create["v1.8.fortinet.policy.create"]
  v1_7_fortinet_address_create["v1.7.fortinet.address.create"] -->|error| v1_2_fortinet_address_create_error_delete["v1.2.fortinet.address.create.error.delete"]
  v1_8_fortinet_policy_create["v1.8.fortinet.policy.create"] -->|next| v1_9_fortinet_policy_edit["v1.9.fortinet.policy.edit"]
  v1_8_fortinet_policy_create["v1.8.fortinet.policy.create"] -->|error| v1_1_paloalto_rule_create_error_delete["v1.1.paloalto.rule.create.error.delete"]
  v1_9_fortinet_policy_edit["v1.9.fortinet.policy.edit"] -->|next| v1_vpn_create["v1.vpn.create"]
  v1_9_fortinet_policy_edit["v1.9.fortinet.policy.edit"] -->|error| v1_9_fortinet_policy_create_error_delete["v1.9.fortinet.policy.create.error.delete"]
  v1_1_paloalto_rule_create_error_delete["v1.1.paloalto.rule.create.error.delete"] -->|next| v1_2_paloalto_host_create_error_delete["v1.2.paloalto.host.create.error.delete"]
  v1_1_paloalto_rule_create_error_delete["v1.1.paloalto.rule.create.error.delete"] -->|error| v1_vpn_create["v1.vpn.create"]
  v1_2_fortinet_address_create_error_delete["v1.2.fortinet.address.create.error.delete"] -->|next| v1_1_paloalto_rule_create_error_delete["v1.1.paloalto.rule.create.error.delete"]
  v1_2_fortinet_address_create_error_delete["v1.2.fortinet.address.create.error.delete"] -->|error| v1_vpn_create["v1.vpn.create"]
  v1_2_paloalto_host_create_error_delete["v1.2.paloalto.host.create.error.delete"] -->|next| v1_3_nsxt_vpn_session_create_error_delete["v1.3.nsxt.vpn.session.create.error.delete"]
  v1_2_paloalto_host_create_error_delete["v1.2.paloalto.host.create.error.delete"] -->|error| v1_vpn_create["v1.vpn.create"]
  v1_3_nsxt_vpn_session_create_error_delete["v1.3.nsxt.vpn.session.create.error.delete"] -->|next| v1_4_nsxt_vpn_endpoint_create_error_delete["v1.4.nsxt.vpn.endpoint.create.error.delete"]
  v1_3_nsxt_vpn_session_create_error_delete["v1.3.nsxt.vpn.session.create.error.delete"] -->|error| v1_vpn_create["v1.vpn.create"]
  v1_4_nsxt_vpn_endpoint_create_error_delete["v1.4.nsxt.vpn.endpoint.create.error.delete"] -->|next| v1_5_nsxt_vpn_service_create_error_delete["v1.5.nsxt.vpn.service.create.error.delete"]
  v1_4_nsxt_vpn_endpoint_create_error_delete["v1.4.nsxt.vpn.endpoint.create.error.delete"] -->|error| v1_vpn_create["v1.vpn.create"]
  v1_5_nsxt_vpn_service_create_error_delete["v1.5.nsxt.vpn.service.create.error.delete"] -->|next| v1_vpn_create["v1.vpn.create"]
  v1_5_nsxt_vpn_service_create_error_delete["v1.5.nsxt.vpn.service.create.error.delete"] -->|error| v1_vpn_create["v1.vpn.create"]
  v1_9_fortinet_policy_create_error_delete["v1.9.fortinet.policy.create.error.delete"] -->|next| v1_1_paloalto_rule_create_error_delete["v1.1.paloalto.rule.create.error.delete"]
  v1_9_fortinet_policy_create_error_delete["v1.9.fortinet.policy.create.error.delete"] -->|error| v1_vpn_create["v1.vpn.create"]
```

### Serviços envolvidos

- [v1.4.paloalto.host.create](paloalto-host.md#fluxo---host-create)
- [v1.5.paloalto.rule.create](paloalto-rule.md#fluxo---rule-create)
- [v1.6.paloalto.rule.edit](#)

## Fluxo - SDN-VPN Delete

```mermaid
graph TD
  A([Start]) --> v1_1_nsxt_vpn_session_delete["v1.1.nsxt.vpn.session.delete"]
  v1_1_nsxt_vpn_session_delete["v1.1.nsxt.vpn.session.delete"] -->|next| v1_2_nsxt_vpn_endpoint_delete["v1.2.nsxt.vpn.endpoint.delete"]
  v1_1_nsxt_vpn_session_delete["v1.1.nsxt.vpn.session.delete"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_2_nsxt_vpn_endpoint_delete["v1.2.nsxt.vpn.endpoint.delete"] -->|next| v1_3_nsxt_vpn_service_delete["v1.3.nsxt.vpn.service.delete"]
  v1_2_nsxt_vpn_endpoint_delete["v1.2.nsxt.vpn.endpoint.delete"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_3_nsxt_vpn_service_delete["v1.3.nsxt.vpn.service.delete"] -->|next| v1_4_paloalto_rule_edit["v1.4.paloalto.rule.edit"]
  v1_3_nsxt_vpn_service_delete["v1.3.nsxt.vpn.service.delete"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_4_paloalto_rule_edit["v1.4.paloalto.rule.edit"] -->|next| v1_5_paloalto_rule_delete["v1.5.paloalto.rule.delete"]
  v1_4_paloalto_rule_edit["v1.4.paloalto.rule.edit"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_5_paloalto_rule_delete["v1.5.paloalto.rule.delete"] -->|next| v1_6_paloalto_host_delete["v1.6.paloalto.host.delete"]
  v1_5_paloalto_rule_delete["v1.5.paloalto.rule.delete"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_6_paloalto_host_delete["v1.6.paloalto.host.delete"] -->|next| v1_7_fortinet_policy_delete["v1.7.fortinet.policy.delete"]
  v1_6_paloalto_host_delete["v1.6.paloalto.host.delete"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_7_fortinet_policy_delete["v1.7.fortinet.policy.delete"] -->|next| v1_8_fortinet_address_delete["v1.8.fortinet.address.delete"]
  v1_7_fortinet_policy_delete["v1.7.fortinet.policy.delete"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_8_fortinet_address_delete["v1.8.fortinet.address.delete"] -->|next| v1_vpn_delete["v1.vpn.delete"]
  v1_8_fortinet_address_delete["v1.8.fortinet.address.delete"] -->|error| v1_vpn_delete["v1.vpn.delete"]
```
