# Desafio final no Killercoda (para quem não tem cluster na máquina)

Use este roteiro se o Kubernetes não funciona no seu computador. Tudo roda no navegador.

**Importante:** a sessão gratuita do Killercoda dura **1 hora** e é apagada no fim.
O que vale para a nota fica no **Git** (seus commits). Se a sessão acabar, abra outra,
rode o bloco de preparação de novo e continue de onde parou.

## 1. Abrir o cluster

1. Acesse https://killercoda.com/playgrounds/scenario/kubernetes e entre com a sua conta do GitHub.
2. Aguarde o terminal ficar pronto (cerca de 1 minuto).

## 2. Preparar o ambiente (cerca de 6 minutos)

Você precisa de um **Personal Access Token** do GitHub com permissão *Contents: Read and write*
no seu fork (o mesmo da aula 2; se não tiver, crie em GitHub → Settings → Developer settings).

Troque os dois valores abaixo e cole o bloco inteiro no terminal:

```bash
export GH_USER=seu-usuario-github
export GH_TOKEN=seu-token

git clone https://$GH_USER:$GH_TOKEN@github.com/$GH_USER/gitops-curso.git
cd gitops-curso
git config user.name "$GH_USER"
git config user.email "$GH_USER@users.noreply.github.com"

# Aponta as Applications para o seu fork (se ainda não tiver feito)
grep -rl SEU_USUARIO argocd | xargs -r sed -i "s/SEU_USUARIO/$GH_USER/g"
git add argocd && git commit -m "Aponta Applications para o meu fork" && git push || true

# Instala o Argo CD
kubectl create namespace argocd
kubectl apply -n argocd --server-side --force-conflicts \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/v3.5.3/manifests/install.yaml
kubectl -n argocd patch configmap argocd-cmd-params-cm --type merge -p '{"data":{"server.insecure":"true"}}'
kubectl -n argocd patch configmap argocd-cm --type merge -p '{"data":{"timeout.reconciliation":"60s"}}'
kubectl -n argocd rollout restart deployment argocd-server argocd-repo-server
kubectl -n argocd rollout restart statefulset argocd-application-controller
kubectl -n argocd rollout status deployment argocd-server --timeout=300s

# Cria a root (App of Apps) com os ambientes dev e prod
kubectl apply -f argocd/root.yaml

# Senha do admin
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

Guarde a senha que aparece no final.

## 3. Abrir a interface do Argo CD

```bash
kubectl port-forward --address 0.0.0.0 svc/argocd-server -n argocd 8080:80 > /dev/null 2>&1 &
```

No Killercoda, abra o menu (canto superior direito) → **Traffic / Ports** → digite **8080** → **Access**.
Entre com o usuário `admin` e a senha.

## 4. Fazer o desafio

Siga o `atividades/desafio-final.md` normalmente, com duas diferenças:

- Para editar arquivos, use o editor do Killercoda (aba **Editor**) ou `nano <arquivo>`.
- Para testar as páginas, use `curl` no próprio terminal:

```bash
kubectl port-forward svc/web -n web-hml 8084:80 > /dev/null 2>&1 &
curl -s localhost:8084 | grep Ambiente
```

## 5. Antes de a sessão acabar

Faça `git push` de tudo e tire um print de:

```bash
kubectl get applications -n argocd
git log --oneline
```

Esses prints e o seu fork no GitHub são a evidência para a avaliação.
