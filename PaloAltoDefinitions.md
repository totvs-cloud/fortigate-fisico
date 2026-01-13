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
- Campos opcionais relevantes: `profile-setting` (ex.: antivírus), `log-setting`, `schedule` (usa objetos de `Schedules`), `tag.member`.

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
    "application": { "member": ["any"] },
    "service": { "member": ["TCP-2203", "TCP-2204", "TCP-2202"] },
    "action": "allow",
    "log-setting": "FWD_ELK_syslog_vsys2",
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
    "tag": { "member": ["TFEOT4_CKOYPH_ate"] },
    "profile-setting": {
      "profiles": {
        "virus": { "member": ["default"] },
        "vulnerability": { "member": ["ips_base"] }
      }
    }
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
    "tag": { "member": ["TFCVY2_C6HOGA_ate"] },
    "schedule": { "member": ["SCH_TFEOT4_MAINT"] }
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

## 🔀 De-Para Palo Alto ➜ Fortinet

| Tipo Palo Alto | Conversão Fortinet | Observações |
| --- | --- | --- |
| `ip-netmask` | `set subnet <ip> <mask>` | /32 = `255.255.255.255`. |
| `ip-range` | `set type iprange` + `set start-ip`/`set end-ip` | Validar sobreposição com objetos existentes. |
| `ip-wildcard` | Multiplicar em várias sub-redes/hosts | Automatizar via script para evitar erros. |
| `fqdn` | `set type fqdn` + `set fqdn` | Confirmar modo DNS (cache) compatível. |
| AddressGroup estático | `set member` com objetos equivalentes | Suporta membros que são grupos. |
| AddressGroup dinâmico | Transformar em lista estática | Necessário extrair tags e gerar script Fortinet. |
| Schedule | `set schedule <nome>` ou janelas simples | Fortinet não replica non-recurring complexos; crie múltiplas janelas. |
| Security Rule | `config firewall policy` | Observe diferenças de ordem e campos (profiles/log). |

---

## ✅ Checklist de Migração
1. **Inventariar**: `GET` em Addresses, AddressGroups, Services, ServiceGroups, Schedules e SecurityRules; exportar para CSV.
2. **Classificar**: marcar tipo (host, range, fqdn, dinâmico, tcp, udp, regra) e prioridade.
3. **Normalizar nomes**: usar `:rename` para padronizar antes de gerar scripts.
4. **Gerar scripts Fortinet**: converter JSON ➜ CLI (`config firewall address/addrgrp`) e mapear serviços.
5. **Validar**: reexecutar `GET`, comparar contagem e nomes com inventário Fortinet.

> 🔁 Recomendação: após cada lote migrado, execute políticas de auditoria (`diagnose/firewall/policy list`) para garantir que todos os objetos Fortinet estão associados às políticas esperadas.

---

## 📌 Próximos Passos
- Replicar este padrão de documentação para Services/ServiceGroups e demais objetos necessários.
- Automatizar o de-para com scripts que leem os payloads acima e produzem a CLI Fortinet correspondente.
