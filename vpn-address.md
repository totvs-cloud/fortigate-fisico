# VPN-ADDRESS

**Objetivo** Configurar endereços VPN em dispositivos PaloAlto para permitir a comunicação segura entre redes remotas e a rede corporativa, garantindo a integridade e confidencialidade dos dados transmitidos.

## Fluxo - VPN Address Delete

```mermaid
graph TD
  v1_1_paloalto_rule_delete["v1.1.paloalto.rule.delete"] -->|next| v1_2_paloalto_rule_edit["v1.2.paloalto.rule.edit"]
  v1_1_paloalto_rule_delete["v1.1.paloalto.rule.delete"] -->|error| v1_vpn_address_delete["v1.vpn.address.delete"]
  v1_2_paloalto_rule_edit["v1.2.paloalto.rule.edit"] -->|next| v1_3_paloalto_nat_delete["v1.3.paloalto.nat.delete"]
  v1_2_paloalto_rule_edit["v1.2.paloalto.rule.edit"] -->|error| v1_vpn_address_delete["v1.vpn.address.delete"]
  v1_3_paloalto_nat_delete["v1.3.paloalto.nat.delete"] -->|next| v1_4_paloalto_nat_edit["v1.4.paloalto.nat.edit"]
  v1_3_paloalto_nat_delete["v1.3.paloalto.nat.delete"] -->|error| v1_vpn_address_delete["v1.vpn.address.delete"]
  v1_4_paloalto_nat_edit["v1.4.paloalto.nat.edit"] -->|next| v1_5_paloalto_host_delete["v1.5.paloalto.host.delete"]
  v1_4_paloalto_nat_edit["v1.4.paloalto.nat.edit"] -->|error| v1_vpn_address_delete["v1.vpn.address.delete"]
  v1_5_paloalto_host_delete["v1.5.paloalto.host.delete"] -->|next| v1_6_fortinet_vip_delete["v1.6.fortinet.vip.delete"]
  v1_5_paloalto_host_delete["v1.5.paloalto.host.delete"] -->|error| v1_vpn_address_delete["v1.vpn.address.delete"]
  v1_6_fortinet_vip_delete["v1.6.fortinet.vip.delete"] -->|next| v1_7_fortinet_snat_delete["v1.7.fortinet.snat.delete"]
  v1_6_fortinet_vip_delete["v1.6.fortinet.vip.delete"] -->|error| v1_vpn_address_delete["v1.vpn.address.delete"]
  v1_7_fortinet_snat_delete["v1.7.fortinet.snat.delete"] -->|next| v1_8_fortinet_ippool_delete["v1.8.fortinet.ippool.delete"]
  v1_7_fortinet_snat_delete["v1.7.fortinet.snat.delete"] -->|error| v1_vpn_address_delete["v1.vpn.address.delete"]
  v1_8_fortinet_ippool_delete["v1.8.fortinet.ippool.delete"] -->|next| v1_9_nsxt_lb_edit["v1.9.nsxt.lb.edit"]
  v1_8_fortinet_ippool_delete["v1.8.fortinet.ippool.delete"] -->|error| v1_vpn_address_delete["v1.vpn.address.delete"]
  v1_9_nsxt_lb_edit["v1.9.nsxt.lb.edit"] -->|next| v1_10_nsxt_vs_delete["v1.10.nsxt.vs.delete"]
  v1_9_nsxt_lb_edit["v1.9.nsxt.lb.edit"] -->|error| v1_vpn_address_delete["v1.vpn.address.delete"]
  v1_10_nsxt_vs_delete["v1.10.nsxt.vs.delete"] -->|next| v1_11_nsxt_pool_delete["v1.11.nsxt.pool.delete"]
  v1_10_nsxt_vs_delete["v1.10.nsxt.vs.delete"] -->|error| v1_vpn_address_delete["v1.vpn.address.delete"]
  v1_11_nsxt_pool_delete["v1.11.nsxt.pool.delete"] -->|next| v1_vpn_address_delete["v1.vpn.address.delete"]
  v1_11_nsxt_pool_delete["v1.11.nsxt.pool.delete"] -->|error| v1_vpn_address_delete["v1.vpn.address.delete"]
```