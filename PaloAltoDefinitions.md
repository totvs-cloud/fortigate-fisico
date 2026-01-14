# 🧭 Gestão de Objetos: Addresses, AddressGroups, Services & ServiceGroups

Documento baseado na especificação oficial [pa.espec.json](pa.espec.json). Objetivo: orientar o time de migração Palo Alto ➜ Fortinet no de-para de objetos de endereço e serviço, com exemplos simples e prontos para copiar.

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
| Operação | Método + Caminho | Uso típico | Equivalente Fortinet |
| --- | --- | --- | --- |
| Listar | `GET /Objects/Addresses` | Inventariar objetos existentes | `show firewall address` |
| Criar | `POST /Objects/Addresses` | Inserir host/FQDN/faixa | `config firewall address` |
| Editar | `PUT /Objects/Addresses` | Ajustar IP, descrição ou tags | `set subnet`, `set type` |
| Excluir | `DELETE /Objects/Addresses` | Remover objetos órfãos | `delete <address>` |
| Renomear | `POST /Objects/Addresses:rename` | Padronizar nomes antes da migração | `rename <address>` |

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
| Operação | Método + Caminho | Uso típico | Equivalente Fortinet |
| --- | --- | --- | --- |
| Listar | `GET /Objects/AddressGroups` | Mapear grupos usados em políticas | `show firewall addrgrp` |
| Criar | `POST /Objects/AddressGroups` | Agrupar objetos para regras | `config firewall addrgrp` |
| Editar | `PUT /Objects/AddressGroups` | Atualizar membros ou filtros | `set member` / `unset` |
| Excluir | `DELETE /Objects/AddressGroups` | Limpar grupos não usados | `delete <addrgrp>` |
| Renomear | `POST /Objects/AddressGroups:rename` | Manter convenção de nomes | `rename <addrgrp>` |

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
- Lembrete: filtros dinâmicos consideram **tags dos objetos**. Se não existirem no Fortinet, planeje criação manual.

---

## 🎛 Services

### Endpoints
| Operação | Método + Caminho | Uso típico | Equivalente Fortinet |
| --- | --- | --- | --- |
| Listar | `GET /Objects/Services` | Inventariar portas customizadas | - |
| Criar | `POST /Objects/Services` | Registrar TCP/UDP específicos | - |
| Editar | `PUT /Objects/Services` | Ajustar portas ou timeouts | - |
| Excluir | `DELETE /Objects/Services` | Remover serviços obsoletos | - |
| Renomear | `POST /Objects/Services:rename` | Padronizar nomenclatura | - |

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
- Quando precisar diferenciar origem/destino, documente via tags ou descrição (não há campo dedicado no Fortinet).
- Defina timeouts customizados apenas quando houver requerimento funcional claro.
- Para serviços herdados (`location=predefined`), apenas leitura está disponível; crie novos em `shared` ou `vsys` para personalizações.

---

## 🧬 ServiceGroups

### Endpoints
| Operação | Método + Caminho | Uso típico | Equivalente Fortinet |
| --- | --- | --- | --- |
| Listar | `GET /Objects/ServiceGroups` | Entender blocos de portas reutilizados | - |
| Criar | `POST /Objects/ServiceGroups` | Agrupar serviços para regras | - |
| Editar | `PUT /Objects/ServiceGroups` | Atualizar membros | - |
| Excluir | `DELETE /Objects/ServiceGroups` | Remover grupos não usados | - |
| Renomear | `POST /Objects/ServiceGroups:rename` | Manter organização | - |

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
| Operação | Método + Caminho | Uso típico | Equivalente Fortinet |
| --- | --- | --- | --- |
| Listar | `GET /Policies/SecurityRules` | Auditar políticas vigentes | - |
| Criar | `POST /Policies/SecurityRules` | Publicar regras L3/L4 | - |
| Editar | `PUT /Policies/SecurityRules` | Ajustar campos existentes | - |
| Excluir | `DELETE /Policies/SecurityRules` | Remover regras obsoletas | - |
| Renomear | `POST /Policies/SecurityRules:rename` | Padronizar nomes | - |
| Mover | `POST /Policies/SecurityRules:move` | Reordenar prioridades | - |

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
| Operação | Método + Caminho | Uso típico | Equivalente Fortinet |
| --- | --- | --- | --- |
| Listar | `GET /Policies/NatRules` | Auditar traduções configuradas | - |
| Criar | `POST /Policies/NatRules` | Publicar SNAT ou DNAT | - |
| Editar | `PUT /Policies/NatRules` | Ajustar blocos de tradução | - |
| Excluir | `DELETE /Policies/NatRules` | Limpar regras obsoletas | - |
| Renomear | `POST /Policies/NatRules:rename` | Organizar convenções de nomes | - |
| Mover | `POST /Policies/NatRules:move` | Reordenar precedência | - |

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
- Utilize os fluxos descritos em [nats.md](nats.md) para manter a sequência service ➜ host ➜ rule ➜ nat ➜ Fortinet consistente.


