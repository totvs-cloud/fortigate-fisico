# 🧭 Gestão de Objetos: Addresses, AddressGroups, Services & ServiceGroups

Documento baseado na especificação oficial [pa.espec.json](pa.espec.json). Objetivo: orientar o time de migração focado nos objetos Palo Alto, com exemplos simples e prontos para copiar.

---

## 🌐 Contexto Geral
- **Base URL:** `https://<firewall>/restapi/v10.2`
- **Autenticação:** header `X-PAN-KEY: <api-key>` obtido via endpoint de geração de chave.
- **Formatos:** trabalhar preferencialmente em JSON (`input-format=json`, `output-format=json`).
- **Âmbito:** todos os endpoints exigem `location` (`shared` ou `vsys`) e, quando `location=vsys`, informar `vsys=<nome>`.

---

> 💡 Sempre inclua `name`, `location` e `vsys` nas chamadas que modificam recursos. Para leitura geral (`GET` sem filtro), `name` é opcional.

---

## 🧱 Addresses

### Endpoints
| Operação | Método + Caminho | Uso típico |
| --- | --- | --- |
| Listar | `GET /Objects/Addresses` | Inventariar objetos existentes |
| Criar | `POST /Objects/Addresses` | Inserir host/FQDN/faixa |
| Editar | `PUT /Objects/Addresses` | Ajustar IP, descrição ou tags |
| Excluir | `DELETE /Objects/Addresses` | Remover objetos órfãos |
| Renomear | `POST /Objects/Addresses:rename` | Padronizar nomes antes da migração |

### Campos Obrigatórios
- `entry.@name`: nome único (máx. 63 caracteres, `[0-9a-zA-Z._-]`).
- Um **único** tipo por objeto: `ip-netmask`, `ip-range`, `ip-wildcard` ou `fqdn`.
- `tag.member[]`: até 64 tags, úteis para grupos dinâmicos.

### Modelos de Payload

#### 1. Host / Sub-rede (`ip-netmask`)
```json
{
  "entry": {
    "@name": "TFDNBM_CV5RGT_ate_HST-189.126.152.92",
    "description": "Host remoto visto no fluxo paloalto-host",
    "ip-netmask": "189.126.152.92/32",
    "tag": { "member": ["TFDNBM_CV5RGT_ate"] }
  }
}
```

#### 2. Faixa de IP (`ip-range`)
```json
{
  "entry": {
    "@name": "TDEVOPS_CDEVOPS_range",
    "ip-range": "200.49.58.0-200.49.58.255",
    "description": "Faixa agrupando hosts usados em TDEVOPS",
    "tag": { "member": ["TDEVOPS_CDEVOPS_ate"] }
  }
}
```

#### 3. Wildcard (`ip-wildcard`)
```json
{
  "entry": {
    "@name": "TFCVY2_C6HOGA_geo",
    "ip-wildcard": "181.41.160.0/0.0.0.255",
    "description": "Representa blocos usados na regra TFCVY2",
    "tag": { "member": ["TFCVY2_C6HOGA_ate"] }
  }
}
```

#### 4. FQDN
```json
{
  "entry": {
    "@name": "TFEMP5_CAU7LX_fqdn",
    "fqdn": "vpn.tfemp5.example.com",
    "description": "Mesmo identificador usado pela tag TFEMP5_CAU7LX_ate",
    "tag": { "member": ["TFEMP5_CAU7LX_ate"] }
  }
}
```

### Operações comuns
- **Excluir**: `DELETE https://<fw>/restapi/v10.2/Objects/Addresses?location=vsys&vsys=vsys1&name=TFDNBM_CV5RGT_ate_HST-189.126.152.92`
- **Renomear**: `POST https://<fw>/restapi/v10.2/Objects/Addresses:rename?location=vsys&vsys=vsys1&name=TFDNBM...&newname=TFDNBM..._legacy`
- **Validação rápida**: `GET https://<fw>/restapi/v10.2/Objects/Addresses?name=TFEMP5_CAU7LX_fqdn&location=shared`

---

## 🧩 AddressGroups

### Endpoints
| Operação | Método + Caminho | Uso típico |
| --- | --- | --- |
| Listar | `GET /Objects/AddressGroups` | Mapear grupos usados em políticas |
| Criar | `POST /Objects/AddressGroups` | Agrupar objetos para regras |
| Editar | `PUT /Objects/AddressGroups` | Atualizar membros ou filtros |
| Excluir | `DELETE /Objects/AddressGroups` | Limpar grupos não usados |
| Renomear | `POST /Objects/AddressGroups:rename` | Manter convenção de nomes |

### Tipos Disponíveis
1. **Estático** – lista explícita de objetos ou grupos.
2. **Dinâmico** – filtro baseado em tags (`filter` suporta `AND`, `OR`, parênteses, aspas simples em nomes com hífen).

### Payloads

#### Estático
```json
{
  "entry": {
    "@name": "GRP_External_Clients",
    "description": "Agrupa destinos vistos em rules TDEVOPS",
    "static": {
      "member": [
        "SHARED_HST-181.41.174.74",
        "TFDNBM_CV5RGT_ate_HST-189.126.152.92"
      ]
    },
    "tag": { "member": ["TDEVOPS_CDEVOPS_ate"] }
  }
}
```

#### Dinâmico (Tag-Based)
```json
{
  "entry": {
    "@name": "GRP_DYN_TDEVOPS",
    "dynamic": {
      "filter": "'TDEVOPS_CDEVOPS_ate' AND ('TFEMP5_CAU7LX_ate' OR 'fisico')"
    },
    "tag": { "member": ["TDEVOPS_CDEVOPS_ate"] }
  }
}
```

### Operações comuns
- **Remover membro específico (grupo estático)**: envie um `PUT` com a lista atualizada, por exemplo:
```json
{
  "entry": {
    "@name": "GRP_External_Clients",
    "static": {
      "member": ["SHARED_HST-181.41.174.74"]
    }
  }
}
```
- **Excluir grupo**: `DELETE https://<fw>/restapi/v10.2/Objects/AddressGroups?location=vsys&vsys=vsys1&name=GRP_External_Clients`
- **Listar dependências**: `GET /restapi/v10.2/Policies/SecurityRules?location=vsys&vsys=vsys1&name=TDEVOPS_CDEVOPS_ate-200`

### Cuidados Essenciais
- Grupos podem referenciar outros grupos; mantenha a mesma hierarquia ao migrar.
- Descrições e tags ajudam na revisão posterior das regras.
- Lembrete: filtros dinâmicos consideram **tags dos objetos**. Garanta que elas existam e estejam atualizadas antes de usar filtros.

---

## 🎛 Services

### Endpoints
| Operação | Método + Caminho | Uso típico |
| --- | --- | --- |
| Listar | `GET /Objects/Services` | Inventariar portas customizadas |
| Criar | `POST /Objects/Services` | Registrar TCP/UDP específicos |
| Editar | `PUT /Objects/Services` | Ajustar portas ou timeouts |
| Excluir | `DELETE /Objects/Services` | Remover serviços obsoletos |
| Renomear | `POST /Objects/Services:rename` | Padronizar nomenclatura |

