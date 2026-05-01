# Sistema de Ouvidoria

Sistema para gestão de manifestações (denúncias) com análise assistida por IA, designação automática de ouvidores e fluxo estruturado de defesa, validação e recursos.

---

## Visão Geral

O sistema recebe manifestações de denunciantes, processa-as por meio de um microserviço de IA, designa automaticamente ouvidores responsáveis pela análise e conduz o caso por um fluxo bem definido até a decisão final. O processo contempla defesa do denunciado, validação por instância superior (Ouvidor Geral) e direito a recurso por ambas as partes.

---

## Atores do Sistema

| Ator | Descrição |
|------|-----------|
| **Denunciante** | Pessoa que submete a manifestação. Pode anexar provas, recorrer uma vez e avaliar o atendimento. |
| **Denunciado** | Pessoa contra quem a denúncia é feita. É notificado apenas quando o caso avança para a fase de defesa. Pode apresentar defesa com provas e recorrer uma vez. |
| **Ouvidor** | Ente externo responsável pela análise das provas e elaboração de relatórios. Considerado isento de conflito de interesse. |
| **Ouvidor Geral** | Instância superior. Valida todo relatório final, podendo confirmar, alterar ou repassar para novo ouvidor. |
| **Gestor** | Recebe versão censurada da manifestação (apenas em casos sem conflito de interesse) e é notificado da penalidade ao final. Não tem poder de aprovação. |
| **Microserviço de IA** | Auxilia na análise inicial, detecta conflito de interesse de gestores e gera versões censuradas. |

---

## Regras Gerais

### Designação de Ouvidores

- Ouvidor é sorteado de uma pool composta pelos **3 ouvidores com menor número de casos ativos**.
- Apenas **um ouvidor** interage em cada fase de análise.
- Recursos são designados a um **novo ouvidor**, que analisa o caso do zero (sem acesso à conclusão anterior, evitando viés).

### Conflito de Interesse

- Detectado pelo microserviço de IA.
- Aplica-se apenas a **gestores** (ouvidores são considerados externos e isentos).
- Quando detectado, gestores **não são notificados** em nenhuma etapa.

### SLAs (Prazos)

| Etapa | Prazo |
|-------|-------|
| Análise inicial pelo ouvidor (inclui confirmação das sugestões da IA) | 10 dias |
| Apresentação de defesa pelo denunciado | 10 dias |
| Análise da defesa e parecer final | 10 dias |
| Validação pelo Ouvidor Geral | 10 dias |
| Análise de recurso por novo ouvidor | 10 dias |

### Recursos

- Denunciante e denunciado têm direito a **um recurso cada**.
- Recurso só pode ser aberto **após a decisão do Ouvidor Geral**.
- Recurso exige apresentação de **novas provas**.
- Se ambas as partes recorrerem do mesmo relatório, as provas são **somadas** e analisadas pelo **mesmo novo ouvidor**.

---

## Fluxo Completo

### Fase 1 — Submissão

1. Denunciante submete a manifestação com provas iniciais.
2. Microserviço de IA processa a manifestação:
   - Gera sugestões de análise.
   - Detecta conflito de interesse com gestores.
   - Produz versão censurada (quando aplicável).
3. Ouvidor é designado a partir da pool dos 3 com menos casos ativos.
4. Ouvidor confirma, modifica ou rejeita as sugestões da IA *(dentro do prazo da Fase 2)*.

**Notificação aos gestores:**

- **Sem conflito de interesse:** gestores recebem a versão censurada da manifestação. Sua aprovação **não é necessária** para o prosseguimento do caso.
- **Com conflito de interesse:** gestores **não são notificados**.

---

### Fase 2 — Análise do Ouvidor

**Prazo: 10 dias.**

O ouvidor analisa as provas, podendo registrar observações e comentários sobre cada uma. Existem dois caminhos possíveis:

#### Caminho A — Negação por falta de provas

- Caso o ouvidor entenda que não há provas suficientes para prosseguir, o caso é encerrado **sem relatório final**.
- O **denunciado não é notificado** (nunca chegou a entrar no processo).
- O denunciante pode recorrer (vide Fase 5).

#### Caminho B — Relatório preliminar

- Ouvidor preenche relatório preliminar:
  - **Acatar** a denúncia: definir penalidade proposta.
  - **Negar** a denúncia: justificar a decisão.
- O caso avança para a Fase 3.

---

### Fase 3 — Defesa do Denunciado

**Prazos: 10 dias para defesa + 10 dias para análise.**

