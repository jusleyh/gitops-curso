# Desafio final

**Aula 4 · 15/10 · 55 minutos de execução + 2 minutos de apresentação por aluno ou dupla**

## Antes de começar

> Sem Kubernetes na sua máquina? Faça o desafio no navegador seguindo [killercoda.md](killercoda.md).

O instrutor acabou de publicar os arquivos do desafio. Traga-os para o seu fork:

1. No GitHub, abra o seu fork e clique em **Sync fork → Update branch**.
2. No terminal, dentro da pasta `gitops-curso`:

```bash
git pull
```

## Regra

Depois de criar as Applications, **toda correção é feita pelo Git** (editar, commit, push).
Pode usar `kubectl get`, `describe`, `logs` e a interface do Argo CD para investigar,
mas **não** use `kubectl apply`, `edit`, `scale`, `patch` ou `delete` para corrigir.

---

## Parte A — Novo ambiente de homologação (3 pontos)

Crie o ambiente **hml** sem rodar nenhum `kubectl apply`:

- Pasta `apps/web/overlays/hml/` com `kustomization.yaml` e `index.html`.
- Namespace `web-hml`, **2 réplicas**, página mostrando **Ambiente: HML**.
- Application `argocd/apps/web-hml.yaml`, com sync automático, prune e self-heal.

Dica: copie o overlay de dev e a Application de dev como ponto de partida.

**Pronto quando:** `web-hml` aparece sozinha na interface (criada pela **root**), Synced e Healthy,
e http://localhost:8084 mostra a página HML
(`kubectl port-forward svc/web -n web-hml 8084:80`).

## Parte B — Diagnóstico de uma aplicação quebrada (4 pontos)

1. Em `argocd/desafio/web-desafio.yaml`, troque `SEU_USUARIO` pelo seu usuário do GitHub,
   faça commit e push.
2. Crie a Application:

```bash
kubectl apply -f argocd/desafio/web-desafio.yaml
```

Ela aponta para `apps/web/overlays/desafio`, que tem **três problemas**.
Você só vê o próximo problema depois de corrigir o anterior. Para cada um, preencha:

| # | Sintoma (o que você viu) | Evidência (onde viu) | Causa | Correção (commit) |
|---|--------------------------|----------------------|-------|-------------------|
| 1 | Application não sincronizava. | Erro durante geração dos manifests. | Arquivo referenciado como index.htm. | Alterado para index.html. |
| 2 | Pod não iniciava. | ImagePullBackOff. | Tag inválida 1.27-alpinee. | Alterado para 1.27-alpine. |
| 3 | Aplicação não respondia corretamente. | Teste da página após deploy. | Service apontando para porta incorreta (8080). | Ajustado targetPort para 80. |

Esta Application tem **sync manual**: depois de cada push, use REFRESH e SYNC.

**Pronto quando:** `web-desafio` está Synced e Healthy **e** a página responde:

```bash
kubectl port-forward svc/web -n web-desafio 8085:80
curl http://localhost:8085
```

Atenção: Healthy no Argo CD não garante que a aplicação responde. Teste sempre.

## Parte C — Operação do dia a dia (2 pontos)

1. Promova a **Versao: v3** para dev, depois hml, depois prod — um commit por ambiente.
- Não foi necessário promover para V3 pois todos os ambientes já estavam nessa versão.
   
2. Alguém escala prod para 8 réplicas no cluster. Mostre o que acontece e explique.
- Resposta: O ArgoCD detectou uma divergência entre o estado definido no Git e o estado atual do cluster. Como o self-heal está habilitado,
a alteração manual para 8 réplicas foi revertida automaticamente para o valor definido no repositório Git, restaurando o ambiente para 2 réplicas.

## Parte D — Apresentação (1 ponto)

Em 2 minutos, mostre:

- O problema da Parte B que foi mais difícil de achar: sintoma, evidência e correção.

O terceiro erro foi, sem dúvida, o mais difícil de identificar. Diferente dos dois primeiros, que apresentavam mensagens de erro mais evidentes, esse problema não impedia a aplicação de ser criada nem os pods de iniciarem corretamente. À primeira vista, tudo parecia estar funcionando: a Application estava sincronizada, os recursos existiam no cluster e o ambiente aparentava estar saudável.

A dificuldade surgiu porque o problema só apareceu durante a validação funcional da aplicação. Mesmo com os recursos criados, o acesso à página não funcionava como esperado. Isso exigiu uma investigação mais detalhada, analisando o Service, as portas configuradas e a comunicação entre o Service e o container da aplicação.

Após revisar as configurações, foi identificado que o targetPort do Service estava configurado para a porta 8080, enquanto o container do Nginx escutava na porta 80. Como resultado, o tráfego chegava ao Service, mas não era encaminhado corretamente para a aplicação.

A correção foi alterar o targetPort para 80, realizar commit, push e sincronizar novamente a Application. Após a alteração, o acesso à página passou a funcionar normalmente.

Esse foi o erro mais desafiador porque exigiu ir além da análise dos recursos do Kubernetes e validar o comportamento real da aplicação, reforçando a importância de não confiar apenas no status do Argo CD, mas também testar efetivamente o serviço entregue ao usuário.

- O `git log --oneline` com a sua sequência de commits.

<img width="857" height="495" alt="image" src="https://github.com/user-attachments/assets/c9b9565a-7674-4876-8cfb-874f7e2ba041" />


---

## Critérios de avaliação

| Item | Pontos | O que conta |
|------|--------|-------------|
| A — Ambiente hml | 3 | Overlay correto (1), Application correta (1), criada pela root sem kubectl apply (1) |
| B — Diagnóstico | 4 | 1 ponto por problema corrigido via Git + 1 pela tabela preenchida com evidências |
| C — Operação | 2 | Promoção em ordem com commits separados (1), self-heal demonstrado e explicado (1) |
| D — Apresentação | 1 | Explica sintoma → evidência → correção com clareza |
| **Total** | **10** | Aprovação a partir de 7,0 pontos |

Para concluir o curso: **nota mínima 7,0** neste desafio **e presença mínima de 75%** da carga horária (pelo menos 3 dos 4 encontros).
