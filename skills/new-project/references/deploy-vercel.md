# Modelo de deploy: Vercel / Netlify

Depois de copiar `variants/deploy-vercel/*` (Fase 2, Step 3):

1. Acesse vercel.com (ou netlify.com), "Import Project", conecte
   `pmsartori/<slug>`.
2. Configure `main` como branch de produção.
3. Variáveis de ambiente são configuradas no painel da plataforma, nunca
   no repositório.

Não há secrets do GitHub Actions a configurar — o deploy roda pelo app
oficial da plataforma, fora do GitHub Actions.
