# deteccao-e-resposta-wazuh
# Wazuh SOC Lab — Simulação de Ambiente SaaS

## Objetivo
Simular um ambiente SOC realista baseado em uma empresa SaaS (plataforma de incentivos/pagamentos), monitorando transações e detectando comportamentos suspeitos com Wazuh.

---

## Infraestrutura do Lab

| Máquina | Papel |
|---|---|
| Ubuntu Server | Wazuh Manager |
| Ubuntu | Agente + Servidor SaaS simulado |
| Windows | Agente |
| Kali Linux | Atacante |

Todas as máquinas em modo **Bridge** na mesma faixa `192.168.0.x`.

---

## Cenário B — Fraude SaaS

### Parte 1 — Preparação;

**Feito um Script de simulação SaaS** `/opt/saas_log_generator.py`

**Configuração dos agentes** `/var/ossec/etc/ossec.conf`:

**Regras customizadas** `/var/ossec/etc/rules/saas_rules.xml`:

---

### Parte 2 — O Ataque

**Script de fraude** `/opt/fraud_sim.py`:

**O que o atacante fez:**
- IP `45.33.32.156` (Linode/datacenter — indica automação)
- Exportou 7 relatórios da plataforma às 15:49 (reconhecimento)
- Realizou 10 resgates de 50.000 pontos em menos de 5 segundos
- Total fraudado: 500.000 pontos

---

### Parte 3 — Detecção

| Regra | Level | Descrição | Count |
|---|---|---|---|
| 110001 | 3 | Transação registrada | - |
| 110002 | 10 | Exportação de relatório | 7 |
| 110003 | 12 | Possível fraude - múltiplos resgates | 32 |

---

### Parte 4 — Investigação no Dashboard

1. `Threat Hunting` → identificados 32 alertas level 12 no grupo saas
2. `Events → rule.id: 110003` → detalhes do ataque
3. `data.ip: 45.33.32.156` → escopo do ataque
4. `data.dstuser: user_001 AND rule.id: 110002` → 7 exportações às 15:49
5. `Agents → ubunt` → 88 CVEs críticos, score CIS 47%
6. `Generate Report` → relatório PDF gerado

---

### Parte 5 — Linha do tempo
15:49:32 — Atacante exporta 7 relatórios (vazamento de dados)

16:00:46 — Inicia resgates em loop (10x 50.000 pts)

16:00:48 — Wazuh detecta e gera alerta crítico (rule 110003)

16:00:54 — Segunda confirmação do ataque (firedtimes: 2)

---
### Parte 6 — Conclusão

| Item | Detalhe |
|---|---|
| Classificação | True Positive |
| Técnica MITRE | T1657 — Financial Theft |
| Dados vazados | 7 relatórios exportados |
| Fraude financeira | 500.000 pontos |
| Exposição LGPD | Possível |

**Ações de resposta:**
- IP bloqueado via Active Response
- Conta user_001 suspensa
- Time de fraude notificado
- DPO notificado
- Ticket aberto para reversão das transações
- Recomendação: rate limiting na API de resgates

