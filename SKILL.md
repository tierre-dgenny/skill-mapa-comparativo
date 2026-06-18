---
name: mapa-comparativo
description: Cria e publica automaticamente um Mapa Comparativo dgenny® no GitHub Pages (tierre-dgenny.github.io) para nutrir um lead com argumentos de implicação. TRIGGER quando usuário pedir "mapa comparativo", "mapa para o lead", "mapa de implicação" ou similar.
---

# Skill: Mapa Comparativo dgenny®

## O que esta skill faz

Gera um deck HTML de slides no padrão visual dgenny® (dark theme, Montserrat + Quicksand, paleta azul/roxo/rosa) com conteúdo personalizado para um lead específico — e publica automaticamente no GitHub Pages, entregando uma URL pública pronta para enviar ao cliente.

**Nunca entrega apenas o arquivo `.html` local. Sempre publica e entrega a URL.**

## Template de referência

- **Repositório-modelo:** `https://tierre-dgenny.github.io/dgenny-mapa-newen/`
- Buscar o HTML completo via `WebFetch` para reutilizar exatamente o CSS, componentes e JavaScript
- Adaptar o conteúdo para o lead específico — nunca alterar o padrão visual

## Credenciais GitHub

- **Token:** recuperar via `git credential fill` (host: `github.com`) — já está salvo no gerenciador de credenciais do Windows
- **Conta pessoal:** `tierre-dgenny`
- **Padrão do repositório:** `dgenny-mapa-<slug-do-lead>` (ex: `dgenny-mapa-jrc`, `dgenny-mapa-newen`)
- **URL publicada:**
  - `https://tierre-dgenny.github.io/dgenny-mapa-<slug>/`

## Argumentos esperados

O usuário pode passar:
- **Dados da análise do lead** (copiados do CRM/EspoCRM) — dores, objeções, SPIN, MEDDPICC, próximos passos
- **Nome da empresa/lead** — usado para o slug do repositório e título dos slides
- **Informações adicionais** — contexto de reunião, urgência, decisor, ERP usado, volume de compras

Se faltar alguma informação crítica para um slide, inferir com base no que foi fornecido ou omitir o slide correspondente.

## Estrutura padrão dos slides (adaptar conforme o lead)

1. **Capa** — nome da empresa, mês/ano, tagline "O que muda com a decisão certa"
2. **Diagnóstico** — como o processo funciona hoje, onde está o gargalo
3. **A dor central** — nomear a dor principal do lead com dados/citações reais da reunião
4. **Por que o problema persiste** — argumento de implicação: o que impede a solução atual de funcionar
5. **Sem dgenny × Com dgenny (processo)** — tabela comparativa do fluxo de cotação
6. **Sem dgenny × Com dgenny (engajamento)** — checklists antes/depois na dor específica
7. **A alavanca estratégica** — o trabalho que nunca acontece por causa do operacional (ex: homologação)
8. **Transformação operacional** — tabela com métricas antes/depois
9. **Para o decisor** — slide direcionado ao Economic Buyer com ângulo de IA, crescimento ou ROI
10. **Custo de oportunidade** — o que fica na mesa todo mês sem agir
11. **Próximos Passos** — **destaque visual obrigatório** — cards com `next-step-card` e datas reais

## Argumentos de implicação obrigatórios

Sempre usar argumentos que mostrem o CUSTO de não agir:
- Quanto tempo é perdido por ciclo manual
- O que NUNCA é feito por falta de tempo (homologação, análise estratégica, etc.)
- O ciclo vicioso que se retroalimenta (menos fornecedores → menos barganha → preços piores)
- Que trocar de ERP não resolve o problema se o gargalo é comportamental (fornecedores no WhatsApp)
- Que o problema cresce com o crescimento da empresa (mais volume = mais pressão no mesmo gargalo)

## Fluxo de execução

### 1. Buscar o template
```
WebFetch: https://tierre-dgenny.github.io/dgenny-mapa-newen/
Prompt: "Me dê o HTML completo desta página incluindo todo o CSS inline e JavaScript"
```

### 2. Gerar o HTML adaptado
- Substituir todo o conteúdo pelo contexto do lead
- Manter exatamente o mesmo CSS, classes, paleta e JavaScript de navegação
- Nome da empresa no título, tag e rodapé
- Citações reais da reunião nos cards de dor
- Próximos passos com datas reais se disponíveis
- Slide 11 (Próximos Passos) com cards `.next-step-card` em destaque

**Tamanhos de fonte obrigatórios para o texto miúdo (body text):**
- `.clabel`: 12px — `.ctitle`: 16px — `.cdesc`: 14px
- `.dtable` (body): 15px — `.dtable th`: 12px — `.dtable .sub-head td`: 11px
- `.chk`: 14px — `.mrow .ml/.mb/.ma`: 14px — `.roi-lbl`: 13px
- `.f7step .fname`: 12px — `.f7step .fdesc`: 11px — `.f7step .ftag`: 10px
- `.nsc .nt`: 17px — `.nsc .nd`: 14px — `.nsc .na`: 13px

### 3. Recuperar o token GitHub
```bash
git credential fill <<'EOF'
protocol=https
host=github.com
EOF
```
Extrair o valor do campo `password` — esse é o token.

### 4. Criar o repositório via API (conta pessoal)
```bash
curl -s -X POST \
  -H "Authorization: token TOKEN" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/user/repos \
  -d '{"name":"dgenny-mapa-SLUG","description":"Mapa Comparativo dgenny® · EMPRESA","private":false,"auto_init":false}'
```
Verificar que retornou HTTP 201.

