# VPNs

**Obejtivo** Configurar VPNs (Virtual Private Networks) para permitir conexões seguras e criptografadas entre redes remotas e a rede local, garantindo a integridade e confidencialidade dos dados transmitidos.

## Fluxo - VPN Create

```mermaid
graph TD
  A([Start]) --> v1_1_paloalto_host_create["v1.1.paloalto.host.create"]
  v1_1_paloalto_host_create["v1.1.paloalto.host.create"] -->|next| v1_2_paloalto_host_edit["v1.2.paloalto.host.edit"]
  v1_1_paloalto_host_create["v1.1.paloalto.host.create"] -->|error| v1_vpn_create["v1.vpn.create"]
  v1_2_paloalto_host_edit["v1.2.paloalto.host.edit"] -->|next| v1_3_paloalto_ike_create["v1.3.paloalto.ike.create"]
  v1_2_paloalto_host_edit["v1.2.paloalto.host.edit"] -->|error| v1_vpn_create["v1.vpn.create"]
  v1_3_paloalto_ike_create["v1.3.paloalto.ike.create"] -->|next| v1_4_paloalto_tunnel_create["v1.4.paloalto.tunnel.create"]
  v1_3_paloalto_ike_create["v1.3.paloalto.ike.create"] -->|error| v1_vpn_create["v1.vpn.create"]
  v1_4_paloalto_tunnel_create["v1.4.paloalto.tunnel.create"] -->|next| v1_5_paloalto_zone_edit["v1.5.paloalto.zone.edit"]
  v1_4_paloalto_tunnel_create["v1.4.paloalto.tunnel.create"] -->|error| v1_vpn_create["v1.vpn.create"]
  v1_5_paloalto_zone_edit["v1.5.paloalto.zone.edit"] -->|next| v1_6_paloalto_rule_create["v1.6.paloalto.rule.create"]
  v1_5_paloalto_zone_edit["v1.5.paloalto.zone.edit"] -->|error| v1_vpn_create["v1.vpn.create"]
  v1_6_paloalto_rule_create["v1.6.paloalto.rule.create"] -->|next| v1_7_paloalto_router_edit["v1.7.paloalto.router.edit"]
  v1_6_paloalto_rule_create["v1.6.paloalto.rule.create"] -->|error| v1_vpn_create["v1.vpn.create"]
  v1_7_paloalto_router_edit["v1.7.paloalto.router.edit"] -->|next| v1_8_paloalto_ipsec_create["v1.8.paloalto.ipsec.create"]
  v1_7_paloalto_router_edit["v1.7.paloalto.router.edit"] -->|error| v1_vpn_create["v1.vpn.create"]
  v1_8_paloalto_ipsec_create["v1.8.paloalto.ipsec.create"] -->|next| v1_9_paloalto_pbf_create["v1.9.paloalto.pbf.create"]
  v1_8_paloalto_ipsec_create["v1.8.paloalto.ipsec.create"] -->|error| v1_vpn_create["v1.vpn.create"]
  v1_9_paloalto_pbf_create["v1.9.paloalto.pbf.create"] -->|next| v1_10_paloalto_ipsec_edit["v1.10.paloalto.ipsec.edit"]
  v1_9_paloalto_pbf_create["v1.9.paloalto.pbf.create"] -->|error| v1_vpn_create["v1.vpn.create"]
  v1_10_paloalto_ipsec_edit["v1.10.paloalto.ipsec.edit"] -->|next| v1_11_paloalto_tunnel_edit["v1.11.paloalto.tunnel.edit"]
  v1_10_paloalto_ipsec_edit["v1.10.paloalto.ipsec.edit"] -->|error| v1_vpn_create["v1.vpn.create"]
  v1_11_paloalto_tunnel_edit["v1.11.paloalto.tunnel.edit"] -->|next| v1_vpn_create["v1.vpn.create"]
  v1_11_paloalto_tunnel_edit["v1.11.paloalto.tunnel.edit"] -->|error| v1_vpn_create["v1.vpn.create"]
```

## Fluxo - VPN Delete

