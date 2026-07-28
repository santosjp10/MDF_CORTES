# Otimizador de Corte de Chapas - App Android

Aplicativo de planejamento e otimização de cortes em chapas planas (MDF, MDP, compensado, etc.). Gera automaticamente o melhor plano de aproveitamento, reduzindo o desperdício.

## Funcionalidades
- Cadastro de chapas e peças com dimensões, espessura e orientação de corte.
- Algoritmo MaxRects com múltiplas tentativas e ordenações inteligentes.
- Visualização gráfica do plano de corte.
- Exportação de CSV e PDF (formato retrato, proporcional).
- Salvar e carregar projetos no próprio dispositivo.
- Botão "Reotimizar" para buscar soluções ainda melhores.

## Como gerar o APK (sem Android Studio)
Todo o processo de build é feito automaticamente pelo GitHub Actions. Você só precisa enviar este projeto para um repositório no GitHub.

1. **Crie um repositório** no GitHub (público ou privado).
2. **Envie os arquivos** deste projeto para a branch `main`:
   ```bash
   git init
   git add .
   git commit -m "Primeira versão"
   git remote add origin https://github.com/seu-usuario/seu-repo.git
   git push -u origin main