### 5. Fazer push via git

> ⚠️ **Windows:** o Write tool escreve em `C:\tmp\` mas o bash vê `/tmp` como `C:\Users\tierr\AppData\Local\Temp`. Usar Python para as operações git — é o único runtime que acessa os dois caminhos corretamente.

```python
# Usar sempre via Bash tool com bloco python3 << 'PYEOF' ... PYEOF
import os, subprocess

repo_dir = r'C:\tmp\mapa-SLUG'
token = 'TOKEN'

def run(args, cwd=None):
    r = subprocess.run(args, cwd=cwd, capture_output=True, text=True)
    if r.stdout.strip(): print(f'OUT: {r.stdout.strip()}')
    if r.returncode != 0 and r.stderr.strip(): print(f'ERR: {r.stderr.strip()}')
    return r.returncode == 0

run(['git', 'init'], cwd=repo_dir)
run(['git', 'checkout', '-b', 'main'], cwd=repo_dir)
run(['git', 'config', 'user.email', 'tierre@dgenny.com.br'], cwd=repo_dir)
run(['git', 'config', 'user.name', 'tierre-dgenny'], cwd=repo_dir)
run(['git', 'add', 'index.html'], cwd=repo_dir)
run(['git', 'commit', '-m', 'feat: mapa comparativo EMPRESA'], cwd=repo_dir)
run(['git', 'remote', 'add', 'origin',
     f'https://tierre-dgenny:{token}@github.com/tierre-dgenny/dgenny-mapa-SLUG.git'], cwd=repo_dir)
run(['git', 'push', '-u', 'origin', 'main'], cwd=repo_dir)
```

### 6. Ativar GitHub Pages
```bash
curl -s -X POST \
  -H "Authorization: token TOKEN" \
  -H "Accept: application/vnd.github+json" \
  -H "Content-Type: application/json" \
  https://api.github.com/repos/tierre-dgenny/dgenny-mapa-SLUG/pages \
  --data-raw '{"source":{"branch":"main","path":"/"}}'
```
Verificar que retornou HTTP 201.

### 7. Gerar a mensagem de envio (.txt)

Após publicar o mapa, gerar automaticamente uma mensagem de WhatsApp/texto pronta para a Tierre encaminhar ao lead — e salvar como arquivo `.txt` no Desktop.

**Regras da mensagem:**
- Tom pessoal, direto, sem formalidade excessiva — estilo WhatsApp profissional
- Abrir com o primeiro nome do contato principal (ex: "Ricardo, boa tarde!")
- Referenciar algo real dito na reunião (citação ou dor específica mencionada)
- Incluir o link do mapa comparativo
- Destacar 1 slide específico para o decisor (número + ângulo: ROI, crescimento, risco)
- CTA claro: encaminhar o link pro decisor + propor data para demo ao vivo
- Fechar com oferta de ajuste rápido se quiser personalizar antes de enviar

**Salvar o arquivo:**
```python
import os

slug = 'SLUG'
nome_contato = 'NOME'
url_mapa = f'https://tierre-dgenny.github.io/dgenny-mapa-{slug}/'

mensagem = f"""[NOME], boa tarde!

Conforme combinamos, preparei o Mapa Comparativo da dgenny® com o cenário real da [EMPRESA]:

👉 {url_mapa}

[PARÁGRAFO: referência à dor central mencionada na reunião + o que o mapa mostra sobre ela]

Tem um slide específico pro [NOME DO DECISOR] (slide X) com o ângulo de [ÂNGULO PRINCIPAL: ROI / crescimento / risco operacional]. Acho que facilita bastante a conversa com ele.

Próximo passo sugerido: você encaminha esse link pro [NOME DO DECISOR] antes da reunião e a gente agenda a demonstração ao vivo — com [REFERÊNCIA AO ERP/FLUXO REAL], cotação rodando no WhatsApp e o mapa comparativo gerado na frente de vocês. Assim ele vê funcionando, não só falando.

Me fala como fica melhor pra vocês — semana que vem já dá?"""

desktop = os.path.join(os.path.expanduser('~'), 'Desktop')
filepath = os.path.join(desktop, f'mensagem-{slug}.txt')
with open(filepath, 'w', encoding='utf-8') as f:
    f.write(mensagem)
print(f'Mensagem salva em: {filepath}')
```

Adaptar cada placeholder `[...]` com o conteúdo real do lead antes de salvar.

### 8. Entregar URL + mensagem
Informar à usuária:
```
✅ Publicado em:
• https://tierre-dgenny.github.io/dgenny-mapa-SLUG/
(disponível em 1–2 minutos)

📝 Mensagem de envio salva em:
• Desktop/mensagem-SLUG.txt
```

## Regras importantes

- **Nunca usar "Jane"** — o agente se chama dgenny®
- **Nunca usar "Digine"** — a solução se chama Dgenny
- O slide de Próximos Passos deve ter **destaque visual diferenciado** (cards com borda colorida, datas, setas →)
- Se o lead tiver um ERP com portal de cotação, sempre incluir o argumento "fornecedores não usam portals, vivem no WhatsApp — a dgenny® resolve isso"
- Se houver CEO/decisor identificado que gosta de IA, criar slide específico para ele
- Sempre usar citações reais da reunião (entre aspas) nos cards de dor — dá autenticidade
- O slug do repositório deve ser em minúsculas, sem espaços, sem caracteres especiais (ex: `grupo-jrc` → `jrc`)
