# second-project

Projeto demonstrativo no **n8n** que integra WhatsApp (via Z-API) com IA (Groq + Llama 3.3) e salva conversas no Google Sheets.

### O que este workflow faz

1. Recebe mensagem via **Webhook** da Z-API (endpoint `/message-zapi`)
2. Verifica se é mensagem individual (não grupo, newsletter, broadcast ou da API)
3. Extrai o **ID da conversa** (gerado pela Z-API/WhatsApp – geralmente um ID privado como "170059381137521@lid" para proteger privacidade), nome do remetente e texto da mensagem.
4. **Salva ou atualiza** no Google Sheets: coluna `Whatsapp` (chave) + `Nome`
5. Envia a mensagem para um **AI Agent** (Groq com Llama 3.3 70b-versatile)
   - System prompt: "Você é um agente de suporte, seja educado e utilize emojis em sua resposta."
   - Memória simples (últimas interações)
   - Ferramentas: Calculadora + Wikipedia
6. Recebe a resposta da IA e **envia de volta** para o usuário no WhatsApp via HTTP Request (Z-API)
7. Finaliza (o fluxo para grupos/newsletter/broadcast vai para um NoOp)

Objetivo: Mostrar uma automação real de **chatbot de suporte** no WhatsApp com IA, log simples no Sheets e resposta automática.

### Tecnologias / Serviços utilizados
- n8n (Webhook + LangChain nodes + HTTP Request)
- Z-API (integração com WhatsApp – usei trial de 2 dias para teste)
- Google Sheets (salva/atualiza contato + nome)
- Groq API (modelo Llama 3.3 70b-versatile)
- Ferramentas da IA: Calculadora e Wikipedia

### Como importar e usar no seu n8n

1. Abra sua instância do **n8n**
2. Vá em **Workflows** → **+** → **Import from File**
   - Selecione o arquivo: `second-project.json`
3. Ou importe direto pela URL (raw do GitHub)
4. **Configure as credenciais** (elas não vêm no JSON por segurança):
- **Google Sheets OAuth2** – conecte sua conta Google
- **Groq API** – insira sua chave Groq

5. **Atualize as partes sensíveis**:
- No node **"Append or update row in sheet"** → troque o `YOUR_GOOGLE_SHEET_ID_HERE` pelo ID da sua planilha
- No node **"HTTP Request"** (envio da resposta):
  - URL → coloque sua instance e token da Z-API: `https://api.z-api.io/instances/SUA_INSTANCE/token/SEU_TOKEN/send-text`
  - Header `client-token` → coloque seu client-token real da Z-API

### Estrutura esperada da planilha Google Sheets
Crie uma planilha com pelo menos estas colunas (na ordem):

| Whatsapp                        | Nome          |
|---------------------------------|---------------|
| 124346381137323@lid          | Maria Oliveira|

- `Whatsapp` recebe o **ID completo da conversa** (@lid) – é a chave única e estável usada pela Z-API para identificar e responder mensagens. Não revela o telefone real por privacidade do WhatsApp.

### Observações importantes
- O workflow está **desativado** por padrão (`active: false`) pois é uma versão para testes.
- Use **apenas números individuais** (o If filtra grupos, broadcasts, etc.)
- O trial da Z-API dura poucos dias → crie sua conta própria para uso real
- Não inclui tratamento de erros avançado nem loop de conversa infinita (é uma demo simples)
