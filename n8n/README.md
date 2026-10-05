# Documentação da pasta `n8n`

## Objetivo

Esta pasta centraliza os artefatos de orquestração do projeto no n8n. Ela representa a etapa entre a coleta de clientes no notebook Python (`rpa/extrair_clientes.ipynb`) e a geração da recomendação por perfil de investidor.

## Componentes esperados

- `workflow.json` (**artefato esperado do desafio**): exportação versionada e reproduzível do workflow MVP.
- Outros arquivos `.json` opcionais: variações de fluxo, experimentos ou versões com integrações extras (ex.: LLM), quando fizer sentido para evolução do projeto.

> Se `workflow.json` ainda não existir, mantenha este arquivo como referência do contrato e da estrutura esperada para publicação do fluxo principal.

## Contrato de entrada e saída

### Entrada (Webhook)

- Método: `POST`
- Conteúdo: `application/json`
- Estrutura mínima esperada:

```json
{
  "clientes": [
    {
      "nome": "Cliente Exemplo",
      "email": "cliente@example.com",
      "saldo": "10000",
      "perfil": "Moderado"
    }
  ]
}
```

Campos mínimos por cliente: `nome`, `email`, `saldo`, `perfil`.

### Saída

- Lista de clientes com recomendação correspondente ao perfil.
- Formato pode variar conforme os nós finais (HTTP Response, Set, Code, armazenamento etc.), mas deve manter rastreabilidade entre cliente de entrada e mensagem gerada.

## Fluxo esperado dos nós (MVP)

1. **Webhook** recebe os dados enviados pelo notebook Python.
2. **Leitura do CSV** consulta `docs/data.csv` (via GitHub Pages).
3. **Cruzamento por perfil** mapeia cliente para investimento recomendado.
4. **Geração de mensagem** usa template estático por perfil.
5. **Resposta/saída** retorna o resultado processado.

## Como importar e testar ponta a ponta

1. Abra o n8n (Cloud ou local) e use a opção **Import workflow**.
2. Selecione `n8n/workflow.json` (quando o arquivo estiver disponível no repositório).
3. Copie a URL do nó Webhook e atualize no notebook `rpa/extrair_clientes.ipynb` (variável de webhook).
4. Execute o notebook para coletar clientes e disparar o `POST` para o n8n.
5. Verifique no painel de execução do n8n se houve:
   - recebimento da lista de clientes;
   - leitura do CSV de investimentos;
   - cruzamento correto por perfil;
   - geração de uma mensagem por cliente.

## Decisões técnicas

- **Webhook como contrato entre sistemas:** simplifica a integração Python/RPA ↔ n8n com baixo acoplamento.
- **CSV em GitHub Pages:** fonte simples, transparente e fácil de atualizar para o MVP.
- **Separação por camadas:** coleta (Python), orquestração (n8n) e apresentação da recomendação em etapas independentes.
- **Mensagens estáticas no MVP:** prioriza validação rápida do fluxo antes de introduzir complexidade de agente/LLM.
- **Validação e erro mínimo:** rejeitar/tratar payload sem `nome`, `email`, `saldo` ou `perfil`; reforçar que o resultado é demonstrativo e não consultoria financeira.

## Segurança e versionamento

- Não versionar credenciais, tokens, chaves ou segredos em exportações do n8n.
- Evitar URLs privadas/sensíveis (ex.: túneis temporários) nos arquivos versionados.
- Utilizar apenas dados fictícios, sem dados pessoais reais.
- Revisar o JSON exportado antes de commit para remover metadados sensíveis.
