# 🚀 Agente Inteligente de Atendimento – Telegram + n8n + LLM + MongoDB

![Status](https://img.shields.io/badge/status-operacional-brightgreen)  
![Stack](https://img.shields.io/badge/stack-n8n%20%7C%20Telegram%20Bot%20API%20%7C%20LLM%20%7C%20MongoDB-blue)  

Este projeto implementa um **agente de atendimento automatizado** integrado ao Telegram, utilizando **n8n**, **LLM (OpenAI)** e **MongoDB** para armazenamento de sessões e histórico.  
O agente conduz conversas, persiste pedidos e retorna respostas inteligentes.

---

## 🎯 Objetivo do Projeto

Criar um **agente de atendimento inteligente** que:

- Recebe mensagens via Telegram;    
- Processa entradas com uma LLM;  
- Persiste histórico/pedidos no MongoDB;  
- Finaliza pedidos e reinicia sessão limpa ao término.

---

## 📦 Entregáveis

O pacote entregue contém:

1. **Link público do chat (para testar o agente):**  
   > `t.me/marmitatestebot`  

2. **Prints do fluxo no n8n**:
![WhatsApp Image 2025-11-17 at 23 24 29](https://github.com/user-attachments/assets/416aa414-4e45-4fca-a384-fa0a618baf1e)
<img width="1877" height="921" alt="image" src="https://github.com/user-attachments/assets/00cd6443-fa37-4a03-a4a6-e5fcc40b1c0b" />
<img width="1879" height="923" alt="image" src="https://github.com/user-attachments/assets/004c20e3-bd56-4863-9df1-180b3ee583f5" />
Prompt usado:
Você é o atendente virtual da Marmitaria Sabor Caseiro by LogManager.
Seu objetivo é montar e confirmar pedidos de marmitas.

CARDÁPIO:
P1 - Marmita Tradicional - R$ 18
P2 - Marmita Fitness - R$ 22
P3 - Marmita Veggie - R$ 20
P4 - Parmegiana + Arroz + Purê - R$ 25
P5 - Bife Acebolado + Arroz + Feijão + Salada - R$ 23

ADICIONAIS (+R$ 3):
Ovo, Queijo, Batata Frita

REGRAS:
1. Sempre ofereça o cardápio quando o usuário iniciar.
2. Aceite personalizações (ex.: tirar salada, trocar arroz por macarrão, adicionar ovo).
3. Seu objetivo é coletar dados passo a passo:
- Nome do cliente
- Itens do pedido (produto, quantidade, observações)
- Confirmar o pedido
- Calcular ou validar o valor total
4. REGRA FUNDAMENTAL:
Só chame a função "finalizarPedido" quando TODOS os dados estiverem completos, válidos e confirmados pelo cliente.
ANTES disso:
- Continue fazendo perguntas
- Peça correções quando necessário
- Nunca chame a função de forma parcial
- Nunca retorne JSON diretamente ao usuário

Quando tudo estiver OK:
Agradeça pelo pedido depois chame a function "finalizarPedido".
<img width="1878" height="923" alt="image" src="https://github.com/user-attachments/assets/a936f861-723f-4dd2-8e09-2e07140de012" />
<img width="1878" height="921" alt="image" src="https://github.com/user-attachments/assets/8340ec9e-81c2-4212-9984-c9b8fc40aff7" />
<img width="1878" height="921" alt="image" src="https://github.com/user-attachments/assets/0ffb51c5-b424-4ed8-b22e-8288137d05d0" />
<img width="1878" height="924" alt="image" src="https://github.com/user-attachments/assets/bf79521d-a9bc-4c3f-85c4-fc3084ec9302" />
obs: A entrada do switch ta vermelha por que nesse ponto ela ainda não "existe" mas é a saida da tool
<img width="1877" height="925" alt="image" src="https://github.com/user-attachments/assets/a428b47b-dfd7-47cd-9d94-f2bc255537bc" />
<img width="1877" height="923" alt="image" src="https://github.com/user-attachments/assets/ea95c7f0-039f-4d95-b4d5-093547610f85" />
<img width="1877" height="925" alt="image" src="https://github.com/user-attachments/assets/cc56d942-9fc2-42e9-90be-e540cdfd7d84" />
<img width="1878" height="922" alt="image" src="https://github.com/user-attachments/assets/a981f8e4-556d-47ab-9fc1-1809c5d79cc9" />
<img width="1875" height="922" alt="image" src="https://github.com/user-attachments/assets/e3b20631-c002-445e-aaee-b0e2c8db1a35" />

---

## 🧠 Arquitetura Geral

- Usuário Telegram
- ↓
- Telegram Bot API
- ↓
- n8n (workflow)
- ├─ LLM (OpenAI)
- ├─ MongoDB Chat Memory (load/save)
- └─ Code Tool (formata saida)
- ↓
- Resposta ao Telegram (em caso de pedido confirmado salva os dados no google sheets)

---

## ⚙️ Funcionamento do Fluxo

1. **Recepção**: `Telegram Trigger` no n8n recebe a mensagem.    
2. **Memória**: `MongoDB Chat Memory` carrega histórico pelo `sessionId`.  
3. **LLM**: Envia histórico + mensagem atual ao modelo; usa *code tool* para gerar `finalizarPedido` quando o pedido estiver pronto.
4. **Resposta**: `Telegram Send Message` retorna a resposta ao usuário. 
5. **Persistência**: Se pedido confirmado, grava em Google Sheets; caso contrário, atualiza sessão.     

---

### 🧪 Como Testar o Bot

Siga este passo a passo para testar o agente de atendimento integrado com Telegram + n8n + MongoDB.

Acessar o Bot no Telegram

Clique no link abaixo (ou busque o nome do bot diretamente no Telegram):

https://t.me/marmitatestebot

Inicie uma conversa com ele e seja guiado por ele respondendo as perguntas.

---

## 🛠️ Scripts Dos Nodes

### `src/formatUser.js`
Formata o objeto recebido do Telegram antes de salvar:
```js
// Codigo da tool
const dados = $input.item.json;

const pedidoFinalizado = {
  cliente: dados.cliente || '',
  itens: dados.itens || [],
  valorTotal: dados.valorTotal || 0,
  confirmado: dados.confirmado || false,
  dataHora: new Date().toISOString(),
  sessionId: dados.sessionId || 'unknown'
};

return JSON.stringify(pedidoFinalizado);


----------------------------------------------------


// Codigo do code node
const logs = $input.all().map((item) => item.json);

const organized = logs.flatMap((log) => {
  const { intermediateSteps } = log;

  const step = intermediateSteps?.[0];
  if (!step) return [];

  const text = step.action?.log ?? "";
  let extracted = null;

  try {
    const match = text.match(/\{[\s\S]*\}/);
    if (match) extracted = JSON.parse(match[0]);
  } catch (e) {
    extracted = null;
  }

  if (!extracted || !Array.isArray(extracted.itens)) return [];

  return extracted.itens.map((item) => {
    return {
      cliente: extracted.cliente ?? null,
      produto: item.produto ?? null,
      quantidade: item.quantidade ?? null,
      observacoes: item.observacoes ?? null,
      valorTotal: extracted.valorTotal ?? null,
      confirmado: extracted.confirmado ?? null,
      id: extracted.id ?? null
    };
  });
});

return organized;
