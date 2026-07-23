# Painel Kanban de Gestão de OS — WonderLab (versão single-file)

Aplicação **single-file, offline-first** para acompanhar em Kanban as Ordens
de Serviço (OS) geradas no WonderLab (`Criador de relatórios Lab/index.html`),
com prazo (SLA) automático por prioridade, indicadores e cadastro de equipe.

## Atenção: existem DOIS painéis de gestão nesta pasta

Este repositório tem dois sistemas distintos que resolvem o mesmo problema
(gestão de OS em Kanban) e não devem ser confundidos nem misturados:

| | `painel-gestao.html` (este arquivo) | `painel-gestao/` (pasta, backend + frontend) |
|---|---|---|
| Arquitetura | single-file, sem servidor, sem build | FastAPI + SQLite (`backend/`) servindo um frontend próprio (`frontend/admin.html`) |
| Onde ficam os dados | IndexedDB do navegador — local a cada dispositivo/perfil | banco SQLite central no servidor |
| Como a OS chega | importação manual do JSON exportado do WonderLab | botão **"☁ Enviar ao Painel de Gestão"** no próprio WonderLab, via rede |
| Multi-dispositivo / multi-sede (Itajaí ↔ Balneário Camboriú) | não — cada navegador tem sua própria base, sem sincronização | sim, com o servidor acessível pela rede/VPN (ver `painel-gestao/README.md`) |
| Vocabulário de estágios do Kanban | Recebido, Em Diagnóstico, **Consignação**, Pronto / Testado, Entregue ao Cliente, **Sem Reparo** | Recebido, Em Diagnóstico, Em Reparo, Aguardando Peça, Pronto / Testado, Entregue ao Cliente |

Os dois **não compartilham dados** entre si — são bases separadas. Note
também que o vocabulário de estágios já divergiu entre os dois (este arquivo
renomeou "Aguardando Peça" → "Consignação" e adicionou "Sem Reparo"; o
backend em `painel-gestao/` ainda usa o conjunto original). Se a intenção for
manter só um dos dois como caminho oficial, vale reconciliar isso antes de
depender dos dois em paralelo.

Este README documenta **apenas o arquivo único `painel-gestao.html`**. Para o
sistema com backend, veja `painel-gestao/README.md`. Para o gerador de
relatório de bancada (WonderLab em si), veja o `README.md` na raiz da pasta.

## Características

- 100% no navegador — sem servidor, sem build, sem instalação. Dados no
  IndexedDB do navegador (banco `wonderlab_painel`) + localStorage (técnicos
  cadastrados, tema claro/escuro).
- Quadro **Kanban** com 6 colunas: Recebido, Em Diagnóstico, Consignação,
  Pronto / Testado, Entregue ao Cliente, Sem Reparo. Arrastar-e-soltar entre
  colunas e para reordenar dentro da mesma coluna.
- **Lista de OS** em tabela, com busca, filtros (técnico/prioridade) e
  colunas ordenáveis (Nº OS, Cliente, Embarcação, Status, Prioridade,
  Responsável, Entrada, Prazo, Equip.).
- **Indicadores**: total de OS, abertas/entregues no mês, tempo médio de
  execução (entrada → entrega), OS por status/técnico/prioridade, total de
  equipamentos e equipamentos sem reparo, e lista de OS com SLA expirado.
- **Equipe**: cadastro simples de técnicos, preenchido automaticamente a
  partir do campo "Responsável Técnico (Lab)" de cada OS importada.
- Tema claro/escuro (persistido em localStorage), com favicon e cores de
  controles nativos (date picker, scrollbar) ajustadas ao tema ativo.
- Backup/restauração manual via JSON ("Exportar backup do painel" /
  "Restaurar backup" na barra lateral) — necessário porque os dados vivem
  só no navegador local, sem servidor.

## Uso

1. Abra `painel-gestao.html` direto no navegador (duplo clique) ou sirva a
   pasta via HTTP — funciona das duas formas.
2. No WonderLab, exporte a OS em JSON.
3. Aqui, clique em **"Importar OS (JSON)"** e selecione um ou mais arquivos.
   Reimportar a mesma OS (mesmo `id`) **atualiza os dados técnicos** sem
   resetar status de gestão, técnico responsável, prazo, "Pronto em (SLA)"
   ou notas internas já definidos aqui.

## Regra de SLA

- **Prazo (SLA)**: calculado automaticamente a partir da data de entrada +
  dias úteis por prioridade — Baixa 20, Normal 15, Alta 10, Urgente 5
  (segunda a sexta; não há calendário de feriados integrado). Recalcula
  automaticamente ao trocar a prioridade no modal de detalhe (visível antes
  de salvar); uma edição manual do campo é sempre respeitada enquanto a
  prioridade não mudar de novo.
- **Pronto em (SLA)**: preenchido automaticamente só na primeira vez que a
  OS entra em "Pronto / Testado" ou "Entregue ao Cliente". A partir daí é
  permanente — nunca é apagado ou recalculado sozinho ao mudar de coluna;
  só muda por edição manual no modal.
- **Veredito de SLA** (✓ cumprido / ⚠ descumprido): compara "Pronto em" com
  o Prazo e, uma vez confirmado, fica **travado** — continua visível no
  card, na lista e no modal mesmo que a OS depois volte para retrabalho ou
  vá para "Sem Reparo".

## Estrutura

```
painel-gestao.html   # app completo: HTML + CSS + JS + logo embutida (base64) + favicon
```

Não depende de `fonts/`, `lib/` ou `logo.png` — usa fontes do sistema e a
mesma logo já embutida em base64 (reaproveitada também como favicon da aba).

## Dados e backup

Tudo fica no IndexedDB do navegador — local a cada navegador/dispositivo,
sem sincronização entre eles (diferente do sistema com backend, que
centraliza num servidor). Use **"Exportar backup do painel"**
periodicamente e guarde o JSON gerado; **"Restaurar backup"** recarrega esse
JSON (mescla por `id`, sem apagar o que já estiver na base atual).

Os dois campos de arquivo (**"Importar OS (JSON)"** e **"Restaurar
backup"**, inclusive arrastar-e-soltar direto na página) aceitam os dois
formatos — um backup do painel ou uma OS individual do WonderLab — e tratam
cada um corretamente não importa por qual dos dois você escolha o arquivo.

---
Wonder Boat · WonderHUB.AI
