# zabbix-templates

Coleção de templates Zabbix desenvolvidos pela **9LEVEL** para monitoramento de infraestrutura de rede, telecom e qualidade VoIP.

Foco em ambientes com call center, SD-WAN e links críticos para voz — onde visibilidade sobre latência, jitter, packet loss e MOS faz diferença entre resolver um problema em minutos ou descobrir depois que o estrago já foi feito.

---

## Templates disponíveis

| Template | Arquivo | Versão | Descrição |
|---|---|---|---|
| FortiGate VoIP MOS Monitor | `zbx_template_fortigate_mos_v1_7.yaml` | 1.7.0 | Monitoramento de qualidade de link e cálculo de MOS via SNMP com auto-discovery de Link Monitors e SD-WAN Health-Checks |

> Novos templates serão adicionados conforme demanda e maturidade.

---

## Características gerais

- **Auto-discovery (LLD)** — os templates descobrem automaticamente os recursos monitoráveis, sem necessidade de cadastro manual de itens
- **Thresholds via macros** — todos os limites de alerta são configuráveis por host, sem precisar editar o template
- **Triggers com dependências** — hierarquia WARNING → HIGH → DISASTER para evitar flood de alertas
- **Tags padronizadas** — `component`, `scope` e `action` para facilitar filtragem, dashboards e integração com sistemas de ticket
- **Compatibilidade** — Zabbix 7.0+

---

## Estrutura do repositório

```
zabbix-templates/
├── README.md
├── fortigate/
│   ├── zbx_template_fortigate_mos_v1_7.yaml
│   └── zbx_fortigate_mos.md
└── ...
```

Cada template possui seu próprio `.md` com documentação detalhada (métricas, triggers, macros, fórmulas e instruções de instalação).

---

## Instalação rápida

1. Clone o repositório ou baixe o arquivo `.yaml` desejado
2. No Zabbix, vá em **Configuration → Templates → Import**
3. Selecione o arquivo e importe
4. Vincule o template ao host
5. Ajuste as macros (community SNMP, thresholds) conforme seu ambiente
6. Aguarde o ciclo de discovery ou force manualmente

---

## Requisitos

- Zabbix Server ou Proxy 7.0+
- SNMP habilitado nos dispositivos monitorados
- Acesso de rede (firewall/ACL) do Zabbix ao dispositivo

Requisitos específicos de cada template estão documentados no respectivo `.md`.

---

## Contribuindo

Encontrou um bug, quer sugerir um threshold melhor ou tem um template novo? Abre uma issue ou manda um PR.

---

## Licença

MIT

---

Mantido por [9LEVEL](https://9level.com.br)
