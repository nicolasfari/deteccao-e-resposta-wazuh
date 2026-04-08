# Resposta ao Incidente — INC-2026-001

**Data:** 08/04/2026  
**Severidade:** Crítica (Level 12)  
**Status:** Contido  
---

## Resumo Executivo
Durante monitoramento contínuo via Wazuh, foram detectados dois alertas críticos originados do agente `ubunt` (192.168.0.150), servidor SaaS da plataforma de incentivos. O atacante, utilizando IP de datacenter externo, realizou reconhecimento via exportação de relatórios e em seguida executou fraude automatizada de resgates de pontos.
---

## Alertas Disparados

| Regra | Level | Descrição | Ocorrências |
|---|---|---|---|
| 110002 | 10 | Exportação de relatório detectada | 7 |
| 110003 | 12 | Possível fraude - múltiplos resgates | 32 |

---

## Linha do Tempo

- 15:49:32 — Regra 110002 dispara — user_001 exporta relatório via IP externo
- 15:49:33 — Regra 110002 dispara — 2ª exportação
- 15:49:34 — Regra 110002 dispara — 3ª exportação
- 15:49:35 — Regra 110002 dispara — 4ª exportação
- 15:49:36 — Regra 110002 dispara — 5ª exportação
- 15:49:37 — Regra 110002 dispara — 6ª exportação
- 15:49:38 — Regra 110002 dispara — 7ª exportação (total: 7 relatórios exportados)
- 16:00:46 — Regra 110003 dispara — resgates em loop iniciados
- 16:00:48 — Wazuh confirma fraude (5 resgates em 2 segundos)
- 16:00:54 — Segunda confirmação (firedtimes: 2)
- 16:00:54 — Total: 10 resgates de 50.000 pontos

---

## Indicadores de Comprometimento (IoCs)

| Indicador | Valor |
|---|---|
| IP atacante | 45.33.32.156 |
| Tipo de IP | Datacenter Linode (automação) |
| Usuário comprometido | user_001 |
| Ações realizadas | exportar_relatorio, resgate_premio |
| Horário inicial | 15:49:32 |
| Horário final | 16:00:54 |

---

## Análise do Ataque

### Alerta 110002 — Exportação de Relatórios

- **O que aconteceu:** user_001 exportou 7 relatórios via IP externo
- **Por que é suspeito:** usuários legítimos acessam de IPs internos
- **Impacto:** possível vazamento de dados de clientes
- **Classificação:** True Positive
- **MITRE ATT&CK:** T1530 — Data from Cloud Storage

### Alerta 110003 — Fraude de Resgates

- **O que aconteceu:** user_001 realizou 10 resgates de 50.000 pontos em 5 segundos
- **Por que é suspeito:** impossível ser humano — padrão de bot
- **Impacto:** 500.000 pontos fraudados
- **Classificação:** True Positive
- **MITRE ATT&CK:** T1657 — Financial Theft

---

## Padrão do Ataque

O atacante seguiu sequência clássica:

- **1º Reconhecimento** — exportou relatórios para mapear a plataforma
- **2º Exploração** — usou os dados para fraudar resgates em loop

---

## Impacto

| Item | Detalhe |
|---|---|
| Dados vazados | 7 relatórios exportados |
| Fraude financeira | 10 x 50.000 = 500.000 pontos |
| Conta afetada | user_001 |
| Exposição LGPD | Possível — avaliar conteúdo dos relatórios |

---

## Ações de Resposta

### Contenção (imediato)
- [ ] IP 45.33.32.156 bloqueado via Active Response
- [ ] Conta user_001 suspensa

### Notificações
- [ ] Time de fraude notificado
- [ ] DPO notificado (possível violação LGPD)
- [ ] Gestão notificada

### Remediação
- [ ] Ticket aberto para reversão das 10 transações fraudulentas
- [ ] Auditoria do conteúdo dos 7 relatórios exportados
- [ ] Verificar se outros usuários foram afetados pelo mesmo IP

### Melhorias recomendadas
- [ ] Implementar rate limiting na API de resgates
- [ ] Bloquear acessos de IPs de datacenter na API
- [ ] Adicionar MFA para resgates acima de 10.000 pontos
- [ ] Criar alerta para exportações fora do horário comercial

---

## Postura da Máquina Afetada

Identificado durante investigação do agente `ubunt`:

| Item | Situação |
|---|---|
| Vulnerabilidades críticas | 88 CVEs críticos não corrigidos |
| Score CIS Benchmark | 47% (abaixo do mínimo aceitável) |
| MITRE Top Tactic | Defense Evasion (37 alertas) |

**Recomendação:** aplicar patches críticos com urgência e corrigir as 119 falhas identificadas no CIS Benchmark.

---

## Lições Aprendidas

1. A sequência reconhecimento → exploração foi detectada apenas em análise retroativa — implementar correlação automática entre 110002 e 110003
2. A máquina afetada possui 88 CVEs críticos — aumenta superfície de ataque
3. Rate limiting na API teria impedido o ataque antes da detecção

---

## Referências

- Regras Wazuh: `rules/saas_rules.xml`
- Relatório gerado pelo Wazuh: `imgs/relatorio.pdf`
- MITRE T1657: https://attack.mitre.org/techniques/T1657
- MITRE T1530: https://attack.mitre.org/techniques/T1530