> Referências completas em [pa.espec.json](pa.espec.json#L3320-L3780) e endpoints em [pa.espec.json](pa.espec.json#L52820-L53360).

### Campos Obrigatórios
- `entry.@name`: mesmo padrão dos demais objetos (até 63 caracteres).
- `protocol`: escolha **um** bloco `tcp` ou `udp` por objeto.
- `protocol.<tcp|udp>.port`: lista ou intervalo de portas destino (ex.: `80,443,8000-8010`).
- `protocol.<tcp|udp>.source-port` (opcional): define origem quando necessário.
- `protocol.<tcp|udp>.override`: permite `yes` e customiza `timeout`, `halfclose-timeout`, `timewait-timeout` (TCP) ou `timeout` (UDP).
- `tag.member[]`: mesmas 64 tags possíveis dos outros objetos.

### Payloads de Referência

#### 1. Serviço TCP básico
```json
{
  "entry": {
    "@name": "TCP-63000",
    "description": "Porta registrada pelo micro serviço paloalto-service",
    "protocol": {
      "tcp": {
        "port": "63000"
      }
    },
    "tag": { "member": ["TDEVOPS_CDEVOPS_ate"] }
  }
}
```

#### 2. Serviço UDP com override de timeout
```json
{
  "entry": {
    "@name": "UDP-514",
    "description": "Syslog UDP usado pelos fluxos de regra",
    "protocol": {
      "udp": {
        "port": "514",
        "override": {
          "yes": {
            "timeout": 120
          }
        }
      }
    },
    "tag": { "member": ["TDEVOPS_CDEVOPS_ate"] }
  }
}
```

### Operações comuns
- **Excluir**: `DELETE https://<fw>/restapi/v10.2/Objects/Services?location=shared&name=TCP-63000`
- **Renomear**: `POST https://<fw>/restapi/v10.2/Objects/Services:rename?location=shared&name=UDP-514&newname=UDP-514-legacy`
- **Checar uso**: `GET /restapi/v10.2/Policies/SecurityRules?location=vsys&vsys=vsys1` (filtrar `service.member`).

### Boas Práticas
- Use ranges contínuos sempre que possível para reduzir objetos (ex.: `7000-7020`).
- Quando precisar diferenciar origem/destino, documente via tags ou descrição (não há campo dedicado no objeto).
- Defina timeouts customizados apenas quando houver requerimento funcional claro.
- Para serviços herdados (`location=predefined`), apenas leitura está disponível; crie novos em `shared` ou `vsys` para personalizações.

---

## 🧬 ServiceGroups

### Endpoints
| Operação | Método + Caminho | Uso típico |
| --- | --- | --- |
| Listar | `GET /Objects/ServiceGroups` | Entender blocos de portas reutilizados |
| Criar | `POST /Objects/ServiceGroups` | Agrupar serviços para regras |
| Editar | `PUT /Objects/ServiceGroups` | Atualizar membros |
| Excluir | `DELETE /Objects/ServiceGroups` | Remover grupos não usados |
| Renomear | `POST /Objects/ServiceGroups:rename` | Manter organização |

> Definições em [pa.espec.json](pa.espec.json#L3652-L3780) e endpoints em [pa.espec.json](pa.espec.json#L53220-L53590).

### Estrutura
- `entry.@name`: mesmo padrão de nomenclatura.
- `members.member[]`: lista de serviços ou grupos já existentes (misturar é permitido).
- `tag.member[]`: útil para classificações (ex.: `app-crm`, `tier-core`).

### Payload Exemplo
```json
{
  "entry": {
    "@name": "SGRP_TFEOT4_APP",
    "description": "Serviços citados em regras como TFEOT4",
    "members": {
      "member": ["TCP-10801", "TCP-10802", "TCP-10803"]
    },
    "tag": { "member": ["TFEOT4_CKOYPH_ate"] }
  }
}
```

### Operações comuns
- **Remover serviço do grupo** (enviar membros restantes):
```json
{
  "entry": {
    "@name": "SGRP_TFEOT4_APP",
    "members": {
      "member": ["TCP-10801", "TCP-10802"]
    }
  }
}
```
- **Excluir**: `DELETE https://<fw>/restapi/v10.2/Objects/ServiceGroups?location=shared&name=SGRP_TFEOT4_APP`
- **Renomear**: `POST .../ServiceGroups:rename?name=SGRP_TFEOT4_APP&newname=SGRP_TFEOT4_APP_v2`

### Recomendações
- Revise dependências: um grupo pode referenciar outro; garanta que os objetos base já existam antes do POST.
- Tags ajudam a identificar conjuntos críticos quando for migrar políticas complexas.
- Planeje convenções de nomes (`svc-` para serviços, `grp-` para grupos) para facilitar mapeamentos automatizados.

---

## ⏱️ Schedules

### Endpoints
| Operação | Método + Caminho | Uso típico |
| --- | --- | --- |
| Listar | `GET /Objects/Schedules` | Conferir janelas usadas nas regras |
| Criar | `POST /Objects/Schedules` | Definir políticas com hora/data |
| Editar | `PUT /Objects/Schedules` | Ajustar faixas de tempo |
| Excluir | `DELETE /Objects/Schedules` | Remover janelas antigas |
| Renomear | `POST /Objects/Schedules:rename` | Alinhar convenções |

> Consulte o esquema em [pa.espec.json](pa.espec.json#L14580-L15040) e endpoints em [pa.espec.json](pa.espec.json#L63804-L64190).

### Tipos de Agenda
- **Recurring / Weekly:** até seis faixas `hh:mm-hh:mm` por dia da semana.
- **Recurring / Daily:** listas diárias aplicadas todos os dias.
- **Non-recurring:** intervalo completo `YYYY/MM/DD@hh:mm-YYYY/MM/DD@hh:mm` (bom para mudanças pontuais).

### Campos-chave
- `entry.@name`: até 31 caracteres, siga padrão de tags (ex.: `SCH_TDEVOPS_BUSINESS`).
- `schedule-type`: obrigatório; escolha apenas **um** bloco (`recurring` ou `non-recurring`).
- `description`: detalhe o motivo (janelas de manutenção, horário comercial etc.).

### Payloads inspirados nos fluxos atuais

#### 1. Recorrente semanal (horário comercial)
```json
{
  "entry": {
    "@name": "SCH_TDEVOPS_BUSINESS",
    "description": "Usada pelas regras TDEVOPS para liberar 08h-18h (seg-sex)",
    "schedule-type": {
      "recurring": {
        "weekly": {
          "monday": { "member": ["08:00-18:00"] },
          "tuesday": { "member": ["08:00-18:00"] },
          "wednesday": { "member": ["08:00-18:00"] },
          "thursday": { "member": ["08:00-18:00"] },
          "friday": { "member": ["08:00-18:00"] }
        }
      }
    }
  }
}
```

#### 2. Recorrente diária (janela noturna)
```json
{
  "entry": {
    "@name": "SCH_TFEOT4_MAINT",
    "description": "Janela diária 22h-23h59 para TFEOT4_CKOYPH_ate",
    "schedule-type": {
      "recurring": {
        "daily": {
          "member": ["22:00-23:59"]
        }
      }
    }
  }
}
```

#### 3. Não recorrente (change control)
```json
{
  "entry": {
    "@name": "SCH_TFCVY2_JAN13",
    "description": "Liberação única para atividades em 2026-01-13",
    "schedule-type": {
      "non-recurring": {
        "member": ["2026/01/13@00:00-2026/01/13@23:59"]
      }
    }
  }
}
```

### Boas práticas
- Use prefixos alinhados às tags (`SCH_<TAG>_<contexto>`) para facilitar integrações.
- Documente o fuso horário adotado (ex.: sempre horário de Brasília) e mantenha consistência.
- Antes de remover um schedule, valide dependências em policies (`GET /SecurityRules?name=...`).

### Operações comuns
- **Excluir**: `DELETE https://<fw>/restapi/v10.2/Objects/Schedules?location=vsys&vsys=vsys1&name=SCH_TFCVY2_JAN13`
- **Editar janela**: `PUT` com novo intervalo em `schedule-type` (substitui listas anteriores).
- **Renomear**: `POST .../Schedules:rename?name=SCH_TFEOT4_MAINT&newname=SCH_TFEOT4_MAINT_v2`

---

## 🛡️ Security Rules

### Endpoints
| Operação | Método + Caminho | Uso típico |
| --- | --- | --- |
| Listar | `GET /Policies/SecurityRules` | Auditar políticas vigentes |
| Criar | `POST /Policies/SecurityRules` | Publicar regras L3/L4 |
| Editar | `PUT /Policies/SecurityRules` | Ajustar campos existentes |
| Excluir | `DELETE /Policies/SecurityRules` | Remover regras obsoletas |
| Renomear | `POST /Policies/SecurityRules:rename` | Padronizar nomes |
| Mover | `POST /Policies/SecurityRules:move` | Reordenar prioridades |

> Referencie o schema completo em [pa.espec.json](pa.espec.json#L14780-L15040) e os endpoints em [pa.espec.json](pa.espec.json#L64160-L64640).

### Campos Essenciais
- `entry.@name`: siga o padrão `TAG_contexto-<id>` (ex.: `TDEVOPS_CDEVOPS_ate-200`).
- `from` / `to`: zonas de origem/destino (como `External_Clients-Externo`).
- `source` / `destination`: membros podem ser addresses, grupos ou regiões; use objetos definidos nas seções anteriores.
- `service.member`: referências a Services/ServiceGroups já criados (`TCP-10801`, `SGRP_TFEOT4_APP` etc.).
- `application.member`: frequentemente `any`, mas mantenha explícito.
- `action`: `allow` ou `deny`.
- Campos opcionais relevantes: `description`, `disabled`, `log-setting`, `log-start`, `log-end`, `profile-setting`, `qos`, `hip-profiles`, `target`, `negate-source/destination`, `schedule` (usa objetos de `Schedules`), `tag.member`.

### Estrutura rápida
| Campo | Obrigatório? | Notas |
| --- | --- | --- |
| `from.member[]` | Sim | Zonas de origem; aceita `any`. |
| `to.member[]` | Sim | Zonas de destino. |
| `source.member[]` | Sim | Addresses, AddressGroups, regiões ou `any`. |
| `destination.member[]` | Sim | Mesmo formato de `source`. |
| `service.member[]` | Sim | Lista de Services/ServiceGroups. |
| `application.member[]` | Sim | Normalmente `any`. |
| `action` | Sim | `allow` / `deny`. |
| `log-setting` | Não | Use nomes definidos em Device > Log Settings. |
| `profile-setting` | Não | Referencia perfis (`virus`, `vulnerability`, etc.). |
| `schedule.member[]` | Não | Vincula a objetos de Schedule. |
| `description`, `tag.member[]` | Não | Fortemente recomendados para rastreio. |

### Payloads com base nos fluxos atuais

#### 1. Regra típica L3/L4 (trecho derivado do serviço de load balancer)
```json
{
  "entry": {
    "@name": "TDEVOPS_CDEVOPS_ate-200",
    "from": { "member": ["External_Clients-Externo"] },
    "to": { "member": ["External_Clients-Interno"] },
    "source": { "member": ["BR"] },
    "destination": { "member": ["SHARED_HST-181.41.174.74"] },
    "source-user": { "member": ["any"] },
    "category": { "member": ["any"] },
    "application": { "member": ["any"] },
    "service": { "member": ["TCP-2203", "TCP-2204", "TCP-2202"] },
    "action": "allow",
    "log-start": "yes",
    "log-end": "yes",
    "log-setting": "FWD_ELK_syslog_vsys2",
    "description": "Acesso controlado aos servidores TDEVOPS",
    "disabled": "no",
    "tag": { "member": ["TDEVOPS_CDEVOPS_ate"] }
  }
}
```

#### 2. Regra com múltiplos destinos e profiles
```json
{
  "entry": {
    "@name": "TFEOT4_CKOYPH_ate-1",
    "from": { "member": ["External_Clients-Externo"] },
    "to": { "member": ["External_Clients-Interno"] },
    "source": { "member": ["any"] },
    "destination": { "member": ["SHARED_HST-181.41.163.26"] },
    "service": { "member": ["TCP-10801", "TCP-10802", "TCP-10803"] },
    "application": { "member": ["any"] },
    "action": "allow",
    "negate-source": "no",
    "negate-destination": "no",
    "qos": {
      "marking": {
        "ip-precedence": {
          "member": ["af41"]
        }
      }
    },
    "tag": { "member": ["TFEOT4_CKOYPH_ate"] },
    "profile-setting": {
      "profiles": {
        "virus": { "member": ["default"] },
        "vulnerability": { "member": ["ips_base"] }
      }
    },
    "description": "Regras do app TFEOT4 com inspeção completa"
  }
}
```

#### 3. Regra com Schedule e lista grande de serviços
```json
{
  "entry": {
    "@name": "TFCVY2_C6HOGA_ate-6",
    "from": { "member": ["External_Clients-Externo"] },
    "to": { "member": ["External_Clients-Interno"] },
    "source": { "member": ["BR", "US", "SE", "TW", "PT"] },
    "destination": { "member": ["TFCVY2_C6HOGA_ate_HST-181.41.161.192"] },
    "service": {
      "member": [
        "TCP-2323", "TCP-4060", "TCP-4000", "TCP-4070", "TCP-6800", "TCP-8800",
        "TCP-6900", "TCP-7600", "TCP-8402", "TCP-443", "TCP-8403", "TCP-4010",
        "TCP-4020", "TCP-4030", "TCP-4040", "TCP-4050", "TCP-4080", "TCP-6700",
        "TCP-6000", "TCP-7800"
      ]
    },
    "action": "allow",
    "log-setting": "FWD_ELK_syslog_vsys2",
    "hip-profiles": { "member": ["any"] },
    "tag": { "member": ["TFCVY2_C6HOGA_ate"] },
    "schedule": { "member": ["SCH_TFEOT4_MAINT"] },
    "description": "Regras liberadas apenas na janela SCH_TFEOT4_MAINT"
  }
}
```

### Operações comuns
- **Excluir**: `DELETE https://<fw>/restapi/v10.2/Policies/SecurityRules?location=vsys&vsys=vsys1&name=TDEVOPS_CDEVOPS_ate-200`
- **Edição**: `PUT` com o payload atualizado (ex.: adicionar/remover `service.member`).
- **Remover regra em lote**: iterar lista de nomes via micro serviço, usando o mesmo endpoint `DELETE`.
- **Mover**: `POST https://<fw>/restapi/v10.2/Policies/SecurityRules:move?location=vsys&vsys=vsys1&name=TDEVOPS_CDEVOPS_ate-200&where=before&dst=Drop-Regions-vsys2`
- **Renomear**: `POST .../SecurityRules:rename?name=TFCVY2_C6HOGA_ate-6&newname=TFCVY2_C6HOGA_ate-6-v2`

### Boas práticas
- Garanta que todos os objetos referenciados (addresses, services, schedules) existam antes do POST.
- Centralize `log-setting` e `profile-setting` em constantes para evitar divergência entre políticas.
- Em migrações em massa, exporte as regras via `GET /Policies/SecurityRules?location=vsys&vsys=vsys1` e controle tudo em planilha (colunas: `from`, `to`, `source`, `destination`, `service`, `schedule`, `action`).

---

## 🔁 NAT Rules

### Endpoints
| Operação | Método + Caminho | Uso típico |
| --- | --- | --- |
| Listar | `GET /Policies/NatRules` | Auditar traduções configuradas |
| Criar | `POST /Policies/NatRules` | Publicar SNAT ou DNAT |
| Editar | `PUT /Policies/NatRules` | Ajustar blocos de tradução |
| Excluir | `DELETE /Policies/NatRules` | Limpar regras obsoletas |
| Renomear | `POST /Policies/NatRules:rename` | Organizar convenções de nomes |
| Mover | `POST /Policies/NatRules:move` | Reordenar precedência |

> Consulte o schema completo em [pa.espec.json](pa.espec.json#L15840-L16660) e os endpoints expostos em [pa.espec.json](pa.espec.json#L64633-L64950). Para automação dos fluxos (create/edit/delete), veja [nats.md](nats.md).

### Campos essenciais
- `entry.@name`: sugerimos `NAT_<contexto>_<tipo>` (ex.: `NAT_TDEVOPS_SNAT`).
- `from` / `to`: zonas de origem/destino usadas no roteamento da sessão.
- `source` / `destination`: addresses ou groups já inventariados; `any` quando for genérico.
- `service`: referência única a Service ou ServiceGroup (use `any` quando abranger tudo).
- `nat-type`: `ipv4` (padrão), `nat64` ou `nptv6`.
- `to-interface`: interface de saída após o route lookup; mantenha `any` para herdar da rota.
- `source-translation`: escolha **um** bloco (`dynamic-ip-and-port`, `dynamic-ip` ou `static-ip`).
- `destination-translation` **ou** `dynamic-destination-translation`: responsáveis por DNAT / load balance.
- Campos adicionais frequentes: `active-active-device-binding`, `description`, `tag.member`, `group-tag`, `disabled`.

### Blocos de tradução suportados
| Bloco | Quando usar | Observações |
| --- | --- | --- |
| `dynamic-ip-and-port` | Hide NAT tradicional (PAT) | Pode apontar para `interface-address` ou lista em `translated-address`. |
| `dynamic-ip` | SNAT por pool sem tradução de porta | Aceita `fallback` para interface/address alternativo. |
| `static-ip` | 1:1 NAT com opção `bi-directional` | Ideal para servidores DMZ publicados em ambos os sentidos. |
| `destination-translation` | DNAT simples | Suporta `translated-port` e `dns-rewrite`. |
| `dynamic-destination-translation` | Balanceamento round-robin de VIPs | Escolha `distribution` (`round-robin`, `ip-hash`, etc.). |

### Payloads de referência

#### 1. SNAT com PAT na interface
```json
{
  "entry": {
    "@name": "NAT_TDEVOPS_SNAT",
    "from": { "member": ["External_Clients-Interno"] },
    "to": { "member": ["External_Clients-Externo"] },
    "source": { "member": ["GRP_Internal_Apps"] },
    "destination": { "member": ["any"] },
    "service": "any",
    "nat-type": "ipv4",
    "to-interface": "ethernet1/1",
    "source-translation": {
      "dynamic-ip-and-port": {
        "interface-address": { "interface": "ethernet1/1" }
      }
    },
    "description": "SNAT das aplicações internas para o link principal",
    "tag": { "member": ["TDEVOPS_CDEVOPS_ate"] }
  }
}
```

#### 2. DNAT com log e profiles alinhados às regras
```json
{
  "entry": {
    "@name": "NAT_TFCVY2_DNAT",
    "from": { "member": ["External_Clients-Externo"] },
    "to": { "member": ["External_Clients-Interno"] },
    "source": { "member": ["any"] },
    "destination": { "member": ["SHARED_HST-181.41.174.74"] },
    "service": "TCP-63000",
    "to-interface": "ethernet1/3",
    "source-translation": {
      "static-ip": {
        "static-ip": {
          "translated-address": "189.126.152.200",
          "bi-directional": "no"
        }
      }
    },
    "destination-translation": {
      "translated-address": "10.50.10.15",
      "translated-port": 63000
    },
    "log-setting": "FWD_ELK_syslog_vsys2",
    "tag": { "member": ["TFCVY2_C6HOGA_ate"] },
    "description": "DNAT do host publicado para TCP-63000"
  }
}
```

#### 3. VIP dinâmico com pool e distribuição round-robin
```json
{
  "entry": {
    "@name": "NAT_TFEOT4_VIP",
    "from": { "member": ["External_Clients-Externo"] },
    "to": { "member": ["External_Clients-Interno"] },
    "source": { "member": ["any"] },
    "destination": { "member": ["GRP_External_Clients"] },
    "service": "SGRP_TFEOT4_APP",
    "to-interface": "ethernet1/3",
    "dynamic-destination-translation": {
      "translated-address": "GRP_TFEOT4_VIP_POOL",
      "translated-port": 10801,
      "distribution": "round-robin"
    },
    "tag": { "member": ["TFEOT4_CKOYPH_ate"] },
    "group-tag": "app-tfeot4",
    "description": "VIP balanceado para os serviços TCP-1080x"
  }
}
```

### Operações comuns
- **Excluir**: `DELETE https://<fw>/restapi/v10.2/Policies/NatRules?location=vsys&vsys=vsys1&name=NAT_TDEVOPS_SNAT`
- **Editar**: `PUT` com o payload completo contendo as alterações de tradução.
- **Renomear**: `POST .../Policies/NatRules:rename?name=NAT_TFCVY2_DNAT&newname=NAT_TFCVY2_DNAT_v2`
- **Mover**: `POST .../Policies/NatRules:move?name=NAT_TFEOT4_VIP&where=before&dst=NAT_CleanUp`

### Boas práticas
- Valide dependências com as Security Rules correspondentes; a ordem relativa das NAT Rules impacta diretamente o match.
- Sempre crie os objetos referenciados (addresses, service, tags) antes do POST e mantenha a mesma `location`/`vsys`.
- Documente `nat-type` e `to-interface` nas planilhas de migração para evitar divergência entre ambientes.
- Utilize os fluxos descritos em [nats.md](nats.md) para manter a sequência service ➜ host ➜ rule ➜ nat consistente.


## 🌉 IKE Gateways

### Endpoints
| Operação | Método + Caminho | Uso típico |
| --- | --- | --- |
| Listar | `GET /Network/IKEGateways` | Validar status e parâmetros de VPNs |
| Criar | `POST /Network/IKEGateways` | Provisionar gateways (psk/cert) |
| Editar | `PUT /Network/IKEGateways` | Ajustar peer, crypto ou DPD |
| Excluir | `DELETE /Network/IKEGateways` | Remover gateways obsoletos |
| Renomear | `POST /Network/IKEGateways:rename` | Alinhar convenções por TAG |

> Estrutura completa em [pa.espec.json](pa.espec.json#L27100-L27980); fluxos de automação em [ike-gateway.md](ike-gateway.md).

### Campos essenciais
- `entry.@name`: siga padrão `TAG_contexto-<id>` (ex.: `TEZDNB_CGNRO8_ate-8`).
- `peer-address`: escolha entre IP fixo, FQDN ou `dynamic` (WAN sem IP estático).
- `local-address.interface` + `ip`/`floating-ip`: interface usada no IKE (loopback ou física).
- `authentication`: `pre-shared-key` (com `key`) ou `certificate` (exige `local-certificate` + `certificate-profile`).
- `protocol.version`: `ikev1`, `ikev2`, `ikev2-preferred`; cada bloco referencia `ike-crypto-profile` e `dpd`.
- `protocol-common`: habilita NAT-T, `passive-mode` e `fragmentation`.
- Identificadores: `peer-id`/`local-id` suportam tipos `ipaddr`, `fqdn`, `ufqdn`, `dn` (conforme regex do schema).

### Opções de endereço do peer
| Tipo | Chave | Quando usar |
| --- | --- | --- |
| IP estático | `peer-address.ip` | Parceiros com endereço fixo (mais comum nos fluxos atuais). |
| FQDN | `peer-address.fqdn` | Parceiros dinâmicos que anunciam hostname. |
| Dinâmico | `peer-address.dynamic` | Túneis onde o peer inicia de IP variável (ex.: filiais sem IP fixo). |

### Modos de autenticação
| Tipo | Campos obrigatórios | Observações |
| --- | --- | --- |
| Pre-shared key | `authentication.pre-shared-key.key` | Aceita até 255 caracteres; combine com `peer-id` para restringir. |
| Certificado | `local-certificate.name`, `certificate-profile` | Habilite `hash-and-url` se usar distribuição HTTP; suporte a `strict-validation-revocation`. |

### Payloads de referência

#### 1. Gateway com PSK (derivado do micro serviço)
```json
{
  "entry": {
    "@name": "TEZDNB_CGNRO8_ate-8",
    "peer-address": { "ip": "200.49.58.170" },
    "local-address": { "interface": "loopback.3", "ip": "181.41.161.254/32" },
    "peer-id": { "type": "ipaddr", "id": "200.49.58.170" },
    "local-id": { "type": "ipaddr", "id": "181.41.161.254" },
    "authentication": {
      "pre-shared-key": { "key": "j,\"8jT~)GJbUt)Wm{g3]" }
    },
    "protocol": {
      "version": "ikev2-preferred",
      "ikev1": {
        "exchange-mode": "auto",
        "ike-crypto-profile": "Totvs_default",
        "dpd": { "enable": "yes" }
      },
      "ikev2": {
        "ike-crypto-profile": "Totvs_default",
        "dpd": { "enable": "yes" }
      }
    },
    "protocol-common": {
      "nat-traversal": { "enable": "no" },
      "fragmentation": { "enable": "no" }
    },
    "comment": "Gateway criado pelo fluxo ike-gateway"
  }
}
```

#### 2. Gateway com certificado e NAT-T
```json
{
  "entry": {
    "@name": "TFCVY2_cert_vpn",
    "peer-address": { "fqdn": "vpn.partner.example.com" },
    "local-address": { "interface": "ethernet1/3" },
    "authentication": {
      "certificate": {
        "local-certificate": {
          "name": "corp-gw-cert",
          "hash-and-url": {
            "enable": "yes",
            "base-url": "http://certs.internal.example.com/ike"
          }
        },
        "certificate-profile": "corp-profile",
        "use-management-as-source": "yes",
        "strict-validation-revocation": "yes"
      }
    },
    "protocol": {
      "version": "ikev2",
      "ikev2": {
        "ike-crypto-profile": "aes256-sha384-dh14",
        "require-cookie": "no",
        "dpd": { "enable": "yes", "interval": 10 }
      }
    },
    "protocol-common": {
      "nat-traversal": { "enable": "yes", "keep-alive-interval": 20 },
      "passive-mode": "no",
      "fragmentation": { "enable": "yes" }
    },
    "peer-id": { "type": "fqdn", "id": "vpn.partner.example.com" },
    "local-id": { "type": "fqdn", "id": "vpn.local.example.com" },
    "tag": { "member": ["TFCVY2_C6HOGA_ate"] }
  }
}
```

### Operações comuns
- **Excluir**: `DELETE https://<fw>/restapi/v10.2/Network/IKEGateways?location=vsys&vsys=vsys1&name=TEZDNB_CGNRO8_ate-8`
- **Editar**: `PUT` incluindo o bloco completo alterado (ex.: trocar `ike-crypto-profile`).
- **Renomear**: `POST .../Network/IKEGateways:rename?name=TFCVY2_cert_vpn&newname=TFCVY2_cert_vpn_v2`
- **Rotina de criação**: seguir a ordem `service ➜ host ➜ rule ➜ ike-gateway` mostrada em [ike-gateway.md](ike-gateway.md#fluxo---ike-gateway-create).

### Boas práticas
- Garanta a existência dos `ike-crypto-profile` referenciados (`Network/IkeCryptoProfiles` no schema).
- Documente `peer-id`/`local-id` seguindo o padrão acordado com o peer para evitar incompatibilidades.
- Ative `dpd` e `nat-traversal` conforme as características do enlace para evitar quedas falsas.
- Em certificação, alinhe `certificate-profile` aos parâmetros de CRL/OCSP exigidos pelo SOC.
- Mantenha senhas PSK fora do Git e injete via pipeline/secret manager, apesar do payload de exemplo mostrar valores reais.


## 🔒 IKE Crypto Profiles

### Endpoints
| Operação | Método + Caminho | Uso típico |
| --- | --- | --- |
| Listar | `GET /Network/IkeCryptoProfiles` | Auditar propostas fase 1 reutilizadas |
| Criar | `POST /Network/IkeCryptoProfiles` | Padronizar cifragem/integração |
| Editar | `PUT /Network/IkeCryptoProfiles` | Ajustar algoritmos e lifetime |
| Excluir | `DELETE /Network/IkeCryptoProfiles` | Limpar perfis sem uso |

> Schema completo em [pa.espec.json](pa.espec.json#L27100-L27540).

### Campos essenciais
- `entry.@name`: até 31 caracteres; sugerir prefixos `IKEP_<contexto>`.
- `encryption.member[]`: selecione algoritmos compatíveis (`aes-128-cbc`, `aes-256-gcm`, etc.).
- `hash.member[]`: `sha256` é o padrão atual; evite `md5/non-auth` em projetos novos.
- `dh-group.member[]`: escolha múltiplos grupos para fallback (ex.: `group14`, `group19`).
- `lifetime`: declare unidade única (`seconds`, `minutes`, `hours`, `days`).
- `authentication-multiple`: somente para IKEv2; define quantas vezes o reauth ocorre dentro do lifetime.

### Payloads

#### Perfil padrão AES256/SHA256
```json
{
  "entry": {
    "@name": "IKEP_TOTVS_default",
    "encryption": { "member": ["aes-256-cbc", "aes-128-cbc"] },
    "hash": { "member": ["sha256"] },
    "dh-group": { "member": ["group14", "group19"] },
    "lifetime": { "hours": 8 },
    "authentication-multiple": 0
  }
}
```

#### Perfil com AES-GCM e reautenticação v2
```json
{
  "entry": {
    "@name": "IKEP_TFCVY2_highsec",
    "encryption": { "member": ["aes-256-gcm"] },
    "hash": { "member": ["sha384"] },
    "dh-group": { "member": ["group19", "group20"] },
    "lifetime": { "minutes": 90 },
    "authentication-multiple": 2
  }
}
```

### Boas práticas
- Reuse nomes entre gateways e perfis para facilitar troubleshooting (`gateway` referencia `ike-crypto-profile`).
- Agrupe algoritmos em ordem de preferência do mais forte para o mais compatível.
- Ajuste `authentication-multiple` somente quando o parceiro exigir reauth explícita (IKEv2).


## 🛰️ IPSec Tunnels

### Endpoints
| Operação | Método + Caminho | Uso típico |
| --- | --- | --- |
| Listar | `GET /Network/IpsecTunnels` | Conferir túneis (phase2) |
| Criar | `POST /Network/IpsecTunnels` | Publicar auto-key ou manual |
| Editar | `PUT /Network/IpsecTunnels` | Ajustar proxy-id, monitor, interface |
| Excluir | `DELETE /Network/IpsecTunnels` | Limpar túneis desativados |

> Veja o schema em [pa.espec.json](pa.espec.json#L27980-L28900) e os fluxos em [nats.md](nats.md#serviços-envolvidos) quando o túnel faz parte de migrações automatizadas.

### Campos essenciais (auto-key)
- `entry.@name`: mantenha coerência com o nome do VPN ID (`VPN_<tag>_<id>`).
- `tunnel-interface`: interface lógico (`tunnel.X`) já criado em Network Interfaces.
- `anti-replay`: habilitado por padrão; só desative se o peer não suportar.
- `tunnel-monitor`: configure `enable`, `destination-ip`, `profile` quando usar HA com failover por monitoramento.
- Bloco `auto-key`:
  - `ike-gateway.entry[]`: lista de gateways fase 1 associados (suporta HA ativo/backup).
  - `ipsec-crypto-profile`: referência a `Network/IpsecCryptoProfiles` (fase2).
  - `proxy-id.entry[]`: define local/remote/sub-redes e protocolo exigidos pelo peer (IKEv1).
  - `proxy-id-v6`: versão IPv6 das mesmas entradas.

### Demais modos
- `manual-key`: usado em ambientes legados; exige `peer-address`, `local-address`, SPI local/remoto e chaves ESP/AH definidas manualmente.
- `global-protect-satellite`: fluxo específico para satélites; inclui `portal-address`, publicação de rotas e certificados.

### Payloads

#### 1. Túnel auto-key com proxy-id múltiplos
```json
{
  "entry": {
    "@name": "VPN_TDEVOPS_AUTO",
    "tunnel-interface": "tunnel.100",
    "tunnel-monitor": {
      "enable": "yes",
      "destination-ip": "10.10.10.1",
      "tunnel-monitor-profile": "wait-recover"
    },
    "auto-key": {
      "ike-gateway": {
        "entry": [
          { "@name": "TEZDNB_CGNRO8_ate-8" },
          { "@name": "TEZDNB_CGNRO8_ate-8-bkp" }
        ]
      },
      "ipsec-crypto-profile": "IPSECP_TDEVOPS_default",
      "proxy-id": {
        "entry": [
          {
            "@name": "proxy_TDEVOPS_app",
            "local": "10.50.10.0/24",
            "remote": "172.16.0.0/24",
            "protocol": { "any": {} }
          },
          {
            "@name": "proxy_TDEVOPS_mgmt",
            "local": "10.60.20.0/24",
            "remote": "172.16.20.0/24",
            "protocol": {
              "tcp": { "local-port": 0, "remote-port": 0 }
            }
          }
        ]
      }
    }
  }
}
```

#### 2. Túnel manual-key (legado)
```json
{
  "entry": {
    "@name": "VPN_LEGACY_MANUAL",
    "tunnel-interface": "tunnel.50",
    "manual-key": {
      "peer-address": { "ip": "198.51.100.30" },
      "local-address": { "interface": "ethernet1/3", "ip": "189.126.152.254" },
      "local-spi": "00001000",
      "remote-spi": "00001001",
      "esp": {
        "authentication": { "sha256": { "key": "aaaaaaaa-bbbbbbbb-cccccccc-dddddddd-eeeeeeee-ffffffff-11111111-22222222" } },
        "encryption": {
          "algorithm": "aes-128-cbc",
          "key": "aaaaaaaa-bbbbbbbb-cccccccc-dddddddd"
        }
      }
    }
  }
}
```

### Boas práticas
- Para IKEv2, prefira `proxy-id` apenas quando o peer exigir; caso contrário utilize selectors automáticos (`any`).
- Nomeie `tunnel-interface` com padrão consistente (ex.: `tunnel.100`) para simplificar troubleshooting.
- Monitore `tunnel-monitor.destination-ip` em um host realmente disponível; evite endereços virtuais que podem cair junto com o túnel.
- Em clusters, lembre-se de configurar `floating-ip` quando o enlace termina em interface HA ativa/ativa.


## 🔐 IPSec Crypto Profiles (Phase 2)

### Endpoints
| Operação | Método + Caminho | Uso típico |
| --- | --- | --- |
| Listar | `GET /Network/IpsecCryptoProfiles` | Auditar proposals usados em túneis fase 2 |
| Criar | `POST /Network/IpsecCryptoProfiles` | Definir cifra/autenticação aplicadas ao tráfego protegido |
| Editar | `PUT /Network/IpsecCryptoProfiles` | Ajustar algoritmos, PFS e lifetime |
| Excluir | `DELETE /Network/IpsecCryptoProfiles` | Remover perfis sem uso |

> Schema detalhado em [pa.espec.json](pa.espec.json#L49300-L49740).

### Modos suportados
| Tipo | Descrição | Campos obrigatórios |
| --- | --- | --- |
| ESP | Usado na maioria dos túneis; exige `encryption` e `authentication` | `esp.encryption.member[]`, `esp.authentication.member[]`, `dh-group`, `lifetime` |
| AH | Somente autenticação, sem cifragem | `ah.authentication.member[]`, `dh-group`, `lifetime` |

### Campos essenciais
- `entry.@name`: até 31 caracteres; use prefixos como `IPSECP_<contexto>`.
- `esp.encryption.member[]`: escolha entre `3des`, `aes-{128,192,256}-cbc`, `aes-128-ccm`, `aes-{128,256}-gcm`, `null`.
- `esp.authentication.member[]` ou `ah.authentication.member[]`: combina `sha1`, `sha256`, `sha384`, `sha512` etc.
- `dh-group`: define PFS (`no-pfs`, `group1/2/5/14/15/16/19/20/21`).
- `lifetime`: escolha **uma** unidade (`seconds`, `minutes`, `hours`, `days`).
- `lifesize`: limite opcional por volume (`kb`, `mb`, `gb`, `tb`).

### Payloads de referência

#### 1. ESP com AES-GCM e PFS forte
```json
{
  "entry": {
    "@name": "IPSECP_TDEVOPS_default",
    "esp": {
      "encryption": { "member": ["aes-256-gcm", "aes-128-cbc"] },
      "authentication": { "member": ["sha512", "sha256"] }
    },
    "dh-group": "group19",
    "lifetime": { "hours": 1 },
    "lifesize": { "gb": 50 }
  }
}
```

#### 2. AH-only para integridade
```json
{
  "entry": {
    "@name": "IPSECP_AH_integrity",
    "ah": {
      "authentication": { "member": ["sha256"] }
    },
    "dh-group": "group14",
    "lifetime": { "minutes": 30 }
  }
}
```

### Operações comuns
- **Excluir**: `DELETE https://<fw>/restapi/v10.2/Network/IpsecCryptoProfiles?location=vsys&vsys=vsys1&name=IPSECP_TDEVOPS_default`
- **Editar**: `PUT` reutilizando o payload completo e ajustando apenas algoritmos/lifetime.
- **Padronizar**: combine `GET` + planilha para garantir mesma lista de perfis entre VSYS.

### Boas práticas
- Sempre alinhe os algoritmos às capacidades do parceiro antes de publicar o perfil.
- `aes-256-gcm` com `sha256` cobre requisitos de conformidade modernos, mas valide se o peer aceita GCM.
- Utilize `lifesize` apenas quando necessário para reduzir renegociações frequentes.
- Documente onde cada `IpsecCryptoProfile` é referenciado (`Network/IpsecTunnels`) para evitar exclusões acidentais.


## 🌐 Tunnel Interfaces

### Endpoints
| Operação | Método + Caminho | Uso típico |
| --- | --- | --- |
| Listar | `GET /Network/Interfaces/Tunnel` | Inventariar interfaces lógicas usadas por túneis |
| Criar | `POST /Network/Interfaces/Tunnel` | Provisionar interfaces `tunnel.X` |
| Editar | `PUT /Network/Interfaces/Tunnel` | Ajustar IP, MTU ou atributos IPv6 |
| Excluir | `DELETE /Network/Interfaces/Tunnel` | Remover interfaces não utilizadas |

> Estrutura em [pa.espec.json](pa.espec.json#L30300-L30780); use [network-tunnel-monitor](pa.espec.json#L30729-L30880) para perfis de monitoramento.

### Campos essenciais
- `entry.@name`: numérico (`1-9999`) para subinterfaces; em Panorama use descrições coerentes.
- `df-ignore`: permite fragmentar pacotes mesmo com DF=1 (default `no`).
- `mtu`: 576-9216 (dependendo do Jumbo Frame global).
- `ip.entry[]`: permite múltiplos endereços/objetos (IPv4) associados ao tunnel.
- `ipv6.enabled` + `ipv6.address.entry[]`: habilita pilha dupla no túnel.
- `interface-management-profile`: associa profiles de gestão (ping, https, etc.).
- `netflow-profile`: referencia `ServerProfile/Netflow` para exportação de fluxos.
- `bonjour`, `link-tag`, `comment`: campos opcionais para serviços auxiliares e organização.

### Payloads

#### 1. Tunnel básico com IPv4 e monitoramento
```json
{
  "entry": {
    "@name": "tunnel.100",
    "df-ignore": "no",
    "mtu": 1500,
    "ip": {
      "entry": [
        { "@name": "10.50.10.1/30" },
        { "@name": "OBJ_TUNNEL100_IP" }
      ]
    },
    "interface-management-profile": "mgmt-tun",
    "netflow-profile": "NF_default",
    "comment": "Interface usada pelos VPN_TDEVOPS_AUTO"
  }
}
```

#### 2. Tunnel com IPv6 e serviços Bonjour
```json
{
  "entry": {
    "@name": "200",
    "df-ignore": "yes",
    "mtu": 1400,
    "ipv6": {
      "enabled": "yes",
      "interface-id": "EUI-64",
      "address": {
        "entry": [
          {
            "@name": "2001:db8:100::1/64",
            "enable-on-interface": "yes"
          }
        ]
      }
    },
    "bonjour": {
      "enable": "yes",
      "ttl-check": "yes",
      "group-id": 10
    },
    "link-tag": "vpn-core",
    "comment": "Subinterface IPv6 para túneis de parceiros"
  }
}
```

### Operações comuns
- **Excluir**: `DELETE https://<fw>/restapi/v10.2/Network/Interfaces/Tunnel?location=vsys&vsys=vsys1&name=tunnel.100`
- **Atualizar IP**: `PUT` substituindo `ip.entry` completo para evitar entradas duplicadas.
- **Provisionar em lote**: gere lista de `tunnel.X` no Panorama e aplique via `POST` com payloads em sequência.

### Boas práticas
- Reserve faixas de numeração (ex.: `tunnel.100-199` para filiais) para facilitar troubleshooting.
- Ajuste `mtu` considerando sobrecarga IPSec/Gre; 1400 é valor seguro em cenários com encapsulamento extra.
- Para IPv6, valide se o peer usa `interface-id` customizado antes de alterar `EUI-64`.
- Utilize `comment` e `link-tag` para identificar rapidamente a qual VPN cada interface pertence.


## 🧭 Virtual Routers

### Endpoints
| Operação | Método + Caminho | Uso típico |
| --- | --- | --- |
| Listar | `GET /Network/VirtualRouters` | Auditar tabelas estáticas e associações de interface |
| Criar | `POST /Network/VirtualRouters` | Publicar roteadores virtuais por VSYS |
| Editar | `PUT /Network/VirtualRouters` | Atualizar interfaces, rotas ou perfis de monitoramento |
| Excluir | `DELETE /Network/VirtualRouters` | Remover VRs descontinuados |
| Renomear | `POST /Network/VirtualRouters:rename` | Adequar padrão de nomenclatura |

> Schema completo em [pa.espec.json](pa.espec.json#L30740-L31880); o endpoint exposto acima usa `location`/`vsys` iguais ao restante das APIs de Network.

### Campos essenciais
- `entry.@name`: até 31 caracteres; mantenha o mesmo identificador usado nos diagramas de roteamento.
- `interface.member[]`: lista de interfaces Layer3/tunnel anexadas ao VR; um membro pode existir em apenas um VR.
- `routing-table.ip.static-route.entry[]`: tabela de rotas IPv4; cada rota define `destination`, `interface` (ou `nexthop.next-vr`) e `nexthop` (`ip-address`, `fqdn`, `discard`).
- `routing-table.ipv6.static-route.entry[]`: versão IPv6, com suporte a `nexthop.ipv6-address` e `fqdn`.
- `admin-dist`/`metric`: controlam preferência entre rotas iguais; respeite faixas do schema (10-240 e 1-65535).
- `route-table`: escolha entre `unicast`, `multicast`, `both` ou `no-install` para forçar instalação seletiva.
- `bfd.profile`: vincula perfis de Bidirectional Forwarding Detection (`Network/BfdNetworkProfiles`).
- `path-monitor`: habilita ping baseado em origem/destino, com `hold-time`, `failure-condition` e múltiplos destinos para failover.
- `multicast`: configurações PIM/IGMP, interface-groups, permissões ASM/SSM e RP estático.

### Tipos de next-hop suportados
| Tipo | Bloco | Observações |
| --- | --- | --- |
| IP | `nexthop.ip-address` (IPv4) / `ipv6-address` | Cenário padrão; suporta objetos com `x-panMultiple` para resolver nomes de address objects.
| FQDN | `nexthop.fqdn` | A rota resolve dinamicamente o hostname; útil quando o gateway é publicado por DNS interno.
| Discard | `nexthop.discard` | Blackhole controlado (null route) para conter tráfego não desejado.
| Encadeado | `nexthop.next-vr` | Encaminha para outro Virtual Router quando separam-se domínios de roteamento.

### Payloads de referência

#### 1. VR com rotas IPv4, BFD e path-monitor
```json
{
  "entry": {
    "@name": "VR_TDEVOPS_CORE",
    "interface": { "member": ["ethernet1/3", "tunnel.100", "loopback.3"] },
    "routing-table": {
      "ip": {
        "static-route": {
          "entry": [
            {
              "@name": "default-internet",
              "destination": "0.0.0.0/0",
              "interface": "ethernet1/3",
              "nexthop": { "ip-address": "200.200.200.1" },
              "admin-dist": 10,
              "metric": 10,
              "bfd": { "profile": "BFD_CORE" },
              "path-monitor": {
                "enable": "yes",
                "failure-condition": "any",
                "hold-time": 2,
                "monitor-destinations": {
                  "entry": [
                    {
                      "@name": "default",
                      "source": "181.41.161.254",
                      "destination": "8.8.8.8",
                      "interval": 3,
                      "count": 5
                    }
                  ]
                }
              }
            },
            {
              "@name": "vpn-devops",
              "destination": "10.60.20.0/24",
              "interface": "tunnel.100",
              "nexthop": { "next-vr": "VR_TUNNELS" },
              "route-table": { "unicast": {} }
            }
          ]
        }
      }
    }
  }
}
```

#### 2. VR dual-stack com multicast e rota IPv6 específica
```json
{
  "entry": {
    "@name": "VR_TFCVY2_MCAST",
    "interface": { "member": ["ethernet1/5", "ae1.200", "tunnel.200"] },
    "routing-table": {
      "ipv6": {
        "static-route": {
          "entry": [
            {
              "@name": "default-v6",
              "destination": "::/0",
              "interface": "ethernet1/5",
              "nexthop": { "ipv6-address": "2001:db8:ffff::1" },
              "metric": 50,
              "route-table": { "no-install": {} }
            }
          ]
        }
      }
    },
    "multicast": {
      "enable": "yes",
      "interface-group": {
        "entry": [
          {
            "@name": "mc-core",
            "interface": { "member": ["ae1.200", "tunnel.200"] },
            "igmp": { "enable": "yes", "version": "3", "immediate-leave": "no" },
            "pim": { "enable": "yes", "hello-interval": 30 }
          }
        ]
      },
      "rp": {
        "local-rp": {
          "static-rp": {
            "interface": "loopback.10",
            "address": "192.0.2.10",
            "group-addresses": {
              "entry": [
                { "@name": "ASM_CORP", "group-address": "239.10.0.0/16" }
              ]
            }
          }
        }
      }
    }
  }
}
```

### Operações comuns
- **Excluir**: `DELETE https://<fw>/restapi/v10.2/Network/VirtualRouters?location=vsys&vsys=vsys1&name=VR_TDEVOPS_CORE`
- **Editar rota específica**: `PUT` com o bloco `routing-table` completo garantindo que entradas não fiquem duplicadas.
- **Renomear**: `POST .../Network/VirtualRouters:rename?name=VR_TFCVY2_MCAST&newname=VR_TFCVY2_MCAST_v2`
- **Mover interfaces entre VRs**: remova a interface do VR antigo antes de adicioná-la ao novo para evitar falhas de validação.

### Boas práticas
- Versione planilhas de rotas juntamente com os VRs para evitar divergência entre Panorama e firewalls standalone.
- Utilize `bfd` apenas onde os peers suportam BFD; caso contrário mantenha `None` para reduzir falsos positivos.
- Configure `path-monitor` apenas em rotas críticas (default ou túneis WAN) e valide se a origem tem IP válido.
- Separe VRs por domínio de roteamento (prod, parceiros, DMZ) e use `nexthop.next-vr` para interligá-los de forma controlada.
- Revisite `multicast.interface-group` sempre que adicionar interfaces Layer2/3 para manter PIM/IGMP consistentes.


## 🌀 Loopback Interfaces

### Endpoints
| Operação | Método + Caminho | Uso típico |
| --- | --- | --- |
| Listar | `GET /Network/LoopbackInterfaces` | Inventariar loopbacks usados como origem lógica |
| Criar | `POST /Network/LoopbackInterfaces` | Provisionar `loopback.X` para IKE, GP, serviços |
| Editar | `PUT /Network/LoopbackInterfaces` | Ajustar IPs, MTU ou perfis associados |
| Excluir | `DELETE /Network/LoopbackInterfaces` | Remover loopbacks obsoletos |

> Estrutura completa em [pa.espec.json#L29165-L29830](pa.espec.json#L29165-L29830); as operações seguem o mesmo `location`/`vsys` usado para outros objetos Network.

### Campos essenciais
- `entry.@name`: obrigatório no formato `loopback.<id>` (1-9999); mantenha alinhado ao diagrama/planilha.
- `df-ignore`: permite fragmentar pacotes mesmo com DF=1; normalmente `no`.
- `mtu`: 576-9216 dependendo do modo Jumbo Frame.
- `adjust-tcp-mss`: habilite quando o loopback será origem de sessões encapsuladas (GRE/IPSec) e precisa reduzir MSS.
- `ip.entry[]`: lista de hosts ou objetos; **não** aceita máscara explícita (o loopback já opera como /32).
- `ipv6.enabled` + `ipv6.address.entry[]`: habilita pilha dupla; cada entrada pode definir `enable-on-interface`, `prefix` e `anycast`.
- `interface-management-profile`: controla quais serviços (ping, https, ssh) respondem nesse endereço lógico.
- `netflow-profile`: exporta fluxos originados do loopback.
- `comment`: útil para indicar uso (IKE, GP, BGP, etc.).

### Particularidades de endereçamento
| Item | Detalhe |
| --- | --- |
| IPv4 | Informe apenas o host (ex.: `181.41.161.254`) ou um address object; máscaras não são aceitas conforme schema. |
| IPv6 interface-id | Default `EUI-64`; pode ser sobrescrito por um valor hex de 16 caracteres quando o parceiro exige ID fixo. |
| IPv6 addresses | Cada `entry` aceita `anycast` e `prefix`; `enable-on-interface=no` mantém o endereço apenas como objeto referenciável. |

### Payloads de referência

#### 1. Loopback IPv4 para origem de túneis
```json
{
  "entry": {
    "@name": "loopback.3",
    "df-ignore": "no",
    "mtu": 1500,
    "ip": {
      "entry": [
        { "@name": "181.41.161.254" },
        { "@name": "OBJ_LOOPBACK3" }
      ]
    },
    "interface-management-profile": "mgmt-loopbacks",
    "netflow-profile": "NF_default",
    "comment": "Origem IKE para TEZDNB gateways"
  }
}
```

#### 2. Loopback dual-stack com MSS adjust
```json
{
  "entry": {
    "@name": "loopback.200",
    "df-ignore": "yes",
    "mtu": 1400,
    "adjust-tcp-mss": {
      "enable": "yes",
      "ipv4-mss-adjustment": 80,
      "ipv6-mss-adjustment": 100
    },
    "ip": {
      "entry": [
        { "@name": "203.0.113.200" }
      ]
    },
    "ipv6": {
      "enabled": "yes",
      "interface-id": "EUI-64",
      "address": {
        "entry": [
          {
            "@name": "2001:db8:200::1",
            "enable-on-interface": "yes"
          }
        ]
      }
    },
    "comment": "Loopback dual-stack para portal GP"
  }
}
```

### Operações comuns
- **Excluir**: `DELETE https://<fw>/restapi/v10.2/Network/LoopbackInterfaces?location=vsys&vsys=vsys1&name=loopback.3`
- **Atualizar IPs**: `PUT` substituindo todo o bloco `ip.entry` para evitar resíduos de objetos antigos.
- **Clonar**: combine `GET` + ajuste de payload para replicar padrões entre VSYS (não há endpoint de rename nesse recurso).

### Boas práticas
- Use loopbacks como origem de IKE/IPSec, BGP e GlobalProtect para manter sessões independentes das interfaces físicas.
- Publique rotas /32 correspondentes nos Virtual Routers ou dinamicamente (OSPF/BGP) para garantir reachability.
- Restrinja serviços expostos via `interface-management-profile` e registre o endereço em monitoramentos críticos.
- Ajuste `mtu`/`adjust-tcp-mss` somente quando o loopback participa de túneis com overhead adicional; para usos puramente lógicos mantenha o default.
- Documente associações (túnel, portal, serviço) no `comment` e na planilha de migração para facilitar troubleshooting futuro.


## 🚦 Policy-Based Forwarding Rules

### Endpoints
| Operação | Método + Caminho | Uso típico |
| --- | --- | --- |
| Listar | `GET /Policies/PolicyBasedForwardingRules` | Auditar regras vigentes por VSYS/pre/post |
| Criar | `POST /Policies/PolicyBasedForwardingRules` | Publicar nova regra de forwarding |
| Editar | `PUT /Policies/PolicyBasedForwardingRules` | Ajustar match/action existente |
| Excluir | `DELETE /Policies/PolicyBasedForwardingRules` | Remover regra sem uso |
| Renomear | `POST /Policies/PolicyBasedForwardingRules:rename` | Adequar convenção de nomes |
| Reordenar | `POST /Policies/PolicyBasedForwardingRules:move` | Controlar ordem de avaliação |

> Schema completo em [pa.espec.json#L17315-L18180](pa.espec.json#L17315-L18180); os endpoints acima exigem `location`/`vsys` consistentes com o restante das Policies.

### Campos essenciais
- `entry.@name`: até 63 caracteres; recomendo o padrão `PBF_<contexto>_<ordem>`.
- `from`: escolha entre blocos `zone.member[]` ou `interface.member[]`; combine apenas um tipo por regra.
- `source`/`destination`/`service`: aceitam objetos e grupos (`any` por default); `application.member[]` amplia o match.
- `source-user`: suporta usuários locais, grupos e DAGs; utilize somente quando User-ID estável.
- `schedule`: referencia `Objects/Schedules` para ativação condicional.
- `tag.member[]` + `group-tag`: usados para filtros visuais e automação.
- `negate-source`/`negate-destination`: transforma a lógica em "exceto".
- `action`: define o comportamento (ver tabela abaixo).
- `enforce-symmetric-return`: garante retorno pelo mesmo next-hop via `nexthop-address-list.entry[]`.
- `active-active-device-binding`: em HA A/A force `0`, `1` ou `both` conforme o node responsável.
- `description` e `disabled=yes` ajudam no lifecycle sem remover a regra.

### Ações suportadas
| Tipo | Campos obrigatórios | Quando usar |
| --- | --- | --- |
| `forward` | `forward.egress-interface`, opcional `nexthop.ip-address`/`fqdn`, `monitor.profile` | Desviar tráfego para interface/next-hop específico (ex.: link MPLS). |
| `forward-to-vsys` | `forward-to-vsys` | Encaminhar sessão para outro VSYS/shared gateway. |
| `discard` | nenhum adicional | Drop controlado mantendo contadores de regra. |
| `no-pbf` | nenhum adicional | Faz match para fins de log/tag mas mantém roteamento padrão. |

### Monitoramento da ação `forward`
- `monitor.profile`: referência a `Network/TunnelMonitorNetworkProfiles` ou perfil custom; necessário para habilitar monitoramento.
- `monitor.disable-if-unreachable`: define se a regra é auto-desabilitada quando o monitor falha (`yes` favorece failover para rotas padrão).
- `monitor.ip-address`: IP real a ser pingado; pode ser diferente do `nexthop`.

### Payloads de referência

#### 1. Forward com failover ativo e monitor
```json
{
  "entry": {
    "@name": "PBF_TDEVOPS_INTERNET",
    "from": { "zone": { "member": ["Trust"] } },
    "source": { "member": ["10.50.10.0/24"] },
    "destination": { "member": ["any"] },
    "service": { "member": ["application-default"] },
    "application": { "member": ["web-browsing", "ssl"] },
    "tag": { "member": ["TDEVOPS", "internet-breakout"] },
    "action": {
      "forward": {
        "egress-interface": "ethernet1/5",
        "nexthop": { "ip-address": "200.200.200.2" },
        "monitor": {
          "profile": "wait-recover",
          "disable-if-unreachable": "yes",
          "ip-address": "8.8.8.8"
        }
      }
    },
    "enforce-symmetric-return": {
      "enabled": "yes",
      "nexthop-address-list": {
        "entry": [ { "@name": "200.200.200.2" } ]
      }
    },
    "description": "Força breakout SaaS via link secundário"
  }
}
```

#### 2. Encaminhar para VSYS de serviços e fallback no routing padrão
```json
{
  "entry": {
    "@name": "PBF_TFCVY2_SERVICES",
    "from": { "zone": { "member": ["Prod"] } },
    "source": { "member": ["GRP_APP_SERVERS"] },
    "destination": { "member": ["any"] },
    "service": { "member": ["SGRP_APP"] },
    "action": {
      "forward-to-vsys": "vsys-services"
    },
    "schedule": "BUSINESS_HOURS",
    "negate-destination": "no",
    "description": "Encaminha tráfego dos apps para inspeção L7",
    "active-active-device-binding": "both"
  }
}
```

#### 3. Regra de descarte para destinar RFC1918 não autorizada
```json
{
  "entry": {
    "@name": "PBF_DROP_PRIVATE_EGRESS",
    "from": { "zone": { "member": ["DMZ"] } },
    "source": { "member": ["any"] },
    "destination": { "member": ["RFC1918_SUPerset"] },
    "service": { "member": ["any"] },
    "action": { "discard": {} },
    "description": "Bloqueia endereço privado tentando sair pela DMZ",
    "disabled": "no"
  }
}
```

### Operações comuns
- **Mover prioridade**: `POST .../PolicyBasedForwardingRules:move?name=PBF_TDEVOPS_INTERNET&where=before&dst=PBF_DROP_PRIVATE_EGRESS`
- **Renomear**: `POST .../PolicyBasedForwardingRules:rename?name=PBF_TDEVOPS_INTERNET&newname=PBF_TDEVOPS_INTERNET_v2`
- **Desabilitar sem remover**: `PUT` setando `disabled=yes` para preservar histórico.
- **Clonar**: `GET` + `POST` com novo nome e ajustes de fontes/destinos.

### Boas práticas
- Ordene PBF antes das Security Rules dependerem do efeito (match top-down, sem rulebase separada).
- Use `monitor.disable-if-unreachable=yes` para que o tráfego volte ao roteamento dinâmico quando o next-hop falhar.
- Evite `forward-to-vsys` sem rotas de retorno configuradas; combine com `enforce-symmetric-return` quando aplicável.
- Documente tags e group-tags nas planilhas para rastrear automações (Terraform/Ansible) que filtram por esses valores.
- Gere relatórios com `GET` + `jq` periodicamente para garantir que regras com `discard` ou `no-pbf` estejam justificadas.


## 🛡️ Network Zones

### Endpoints
| Operação | Método + Caminho | Uso típico |
| --- | --- | --- |
| Listar | `GET /Network/Zones` | Inventariar zonas por VSYS e confirmar interfaces members |
| Criar | `POST /Network/Zones` | Publicar zonas Layer2/3, Tunnel, Tap ou Virtual Wire |
| Editar | `PUT /Network/Zones` | Ajustar membros, perfis ou flags (User-ID, Device-ID) |
| Excluir | `DELETE /Network/Zones` | Remover zonas obsoletas (sem interfaces associadas) |

> Lembre de informar `location=vsys` e `vsys=<nome>`; zonas não possuem endpoints de `:rename` ou `:move`, portanto renomeios exigem recriação.

### Campos essenciais
- `entry.@name`: até 31 caracteres; padronize `ZN_<domínio>_<contexto>` para casar com regras de segurança/NAT.
- `network`: define o tipo da zona (`layer3`, `layer2`, `tap`, `virtual-wire`, `tunnel`, `external`). Cada bloco aceita `member[]` com as interfaces permitidas.
- `zone-protection-profile`: referencia `Network/ZoneProtectionNetworkProfiles` para inspeções de flood/scan por zona.
- `log-setting`: aponta para um Log Forwarding Profile para eventos Intrazone/Interzone e logs de sistemas associados à zona.
- `enable-user-identification`: ativa User-ID; combine com `user-acl.include-list` e `exclude-list` quando for necessário limitar quais sub-redes participam.
- `device-identification`: habilita Device-ID para inventário e políticas baseadas em dispositivo.
- `include-list` / `exclude-list`: definem ranges/fqdn que podem (ou não) produzir mapeamentos User-ID/Device-ID.
- `description`, `tag.member[]` e `group-tag`: facilitam automações e revisões de compliance.
- `zone-profile-setting`: opcionalmente referencia perfis de inspeção avançada (Packet Buffer Protection, Content-ID) quando habilitados.

### Tipos de zona suportados
| Tipo | Bloco | Interfaces aceitas | Observações |
| --- | --- | --- | --- |
| Layer3 | `network.layer3.member[]` | Ethernet L3, subinterfaces, loopbacks | Mais comum; suporta User-ID, Device-ID, Zone Protection e logging completo. |
| Layer2 | `network.layer2.member[]` | Interfaces L2/AE subinterfaces | Usado em domínios switching; não participa de roteamento direto. |
| Virtual Wire | `network.virtual-wire.member[]` | Interfaces em pares vwire | Ideal para inspeção transparente; não suporta User-ID. |
| Tap | `network.tap.member[]` | Interfaces TAP | Apenas monitoramento; tráfego não é transmitido. |
| Tunnel | `network.tunnel.member[]` | `tunnel.X` | Necessário para VPNs e SD-WAN; combine com zonas específicas para selecionar regras. |
| External | `network.external.member[]` | `l3` ou `loopback` expostos | Usado por GlobalProtect e integrações GP Portal/Gateway. |

### Recursos de segurança/adicionais
- `intrazone-default` / `interzone-default`: herdam logging e perfis com base nas zonas Trust/Untrust; mantenha coerência entre zona e regras padrões.
- `zone-protection-profile` + `log-setting`: atacam vetores volumétricos e exportam logs; sempre associe quando a zona recebe tráfego público.
- `packet-buffer-protection`: habilitado via `zone-profile-setting` para evitar exploitation por buffer exhaustion.
- `user-acl.include-list`/`exclude-list`: controlam quais sub-redes podem publicar mapeamentos User-ID; útil quando a zona contém redes de terceiros.
- `device-identification`: exige conteúdo dos perfis Device-ID; use somente quando a licença estiver ativa para evitar alertas ruidosos.

### Payloads de referência

#### 1. Zona Layer3 com User-ID e Zone Protection
```json
{
  "entry": {
    "@name": "ZN_TDEVOPS_TRUST",
    "network": {
      "layer3": { "member": ["ethernet1/3", "ethernet1/4.10", "loopback.3"] }
    },
    "zone-protection-profile": "ZP_TRUST_IN",
    "log-setting": "LOG_TDEVOPS",
    "enable-user-identification": "yes",
    "device-identification": "yes",
    "user-acl": {
      "include-list": { "member": ["10.50.10.0/24", "10.60.20.0/24"] },
      "exclude-list": { "member": ["10.50.10.200"] }
    },
    "tag": { "member": ["TDEVOPS", "trust"] },
    "description": "Zona interna das aplicações TOTVS"
  }
}
```

#### 2. Zona Tunnel dedicada às VPNs B2B
```json
{
  "entry": {
    "@name": "ZN_VPN_B2B",
    "network": {
      "tunnel": { "member": ["tunnel.100", "tunnel.101"] }
    },
    "zone-protection-profile": "ZP_VPN",
    "log-setting": "LOG_VPN",
    "enable-user-identification": "no",
    "device-identification": "no",
    "tag": { "member": ["vpn", "b2b"] },
    "description": "Tráfego proveniente dos IPsec tunnels parceiros"
  }
}
```

### Operações comuns
- **Criar zona Layer3**: `POST /Network/Zones?location=vsys&vsys=vsys1` com payload similar ao exemplo Trust.
- **Atualizar membros**: `PUT` substituindo todo o bloco `network.<tipo>.member[]` (o API não oferece adição incremental).
- **Associar Zone Protection**: `PUT` informando `zone-protection-profile` já existente em `Network/ZoneProtectionNetworkProfiles`.
- **Auditar uso**: `GET /Network/Zones?location=vsys&vsys=vsys1 | jq '.response.result.entry[] | {name: .["@name"], type: (.network | keys[])}'`.

### Boas práticas
- Nomeie zonas alinhadas aos domínios de segurança (Trust, DMZ, VPN, Mgmt) e sincronize com planilhas de migração.
- Nunca deixe interfaces órfãs: mova-as para `null zone` (`none`) antes de excluir a zona para evitar falhas no `DELETE`.
- Sempre aplique um Zone Protection Profile em zonas expostas à Internet; combine com log forwarding para facilitar incident response.
- Ative User-ID apenas quando realmente necessário e restrinja sub-redes com `include-list`/`exclude-list` para mitigar spoofing.
- Revise zonas Tunnel e External após alterações em VPNs/GP para garantir que novas interfaces lógicas estejam cobertas pelas políticas corretas.

## 🧩 Device/Virtual Systems

### Endpoints
| Operação | Método + Caminho | Uso típico |
| --- | --- | --- |
| Listar | `GET /Device/VirtualSystems?location=device` | Inventariar VSYS, interfaces e objetos importados |
| Criar | `POST /Device/VirtualSystems?location=device` | Publicar um novo VSYS para um tenant ou domínio lógico |
| Editar | `PUT /Device/VirtualSystems?location=device` | Ajustar descrições, imports e perfis de segurança |
| Excluir | `DELETE /Device/VirtualSystems?location=device` | Remover VSYS que não possuem interfaces/objetos associados |

> Para editar um VSYS existente informe `@name` no payload; `vsys1` é criado automaticamente e não pode ser excluído. Multi-VSYS exige licença ativa.

### Campos essenciais
- `entry.@name`: identificador técnico (`vsys1`, `vsys2`, `vsys-tenantA`). Evite espaços e mantenha até 31 caracteres.
- `display-name`: rótulo amigável exibido no GUI; pode repetir entre appliances, mas mantenha padrão `VSYS_<cliente>`.
- `description`: contexto de uso, responsável e janela de manutenção.
- `import.interface.member[]`: interfaces L2/L3, loopbacks, túneis ou subinterfaces que pertencem ao VSYS.
- `import.virtual-router.member[]`: associa um ou mais Virtual Routers (documentados em `/Networks/VirtualRouter`).
- `import.zone.member[]`: lista as zonas que ficam visíveis dentro do VSYS (só funciona para zonas criadas com `location=vsys`).
- `import.vlan.member[]`, `import.virtual-wire.member[]`, `import.qos.member[]`, `import.sdwan.member[]`: traz objetos avançados compartilhados pelo dispositivo.
- `import.profile-setting.profiles.<tipo>.member[]`: vincula perfis de segurança compartilhados (AV, AntiSpyware, Vulnerability, URL Filtering, File Blocking, WildFire, DoS, etc.).
- `application-override` / `universal-category-usage`: flags específicos para compatibilidade com App-ID legado; use somente se indicado no inventário.

### Importações comuns
| Recurso | Nó de importação | Observações |
| --- | --- | --- |
| Interfaces físicas/lógicas | `import.interface.member[]` | Cada interface só pode pertencer a um VSYS por vez. Remova do VSYS anterior antes de migrar. |
| Virtual Routers | `import.virtual-router.member[]` | Necessário para que o VSYS enxergue rotas e BGP/OSPF associados. |
| Zonas | `import.zone.member[]` | Torna disponível o conjunto de zonas criado para o tenant, permitindo referenciá-las em políticas. |
| VLANs | `import.vlan.member[]` | Útil para ambientes L2 com SVI internos. |
| Virtual Wires | `import.virtual-wire.member[]` | Necessário para firewalls transparentes multi-tenant. |
| QoS Profiles | `import.qos.member[]` | Habilita políticas QoS por VSYS. |
| SD-WAN Profiles | `import.sdwan.member[]` | Disponibiliza roteamento SD-WAN ou templates de virtual path específicos. |
| Perfis de segurança | `import.profile-setting.profiles.<tipo>.member[]` | Permite reutilizar perfis já padronizados no Device Template. |

### Payloads de referência

#### 1. Criação de VSYS corporativo completo
```json
{
  "entry": {
    "@name": "vsys2",
    "display-name": "VSYS_TDEVOPS",
    "description": "Tenant TOTVS DevOps",
    "import": {
      "interface": { "member": ["ethernet1/5", "ethernet1/6.10", "tunnel.120"] },
      "virtual-router": { "member": ["VR_TDEVOPS"] },
      "zone": { "member": ["ZN_TDEVOPS_TRUST", "ZN_TDEVOPS_DMZ"] },
      "qos": { "member": ["QOS_TDEVOPS"] },
      "sdwan": { "member": ["SDWAN_TDEVOPS"] },
      "profile-setting": {
        "profiles": {
          "virus": { "member": ["AV_DEFAULT"] },
          "spyware": { "member": ["AS_DEFAULT"] },
          "vulnerability": { "member": ["VULN_CORP"] },
          "url-filtering": { "member": ["URL_CORP"] },
          "wildfire-analysis": { "member": ["WF_GLOBAL"] }
        }
      }
    }
  }
}
```

#### 2. Atualização para importar nova interface e zona
```json
{
  "entry": {
    "@name": "vsys2",
    "import": {
      "interface": { "member": ["ethernet1/5", "ethernet1/6.10", "ethernet1/7.200", "tunnel.120"] },
      "zone": { "member": ["ZN_TDEVOPS_TRUST", "ZN_TDEVOPS_DMZ", "ZN_TDEVOPS_VPN"] }
    }
  }
}
```

### Operações comuns
- **Inventário rápido**: `GET /Device/VirtualSystems?location=device | jq '.response.result.entry[] | {name: ."@name", interfaces: (.import.interface.member // [])}'`.
- **Criar VSYS**: `POST` com payload completo garantindo que VR, zonas e perfis existam previamente.
- **Atualizar imports**: `PUT` substitui o array informado; inclua todos os `member[]` desejados para evitar remoção acidental.
- **Remover VSYS**: `DELETE` aceito apenas quando não há interfaces ou objetos vinculados; desassocie-os primeiro.

### Boas práticas
- Defina convenções `vsys<id>` + `display-name` legível para facilitar mapeamentos com CMDB/tenants.
- Automatize validações para garantir que toda interface, VR e zona referenciada exista antes do `POST/PUT`.
- Não reutilize `vsys1` para novos tenants; mantenha-o para serviços compartilhados ou legado, simplificando troubleshooting.
- Documente quem é o owner de cada VSYS e quais objetos são compartilhados (`import.*`) para evitar exclusões involuntárias.
- Em ambientes Panorama, mantenha a mesma estrutura de VSYS entre templates para reduzir drift na hora do push.


