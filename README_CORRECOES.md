# JPS Otimizador de Corte — correções aplicadas (v1.1.0)

## 1. PDF não exportava no APK

**Causa:** o código usava `doc.save('plano_corte.pdf')` do jsPDF. Por baixo, esse
método cria um `Blob`, gera uma URL `blob:` e simula o clique em um
`<a download>`. Isso funciona no Chrome do PC, mas a **WebView do Android não
implementa gerenciador de downloads** — o atributo `download` é ignorado e a
navegação para `blob:` é bloqueada. O clique simplesmente não fazia nada, sem
mensagem de erro.

O mesmo problema atingia a **exportação de CSV** (chapas e peças), que usava
exatamente a mesma técnica.

**Correção:** foi criada a função `salvarArquivo()`, que detecta o ambiente:

- **No APK:** grava o arquivo com o plugin `@capacitor/filesystem` na pasta
  Documentos do app e abre a folha de compartilhamento do Android
  (`@capacitor/share`), onde você escolhe salvar no Drive, enviar por WhatsApp,
  abrir no leitor de PDF, etc.
- **No navegador:** mantém o download tradicional.

`exportarPDF()` agora é `async`, valida se o jsPDF carregou e mostra a mensagem
de erro real em caso de falha (antes, qualquer exceção morria silenciosamente).

## 2. jsPDF baixado no momento do build (risco de PDF quebrado)

**Causa:** o workflow baixava o jsPDF do CDN com
`curl -L -o www/assets/js/jspdf.umd.min.js ...`. Sem a flag `-f`, o `curl`
retorna sucesso mesmo em erro HTTP e grava a **página de erro** dentro do
arquivo `.js`. Nesse caso `window.jspdf` fica `undefined` e o
`const { jsPDF } = window.jspdf` lança `TypeError` — botão inerte de novo.

**Correção:** o `jspdf.umd.min.js` (v2.5.2) agora vem **embutido no repositório**
em `www/assets/js/`. O app não depende mais de rede nem no build nem em uso.
O workflow ganhou uma etapa que falha o build se o arquivo estiver ausente ou
corrompido.

## 3. Ícone não era aplicado

**Causa:** o workflow roda `npx @capacitor/assets generate --android`, e essa
ferramenta procura as imagens de origem na pasta **`assets/`** da raiz do
projeto, com nomes **em minúsculas**. O arquivo estava em `resources/Icon.png`
— pasta errada e com letra maiúscula (o runner Linux diferencia maiúsculas de
minúsculas). Nenhuma imagem era encontrada e o Android ficava com o ícone padrão
do Capacitor.

**Correção:** criada a pasta `assets/` com os arquivos nos nomes esperados:

| Arquivo | Uso |
|---|---|
| `icon.png` (1024×1024) | ícone legado |
| `icon-foreground.png` | camada frontal do ícone adaptativo |
| `icon-background.png` | fundo sólido `#092541`, extraído da sua arte |
| `splash.png` / `splash-dark.png` (2732×2732) | tela de abertura |

No `icon-foreground.png` a arte foi reduzida a 62% e centralizada. Isso é
proposital: o Android recorta o ícone adaptativo em círculo/squircle conforme o
launcher, e na arte original os textos das bordas ("MELHOR APROVEITAMENTO",
"CORTE INTELIGENTE") seriam cortados. Com a margem de segurança, o logo JPS fica
inteiro em qualquer formato de ícone.

## 4. APK antigo dentro da pasta `www/`

A pasta `www/` continha `OtimizadorCorte-debug.apk.zip` e
`OtimizadorCorte-debug.apk/app-debug.apk` (≈7 MB). Tudo que está em `www/` é
copiado para dentro dos assets do aplicativo — ou seja, o APK antigo estava
sendo embutido dentro do APK novo. Arquivos removidos e `*.apk` / `*.zip`
adicionados ao `.gitignore`.

## 5. Ajustes menores

- `appId` mudou de `br.com.meunome.otimizadorcorte` para
  `br.com.jps.otimizadorcorte`. **Atenção:** mudar o `appId` faz o Android tratar
  como um app diferente — o novo APK instala ao lado do antigo em vez de
  atualizá-lo. Se preferir atualizar por cima, reverta o `appId` no
  `capacitor.config.json`.
- `appName` para "JPS Otimizador de Corte".
- Removido `bundledWebRuntime` (obsoleto no Capacitor 6).
- Workflow agora também dispara na branch `master` e usa `--stacktrace`.
- CSV passou a sair com BOM UTF-8, para o Excel não quebrar os acentos.

---

## Como gerar o novo APK

1. Suba o conteúdo desta pasta para o repositório no GitHub (substituindo o
   anterior):

   ```bash
   git add -A
   git commit -m "Corrige exportacao de PDF/CSV e icone do app"
   git push
   ```

2. Aba **Actions** → workflow **Build APK Android** (roda sozinho no push, ou
   clique em *Run workflow*).
3. Ao terminar, abra a execução → seção **Artifacts** → baixe
   **`OtimizadorCorte-debug-apk`**.
4. Instale o `.apk` no celular (permitindo "fontes desconhecidas").

## Como testar as correções

1. Cadastre chapas e peças, toque em **Otimizar**.
2. Toque em **Exportar PDF** → deve abrir a folha de compartilhamento do Android.
   Escolha "Salvar em Arquivos" ou envie para si mesmo e abra o PDF.
3. Faça o mesmo com **Exportar CSV**.
4. Confira o ícone na gaveta de aplicativos e a tela de abertura.

Se algo ainda falhar, agora o app mostra a mensagem de erro real em um alerta —
me mande o texto exato que eu identifico o ponto.
