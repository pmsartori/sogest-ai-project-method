# Modelo de deploy: SSH próprio

Depois de copiar `variants/deploy-ssh/*` (Fase 2, Step 3), configure no
repositório novo (Settings → Secrets and variables → Actions):

- `SSH_HOST` — IP ou domínio do servidor
- `SSH_USER` — usuário SSH
- `SSH_KEY` — chave privada (nunca peça isso em texto no chat; instrua o
  usuário a colar direto no campo do GitHub)
- `DEPLOY_PATH` — caminho base no servidor (os workflows assumem
  subpastas `dev/`, `test/`, `prod/` dentro dele)

Sem esses secrets configurados, `deploy-dev.yml` e `deploy-prod.yml`
falham no primeiro push — isso é esperado até a configuração manual ser
feita.