```mermaid
graph TD
  A([Start]) --> v1_1_paloalto_rule_delete["v1.1.paloalto.rule.delete"]
  v1_1_paloalto_rule_delete["v1.1.paloalto.rule.delete"] -->|next| v1_2_paloalto_nat_delete["v1.2.paloalto.nat.delete"]
  v1_1_paloalto_rule_delete["v1.1.paloalto.rule.delete"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_2_paloalto_nat_delete["v1.2.paloalto.nat.delete"] -->|next| v1_3_paloalto_host_edit["v1.3.paloalto.host.edit"]
  v1_2_paloalto_nat_delete["v1.2.paloalto.nat.delete"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_3_paloalto_host_edit["v1.3.paloalto.host.edit"] -->|next| v1_4_paloalto_host_delete["v1.4.paloalto.host.delete"]
  v1_3_paloalto_host_edit["v1.3.paloalto.host.edit"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_4_paloalto_host_delete["v1.4.paloalto.host.delete"] -->|next| v1_5_fortinet_vip_delete["v1.5.fortinet.vip.delete"]
  v1_4_paloalto_host_delete["v1.4.paloalto.host.delete"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_5_fortinet_vip_delete["v1.5.fortinet.vip.delete"] -->|next| v1_6_fortinet_snat_delete["v1.6.fortinet.snat.delete"]
  v1_5_fortinet_vip_delete["v1.5.fortinet.vip.delete"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_6_fortinet_snat_delete["v1.6.fortinet.snat.delete"] -->|next| v1_7_fortinet_ippool_delete["v1.7.fortinet.ippool.delete"]
  v1_6_fortinet_snat_delete["v1.6.fortinet.snat.delete"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_7_fortinet_ippool_delete["v1.7.fortinet.ippool.delete"] -->|next| v1_8_nsxt_vs_delete["v1.8.nsxt.vs.delete"]
  v1_7_fortinet_ippool_delete["v1.7.fortinet.ippool.delete"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_8_nsxt_vs_delete["v1.8.nsxt.vs.delete"] -->|next| v1_9_nsxt_pool_delete["v1.9.nsxt.pool.delete"]
  v1_8_nsxt_vs_delete["v1.8.nsxt.vs.delete"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_9_nsxt_pool_delete["v1.9.nsxt.pool.delete"] -->|next| v1_10_paloalto_pbf_delete["v1.10.paloalto.pbf.delete"]
  v1_9_nsxt_pool_delete["v1.9.nsxt.pool.delete"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_10_paloalto_pbf_delete["v1.10.paloalto.pbf.delete"] -->|next| v1_11_paloalto_ipsec_edit["v1.11.paloalto.ipsec.edit"]
  v1_10_paloalto_pbf_delete["v1.10.paloalto.pbf.delete"] -->|error| v1_vpn_create["v1.vpn.create"]
  v1_11_paloalto_ipsec_edit["v1.11.paloalto.ipsec.edit"] -->|next| v1_12_paloalto_ipsec_delete["v1.12.paloalto.ipsec.delete"]
  v1_11_paloalto_ipsec_edit["v1.11.paloalto.ipsec.edit"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_12_paloalto_ipsec_delete["v1.12.paloalto.ipsec.delete"] -->|next| v1_13_paloalto_router_edit["v1.13.paloalto.router.edit"]
  v1_12_paloalto_ipsec_delete["v1.12.paloalto.ipsec.delete"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_13_paloalto_router_edit["v1.13.paloalto.router.edit"] -->|next| v1_14_paloalto_zone_edit["v1.14.paloalto.zone.edit"]
  v1_13_paloalto_router_edit["v1.13.paloalto.router.edit"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_14_paloalto_zone_edit["v1.14.paloalto.zone.edit"] -->|next| v1_15_paloalto_tunnel_edit["v1.15.paloalto.tunnel.edit"]
  v1_14_paloalto_zone_edit["v1.14.paloalto.zone.edit"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_15_paloalto_tunnel_edit["v1.15.paloalto.tunnel.edit"] -->|next| v1_16_paloalto_tunnel_delete["v1.16.paloalto.tunnel.delete"]
  v1_15_paloalto_tunnel_edit["v1.15.paloalto.tunnel.edit"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_16_paloalto_tunnel_delete["v1.16.paloalto.tunnel.delete"] -->|next| v1_17_paloalto_ike_delete["v1.17.paloalto.ike.delete"]
  v1_16_paloalto_tunnel_delete["v1.16.paloalto.tunnel.delete"] -->|error| v1_vpn_delete["v1.vpn.delete"]
  v1_17_paloalto_ike_delete["v1.17.paloalto.ike.delete"] -->|next| v1_vpn_delete["v1.vpn.delete"]
  v1_17_paloalto_ike_delete["v1.17.paloalto.ike.delete"] -->|error| v1_vpn_delete["v1.vpn.delete"]
```