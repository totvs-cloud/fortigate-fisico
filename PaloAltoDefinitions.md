# 🧭 Gestão de Objetos: Addresses, AddressGroups, Services & ServiceGroups

Documento baseado na especificação oficial [pa.espec.json](pa.espec.json). Objetivo: orientar o time de migração Palo Alto ➜ Fortinet no de-para de objetos de endereço e serviço, com exemplos simples e prontos para copiar.

---

## 🌐 Contexto Geral
- **Base URL:** `https://<firewall>/restapi/v10.2`
- **Autenticação:** header `X-PAN-KEY: <api-key>` obtido via endpoint de geração de chave.
- **Formatos:** trabalhar preferencialmente em JSON (`input-format=json`, `output-format=json`).
- **Âmbito:** todos os endpoints exigem `location` (`shared` ou `vsys`) e, quando `location=vsys`, informar `vsys=<nome>`.

---

## 📮 Endpoints Principais

| Objeto | Operação | Método + Caminho | Uso típico | Equivalente Fortinet |
| --- | --- | --- | --- | --- |
| Address | Listar | `GET /Objects/Addresses` | Inventariar objetos existentes | `show firewall address`
| Address | Criar | `POST /Objects/Addresses` | Inserir host/FQDN/faixa | `config firewall address`
| Address | Editar | `PUT /Objects/Addresses` | Ajustar IP, descrição ou tags | `set subnet`, `set type`
| Address | Excluir | `DELETE /Objects/Addresses` | Remover objetos órfãos | `delete <address>`
| Address | Renomear | `POST /Objects/Addresses:rename` | Padronizar nomes antes da migração | `rename <address>`
| AddressGroup | Listar | `GET /Objects/AddressGroups` | Mapear grupos para regras | `show firewall addrgrp`
| AddressGroup | Criar | `POST /Objects/AddressGroups` | Agrupar objetos para políticas | `config firewall addrgrp`
| AddressGroup | Editar | `PUT /Objects/AddressGroups` | Atualizar membros/filtros | `set member` / `unset`
| AddressGroup | Excluir | `DELETE /Objects/AddressGroups` | Limpar grupos não usados | `delete <addrgrp>`
| AddressGroup | Renomear | `POST /Objects/AddressGroups:rename` | Manter convenção | `rename <addrgrp>`
| Service | Listar | `GET /Objects/Services` | Inventariar portas customizadas | - |
| Service | Criar | `POST /Objects/Services` | Registrar TCP/UDP específicos | - |
| Service | Editar | `PUT /Objects/Services` | Ajustar portas ou timeouts | - |
| Service | Excluir | `DELETE /Objects/Services` | Eliminar serviços obsoletos | - |
| Service | Renomear | `POST /Objects/Services:rename` | Padronizar nomenclatura | - |
| ServiceGroup | Listar | `GET /Objects/ServiceGroups` | Entender blocos de portas reutilizados | - |
| ServiceGroup | Criar | `POST /Objects/ServiceGroups` | Agrupar serviços para regras | - |
| ServiceGroup | Editar | `PUT /Objects/ServiceGroups` | Atualizar membros | - |
| ServiceGroup | Excluir | `DELETE /Objects/ServiceGroups` | Limpar grupos não usados | - |
| ServiceGroup | Renomear | `POST /Objects/ServiceGroups:rename` | Manter organização | - |

> 💡 Sempre inclua `name`, `location` e `vsys` nas chamadas que modificam recursos. Para leitura geral (`GET` sem filtro), `name` é opcional.

---

## 🧱 Addresses

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

---

## 🧩 AddressGroups

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

### Cuidados Essenciais
- Grupos podem referenciar outros grupos; mantenha a mesma hierarquia ao migrar.
- Descrições e tags ajudam na revisão posterior das regras.
- Lembrete: filtros dinâmicos consideram **tags dos objetos**. Se não existirem no Fortinet, planeje criação manual.

---

## 🎛 Services

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

### Boas Práticas
- Use ranges contínuos sempre que possível para reduzir objetos (ex.: `7000-7020`).
- Quando precisar diferenciar origem/destino, documente via tags ou descrição (não há campo dedicado no Fortinet).
- Defina timeouts customizados apenas quando houver requerimento funcional claro.
- Para serviços herdados (`location=predefined`), apenas leitura está disponível; crie novos em `shared` ou `vsys` para personalizações.

---

## 🧬 ServiceGroups

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

### Recomendações
- Revise dependências: um grupo pode referenciar outro; garanta que os objetos base já existam antes do POST.
- Tags ajudam a identificar conjuntos críticos quando for migrar políticas complexas.
- Planeje convenções de nomes (`svc-` para serviços, `grp-` para grupos) para facilitar mapeamentos automatizados.

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

---

## ✅ Checklist de Migração
1. **Inventariar**: `GET` em Addresses, AddressGroups, Services e ServiceGroups; exportar para CSV.
2. **Classificar**: marcar tipo (host, range, fqdn, dinâmico, tcp, udp) e prioridade.
3. **Normalizar nomes**: usar `:rename` para padronizar antes de gerar scripts.
4. **Gerar scripts Fortinet**: converter JSON ➜ CLI (`config firewall address/addrgrp`) e mapear serviços.
5. **Validar**: reexecutar `GET`, comparar contagem e nomes com inventário Fortinet.

> 🔁 Recomendação: após cada lote migrado, execute políticas de auditoria (`diagnose/firewall/policy list`) para garantir que todos os objetos Fortinet estão associados às políticas esperadas.

---

## 📌 Próximos Passos
- Replicar este padrão de documentação para Services/ServiceGroups e demais objetos necessários.
- Automatizar o de-para com scripts que leem os payloads acima e produzem a CLI Fortinet correspondente.