1. Denunciado é notificado oficialmente e tem acesso às provas e ao relatório preliminar.
2. Denunciado apresenta defesa com provas próprias *(10 dias)*.
3. **O mesmo ouvidor** da Fase 2 analisa a defesa *(10 dias)*.

> A escolha de manter o mesmo ouvidor é deliberada: como ele já tem direito a negar a denúncia por falta de provas na Fase 2, a continuidade na análise da defesa preserva coerência. Caso a parte se sinta prejudicada, o instrumento de recurso (Fase 5) garante a designação de novo ouvidor.

4. Ouvidor preenche o **relatório final**, com decisão e penalidade (se aplicável).

---

### Fase 4 — Validação do Ouvidor Geral

**Prazo: 10 dias.**

O Ouvidor Geral é **sempre notificado** ao final da Fase 3 e tem três opções:

1. **Validar** o relatório final como está.
2. **Alterar** o parecer, registrando sua própria decisão.
3. **Repassar** o caso a um novo ouvidor para nova análise.

#### Regra de não-loop

- O Ouvidor Geral pode repassar **apenas uma vez** por caso.
- Após o repass, o novo ouvidor tem 10 dias para emitir novo relatório.
- Esse relatório retorna ao Ouvidor Geral, que **é obrigado a emitir decisão final** (validar ou alterar), sem possibilidade de novo repass.

---

### Fase 5 — Recursos

**Prazo: 10 dias por recurso.**

O recurso só pode ser aberto **após a decisão do Ouvidor Geral**. Tanto denunciante quanto denunciado têm direito a **um recurso cada**, mediante apresentação de novas provas.

#### Cenários

- **Apenas uma parte recorre:** um novo ouvidor é designado e analisa o caso do zero, considerando provas anteriores e novas.
- **Ambas as partes recorrem do mesmo relatório:** as provas das duas partes são **somadas** e analisadas pelo **mesmo novo ouvidor**.

O relatório resultante do recurso passa novamente pela validação do Ouvidor Geral *(Fase 4, com mesmas regras de não-loop)*.

---

### Fase 6 — Encerramento

1. O gestor é notificado da penalidade definitiva.
2. A **execução da penalidade ocorre fora do sistema**.
3. O denunciante avalia o atendimento:
   - Ao optar por **não recorrer**.
   - Ao **fim do recurso**, independentemente do resultado (deferido ou indeferido).

---

## Diagrama Resumido

```
[Denunciante] → submete manifestação
       ↓
[IA] analisa + detecta conflito + gera versão censurada
       ↓
[Ouvidor sorteado] confirma/modifica sugestões da IA
       ↓
   ┌───────────────────────────────────────┐
   │ Sem conflito → Gestores notificados   │
   │ Com conflito → Gestores não notificados│
   └───────────────────────────────────────┘
       ↓
[Ouvidor] analisa provas (10 dias)
       ↓
   ┌─────────────────────────────────────┐
   │ A) Nega por falta de provas → fim   │── (cabe recurso do denunciante)
   │ B) Emite relatório preliminar       │
   └─────────────────────────────────────┘
       ↓ (caminho B)
[Denunciado] notificado + apresenta defesa (10 dias)
       ↓
[Mesmo Ouvidor] analisa defesa (10 dias) → relatório final
       ↓
[Ouvidor Geral] valida (10 dias)
   ├─ Valida → segue
   ├─ Altera parecer → segue
   └─ Repassa (1x) → novo ouvidor (10d) → Ouvidor Geral decide obrigatoriamente
       ↓
[Abre janela de recurso]
   ├─ Denunciante recorre (1x) ┐
   ├─ Denunciado recorre (1x)  ├─ provas somadas se ambos → novo ouvidor → Ouvidor Geral
   └─ Nenhum recorre           ┘
       ↓
[Gestor notificado da penalidade] (execução externa)
       ↓
[Denunciante avalia o atendimento]
```

---

## Considerações de Design

- **Isenção dos ouvidores:** o sistema parte da premissa de que ouvidores são entes externos e, portanto, não estão sujeitos a conflito de interesse. A detecção automática só verifica gestores.
- **Anti-viés em recursos:** novos ouvidores designados em recursos analisam o caso do zero, sem acesso à conclusão prévia.
- **Proteção do denunciado:** o denunciado só é cientificado quando o caso efetivamente avança para a fase de defesa, evitando exposição em casos arquivados por falta de provas.
- **Limite de recursos:** cada parte tem direito a um único recurso, evitando uso protelatório do mecanismo.
- **Garantia de encerramento:** a regra de não-loop do Ouvidor Geral assegura que todo caso chegue a uma decisão final em tempo determinado.