## 🌉 IKE Gateways

### Endpoints
| Operação | Método + Caminho | Uso típico | Equivalente Fortinet |
| --- | --- | --- | --- |
| Listar | `GET /Network/IKEGateways` | Validar status e parâmetros de VPNs | `show vpn ipsec phase1-interface` |
| Criar | `POST /Network/IKEGateways` | Provisionar gateways (psk/cert) | `config vpn ipsec phase1-interface` |
| Editar | `PUT /Network/IKEGateways` | Ajustar peer, crypto ou DPD | `set proposal`, `set peerip` |
| Excluir | `DELETE /Network/IKEGateways` | Remover gateways obsoletos | `delete <phase1>` |
| Renomear | `POST /Network/IKEGateways:rename` | Alinhar convenções por TAG | `rename <phase1>` |

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
- Documente `peer-id`/`local-id` com o mesmo valor usado na contraparte Fortinet (campo `set peerid` / `set localid`).
- Ative `dpd` e `nat-traversal` conforme as características do enlace para evitar quedas falsas.
- Em certificação, alinhe `certificate-profile` aos parâmetros de CRL/OCSP exigidos pelo SOC.
- Mantenha senhas PSK fora do Git e injete via pipeline/secret manager, apesar do payload de exemplo mostrar valores reais.


## 🔒 IKE Crypto Profiles

### Endpoints
| Operação | Método + Caminho | Uso típico | Equivalente Fortinet |
| --- | --- | --- | --- |
| Listar | `GET /Network/IkeCryptoProfiles` | Auditar propostas fase 1 reutilizadas | `show vpn ipsec phase1-interface` |
| Criar | `POST /Network/IkeCryptoProfiles` | Padronizar cifragem/integração | `set proposal` (phase1) |
| Editar | `PUT /Network/IkeCryptoProfiles` | Ajustar algoritmos e lifetime | `set dhgrp`, `set keylife` |
| Excluir | `DELETE /Network/IkeCryptoProfiles` | Limpar perfis sem uso | `delete <phase1-profile>` |

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
| Operação | Método + Caminho | Uso típico | Equivalente Fortinet |
| --- | --- | --- | --- |
| Listar | `GET /Network/IpsecTunnels` | Conferir túneis (phase2) | `show vpn ipsec phase2-interface` |
| Criar | `POST /Network/IpsecTunnels` | Publicar auto-key ou manual | `config vpn ipsec phase2-interface` |
| Editar | `PUT /Network/IpsecTunnels` | Ajustar proxy-id, monitor, interface | `set dst-subnet`, `set keepalive` |
| Excluir | `DELETE /Network/IpsecTunnels` | Limpar túneis desativados | `delete <phase2>` |

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
- Nomeie `tunnel-interface` igual em ambos appliances para simplificar troubleshooting (ex.: `tunnel.100`).
- Monitore `tunnel-monitor.destination-ip` em um host realmente disponível; evite endereços virtuais que podem cair junto com o túnel.
- Em clusters, lembre-se de configurar `floating-ip` quando o enlace termina em interface HA ativa/ativa.




