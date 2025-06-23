# Monitoramento de Máquinas Virtuais no Azure – Guia de Estudo e Prática

## 📌 Objetivo
Demonstrar como configurar o monitoramento de máquinas virtuais no Azure para garantir visibilidade, controle e resposta proativa a eventos críticos — como a exclusão acidental de uma VM. Baseado nos tópicos da certificação **AZ-104**.

---

## 🧱 Conceitos Fundamentais

### Azure Monitor
Serviço centralizado para coleta, análise e atuação sobre métricas e logs de recursos no Azure.
- Reúne dados de performance, logs de diagnóstico e eventos de atividade.
- Suporta integração com alertas, dashboards e automações.

### Log Analytics Workspace
Repositório onde os dados de log são armazenados e consultados usando a linguagem KQL (Kusto Query Language).
- Vincula recursos como VMs, redes, firewalls etc.
- Usado por serviços como Azure Monitor e Sentinel.

### Activity Log (Log de Atividades)
Log de operações realizadas no nível de recurso/assinatura (ex: exclusão de VM, criação de recursos).
- Importante para auditoria e segurança.
- Pode ser integrado a alertas.

### Diagnostic Settings (Configurações de Diagnóstico)
Permite encaminhar logs e métricas para:
- Log Analytics
- Event Hub
- Conta de Armazenamento

### Alertas e Action Groups
- Alertas são criados com base em métricas ou logs.
- Podem acionar notificações, runbooks, webhooks ou tickets.

---

## 🛠️ Como Implementar o Monitoramento (Passo a Passo)

### 1. Criar um Log Analytics Workspace
📍 Portal: `Monitor > Log Analytics workspaces`

1. Clique em "Criar".
2. Escolha uma assinatura e grupo de recursos.
3. Dê um nome (ex: `log-workspace-monitoring`).
4. Defina a região.
5. Clique em "Revisar + Criar".

### 2. Conectar a VM ao Workspace
📍 Portal: `Máquinas Virtuais > [sua VM] > Monitoramento > Logs`

1. Clique em "Habilitar logs".
2. Selecione o workspace criado.
3. Confirme a vinculação.

### 3. Configurar Diagnostic Settings
📍 Portal: `Máquinas Virtuais > [sua VM] > Diagnóstico > Configurações de diagnóstico`

1. Clique em "Adicionar configuração".
2. Dê um nome.
3. Marque logs e métricas a serem enviados.
4. Selecione o destino: Log Analytics.
5. Salve a configuração.

### 4. Criar Alerta para Exclusão de VM
📍 Portal: `Monitor > Alerts > Nova regra`

1. **Escopo:** escolha a assinatura.
2. **Condição:** selecione `Signal type: Activity Log` > `Operação: Delete Virtual Machine`.
3. **Ação:** selecione ou crie um Action Group (ex: envio de e-mail).
4. **Detalhes:** nome da regra, gravidade e descrição.
5. Clique em "Criar alerta".

### 5. Criar Action Group
📍 Portal: `Monitor > Action Groups`

1. Clique em "Criar".
2. Nomeie e defina a região.
3. Adicione ações: e-mail, webhook, ITSM, runbook.
4. Salve.

---

## 🧪 Exemplo prático: Alerta para Exclusão de VM

- Evento monitorado: `Microsoft.Compute/virtualMachines/delete`
- Fonte: Activity Log
- Condição: qualquer exclusão de VM
- Ação: enviar e-mail ao administrador
- Gravidade: Crítico (Sev 0 ou 1)

---

## 📦 Exemplos com CLI e Bicep

### Exemplo: Conectar VM ao Log Analytics Workspace via Azure CLI
```bash
# Variáveis
RESOURCE_GROUP="meu-grupo"
VM_NAME="minhaVM"
WORKSPACE_NAME="meu-workspace"

# Obter ID do workspace
WORKSPACE_ID=$(az monitor log-analytics workspace show \
  --resource-group $RESOURCE_GROUP \
  --workspace-name $WORKSPACE_NAME \
  --query id -o tsv)

# Conectar a VM
az monitor diagnostic-settings create \
  --name "vm-diag" \
  --resource "/subscriptions/<ID>/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Compute/virtualMachines/$VM_NAME" \
  --workspace $WORKSPACE_ID \
  --metrics '[{"category":"AllMetrics","enabled":true}]' \
  --logs '[{"category":"VMPerformanceCounters","enabled":true},{"category":"WindowsEventLogs","enabled":true}]'
```

### Exemplo: Criar alerta de exclusão de VM com CLI
```bash
az monitor activity-log alert create \
  --name "alerta-exclusao-vm" \
  --resource-group $RESOURCE_GROUP \
  --scopes "/subscriptions/<ID>" \
  --condition "category=Administrative and operationName=Microsoft.Compute/virtualMachines/delete" \
  --action-group "meu-grupo-acao" \
  --description "Alerta para exclusão de VMs"
```

### Exemplo: Conectar VM ao Log Analytics via Bicep
```bicep
resource workspace 'Microsoft.OperationalInsights/workspaces@2021-06-01' existing = {
  name: 'meu-workspace'
}

resource diagSetting 'Microsoft.Insights/diagnosticSettings@2021-05-01-preview' = {
  name: 'vm-diag'
  scope: resourceId('Microsoft.Compute/virtualMachines', 'minhaVM')
  properties: {
    workspaceId: workspace.id
    logs: [
      {
        category: 'VMPerformanceCounters'
        enabled: true
      }
    ]
    metrics: [
      {
        category: 'AllMetrics'
        enabled: true
      }
    ]
  }
}
```

## 🔧 Ferramentas e Recursos Usados
- Azure Monitor
- Log Analytics Workspace
- Activity Log
- Diagnostic Settings
- Alertas
- Action Groups
- Azure CLI (opcional)

---

## 💡 Dicas para a Prova AZ-104
- Saiba configurar um Diagnostic Setting.
- Entenda o fluxo: VM → Diagnostic → Log Analytics → Alert → Ação.
- Conheça os tipos de sinal para alertas (métrica, log, atividade).
- Pratique a criação de alertas via portal e CLI.

---

## 📚 Referências Oficiais
- [Monitoramento com Azure Monitor](https://learn.microsoft.com/pt-br/azure/azure-monitor/overview)
- [Log Analytics Workspace](https://learn.microsoft.com/pt-br/azure/azure-monitor/logs/log-analytics-workspace-overview)
- [Alertas com Activity Log](https://learn.microsoft.com/pt-br/azure/azure-monitor/alerts/activity-log-alerts)
- [Configurar Diagnóstico em VMs](https://learn.microsoft.com/pt-br/azure/azure-monitor/vm/vminsights-enable-overview)
- [Criar Action Group](https://learn.microsoft.com/pt-br/azure/azure-monitor/alerts/action-groups)

