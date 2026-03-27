# FortiGate VoIP MOS Monitor by SNMP - LLD

Template Zabbix 7.0 para monitoramento de qualidade de link e cálculo de MOS (Mean Opinion Score) em FortiGate via SNMP, com auto-discovery (LLD).

**Versão:** 1.7.0
**Autor:** 9LEVEL / Liberato
**Grupo:** Templates/Network devices

---

## O que faz

Descobre automaticamente os **Link Monitors** e **SD-WAN Health-Checks** configurados no FortiGate e monitora a qualidade de cada link em tempo real. A partir das métricas brutas coletadas via SNMP (latência, jitter, packet loss, bandwidth), calcula o **R-Factor** e o **MOS Score** usando um modelo E simplificado baseado na recomendação ITU-T.

---

## Discovery Rules

O template possui duas regras de descoberta automática (intervalo de 1h, lifetime de 7d):

**Link Monitor Discovery** — Descobre entradas na tabela `fgLinkMonitor` (OID `1.3.6.1.4.1.12356.101.4.8.2.1`) identificando nome e estado de cada Link Monitor configurado.

**SD-WAN Health-Check Discovery** — Descobre entradas na tabela `fgVWLHealthCheckLink` (OID `1.3.6.1.4.1.12356.101.4.9.2.1`) identificando nome do health-check, estado e **nome da interface** associada, permitindo granularidade por link físico/lógico.

---

## Métricas Coletadas

Para cada link descoberto (Link Monitor e SD-WAN), são coletados/calculados 8 itens com intervalo de 60s:

| Métrica | Tipo | Unidade | Descrição |
|---|---|---|---|
| Latência | SNMP | ms | Round-trip delay |
| Jitter | SNMP | ms | Variação de latência |
| Packet Loss | SNMP | % | Percentual de perda de pacotes |
| Bandwidth In | SNMP | Mbps | Largura de banda de entrada |
| Bandwidth Out | SNMP | Mbps | Largura de banda de saída |
| Bandwidth Bidi | SNMP | Mbps | Largura de banda bidirecional |
| R-Factor | Calculado | — | Modelo E simplificado |
| MOS Score | Calculado | — | Conversão R→MOS (escala 1.0–4.5) |

**Retenção:** 90 dias de histórico, 365 dias de trends.

---

## Cálculo do MOS

O MOS é derivado em duas etapas:

**1. R-Factor (modelo E simplificado):**

```
R = 93.2 - (latency / 40) - (jitter / 10) - (packetloss * 2.5)
```

Clamped entre 0 e 93.2.

**2. MOS (conversão ITU-T):**

```
MOS = 1 + (0.035 * R) + (R * (R - 60) * (100 - R) * 0.000007)
```

Clamped entre 1.0 e 4.5.

---

## Triggers

Todos os thresholds são configuráveis via macros. Os triggers possuem dependências encadeadas para evitar flood de alertas.

| Trigger | Severidade | Condição padrão |
|---|---|---|
| MOS degradado | WARNING | MOS < 3.6 |
| MOS CRÍTICO | HIGH | MOS < 3.1 |
| MOS PERSISTENTE 15min | HIGH (`open-ticket`) | MOS < 3.6 por 15 minutos contínuos |
| DEAD - 100% packet loss | DISASTER | Packet loss = 100% |
| Latência elevada | WARNING | > 150 ms |
| Latência CRÍTICA | HIGH | > 300 ms |
| Jitter elevado | WARNING | > 30 ms |
| Jitter CRÍTICO | HIGH | > 50 ms |
| Packet Loss elevado | WARNING | > 1% |
| Packet Loss CRÍTICO | HIGH | > 3% |
| Bandwidth baixa | WARNING | < 10 Mbps (bidi > 0) |

**Hierarquia de dependências:** WARNING → CRITICAL → DEAD, evitando que triggers de menor severidade disparem quando um de maior já está ativo.

---

## Macros Configuráveis

| Macro | Padrão | Descrição |
|---|---|---|
| `{$SNMP_COMMUNITY}` | `public` | Community SNMP do FortiGate |
| `{$MOS_WARNING}` | `3.6` | Threshold MOS para warning |
| `{$MOS_CRITICAL}` | `3.1` | Threshold MOS para critical |
| `{$LATENCY_WARNING}` | `150` | Latência warning (ms) |
| `{$LATENCY_CRITICAL}` | `300` | Latência critical (ms) |
| `{$JITTER_WARNING}` | `30` | Jitter warning (ms) |
| `{$JITTER_CRITICAL}` | `50` | Jitter critical (ms) |
| `{$PACKETLOSS_WARNING}` | `1` | Packet loss warning (%) |
| `{$PACKETLOSS_CRITICAL}` | `3` | Packet loss critical (%) |
| `{$BANDWIDTH_MIN_MBPS}` | `10` | Bandwidth mínima (Mbps) |

---

## Value Map

O template inclui o value map **FortiGate Link Quality**:

| Valor | Classificação |
|---|---|
| 5 | Excelente |
| 4 | Bom |
| 3 | Aceitável |
| 2 | Ruim |
| 1 | Inaceitável |

---

## Pré-requisitos

- Zabbix Server/Proxy 7.0+
- SNMP v2c (ou v3) habilitado no FortiGate
- Link Monitors e/ou SD-WAN Health-Checks configurados no FortiGate
- Acesso SNMP do Zabbix ao FortiGate (firewall/ACL liberados)

## Instalação

1. Importe o arquivo `zbx_template_fortigate_mos_v1_7.yaml` no Zabbix (Configuration → Templates → Import)
2. Vincule o template ao host FortiGate
3. Ajuste a macro `{$SNMP_COMMUNITY}` com a community correta
4. Ajuste os thresholds conforme necessidade do ambiente
5. Aguarde o ciclo de discovery (1h) ou force manualmente

---

## Tags

Os itens e triggers utilizam tags para facilitar filtragem e automações:

- `component: voip` — métricas de qualidade de voz
- `component: network` — métricas de bandwidth
- `component: quality` — itens calculados (R-Factor/MOS)
- `scope: voip` — triggers de MOS
- `scope: performance` — triggers de latência, jitter e bandwidth
- `scope: availability` — triggers de packet loss e link dead
- `action: open-ticket` — trigger de MOS persistente (integração com ticket system)
