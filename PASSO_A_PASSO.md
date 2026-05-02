# Passo a Passo — Implementação e Atualização da Dashboard Turma B

---

## PARTE 1 — Primeira publicação no GitHub Pages

### Passo 1 — Criar o repositório no GitHub

1. Acesse https://github.com e faça login
2. Clique em **"New repository"** (botão verde no canto superior direito)
3. Preencha:
   - **Repository name**: `dash_B_maximo` (ou o nome que já está usando)
   - **Visibility**: Private (recomendado para dados internos)
4. Clique em **"Create repository"**

---

### Passo 2 — Fazer upload dos arquivos

Existem duas formas. Escolha a mais simples para você:

#### Opção A — Pelo navegador (sem precisar instalar nada)

1. Acesse o repositório recém-criado no GitHub
2. Clique em **"uploading an existing file"** ou arraste os arquivos
3. Arraste os seguintes arquivos de uma vez:
   - `index.html`
   - `logo_OPE.png`
   - `logo_MNT.png`
   - `logo_report.png`
4. Na caixa **"Commit changes"**, escreva: `Atualização v2.1`
5. Clique em **"Commit changes"**

#### Opção B — Pelo terminal (Git)

```bash
# Clone o repositório (substitua pelo URL do seu)
git clone https://github.com/oempe3/dash_B_maximo.git
cd dash_B_maximo

# Copie os 4 arquivos para dentro desta pasta
# (index.html, logo_OPE.png, logo_MNT.png, logo_report.png)

# Adicione, commite e envie
git add .
git commit -m "Atualização v2.1 — Painel HH Apontado x Programado"
git push origin main
```

---

### Passo 3 — Ativar o GitHub Pages

1. No repositório, clique em **"Settings"** (aba no topo)
2. No menu lateral esquerdo, clique em **"Pages"**
3. Em **"Source"**, selecione:
   - Branch: `main`
   - Pasta: `/ (root)`
4. Clique em **"Save"**
5. Aguarde ~2 minutos e acesse a URL exibida:
   `https://oempe3.github.io/dash_B_maximo/`

---

## PARTE 2 — Atualizar os dados mensalmente

### Como funciona a atualização

A dashboard carrega os dados embutidos no `index.html` (bloco JSON interno). Para adicionar um novo mês, você precisa atualizar esse JSON.

---

### Passo 4 — Baixar os arquivos do Google Drive

1. Acesse a pasta do Drive:
   https://drive.google.com/drive/u/1/folders/1ckqvL24ShJQxY8ZK1Aiw3BDkiKPrrXNE

2. Baixe o arquivo do mês mais recente (formato `.xls` ou planilha)

3. Abra no Excel e verifique as colunas esperadas:
   - Data do lançamento
   - Operador
   - Tipo de atividade (OPER, CORR, EM, PRED, PREV, SERV, MODI)
   - Horas lançadas
   - Número da OS

---

### Passo 5 — Converter a planilha para JSON

Use o script `convert.py` que já está no repositório:

```bash
# Pré-requisito: Python 3.8+ instalado
pip install pandas openpyxl

# Executar o conversor
python convert.py arquivo_do_mes.xls
```

Isso vai gerar um arquivo `2026-05.json` (ou o mês correspondente).

**Estrutura esperada do JSON gerado:**

```json
{
  "FRANCISCO.SANTOS": {
    "horas_oper": 120.0,
    "horas_manut": 45.0,
    "horas_total": 165.0,
    "tipos": {"OPER": 120, "CORR": 5, "EM": 0, "PRED": 0, "PREV": 40, "SERV": 0, "MODI": 0},
    "por_dia": [
      {"dia": "02", "OPER": 7.0, "CORR": 0.0, ..., "oper_total": 7.0, "manut_total": 0.0}
    ],
    "os_por_tipo": {"OPER": 1, "CORR": 0, ...},
    "total_os": 8,
    "dias_trabalhados": 18
  },
  "_ALL": { ... },
  "_meta": {"dias_uteis_calendario": 22, "dias_corridos": 31}
}
```

---

### Passo 6 — Inserir o novo mês no index.html

1. Abra o `index.html` em um editor de texto (Notepad++, VS Code, etc.)

2. Localize o bloco JSON. Procure por (Ctrl+F):
   ```
   <script id="data-blob" type="application/json">
   ```

3. Dentro do JSON, encontre a seção `"meses"` e adicione o novo mês:
   ```json
   "meses": ["2026-01", "2026-02", "2026-03", "2026-04", "2026-05"],
   ```

4. Na seção `"dados"`, adicione o novo mês **antes** de `"ANUAL"`:
   ```json
   "2026-05": { ... conteúdo do JSON gerado ... },
   "ANUAL": { ... }
   ```

5. Atualize também a seção `"ANUAL"` para incluir os dados acumulados do novo mês.

6. Salve o arquivo.

---

### Passo 7 — Publicar a atualização

#### Pelo navegador (GitHub.com):
1. Abra o repositório no GitHub
2. Clique no arquivo `index.html`
3. Clique no ícone de lápis (editar)
4. Cole o conteúdo atualizado
5. Clique em **"Commit changes"**

#### Pelo terminal:
```bash
cd dash_B_maximo
git add index.html
git commit -m "Dados de maio/2026 adicionados"
git push origin main
```

A dashboard estará atualizada em ~2 minutos.

---

## PARTE 3 — Entender os cálculos do painel HH × HH

### Regra de negócio aplicada

| Conceito | Valor | Explicação |
|----------|-------|-----------|
| HH Programado por dia | 7h | Cada dia com qualquer lançamento conta como 7h planejadas |
| HH Programado total | dias × 7h | Total de horas de manutenção programadas no período |
| HH Apontado | horas_total | Soma de TODAS as horas lançadas (OPER + MNT) |
| Aderência HH | (Apontado ÷ Programado) × 100% | % de cumprimento da carga de trabalho |

### Exemplo — Abril/2026 (_ALL)

```
Dias com lançamento: 18
HH Programado:  18 × 7h = 126h
HH Apontado:    446h (223h OPER + 223h MNT)
Aderência:      446 ÷ 126 × 100 = 354%
```

> **Nota**: Aderência acima de 100% significa que foram registradas mais horas do que o planejado mínimo. Isso é comum quando operadores acumulam horas extras ou trabalham em finais de semana.

### Semáforo de cores

| % Aderência | Cor | Situação |
|-------------|-----|---------|
| ≥ 100% | 🟢 Verde | Meta atingida |
| 80–99% | 🟡 Amarelo | Abaixo do previsto |
| < 80% | 🔴 Vermelho | Crítico |

---

## PARTE 4 — Solução de problemas comuns

### Dashboard mostra "Sem dados"
- Verifique se o arquivo `index.html` foi salvo corretamente
- Abra o `index.html` localmente no navegador (duplo clique) para testar antes de publicar
- Pressione **F12 → Console** para ver erros de JavaScript

### Botão "Atualizar" retorna erro
- O botão busca um arquivo `data/index.json` no repositório
- Se esse arquivo não existir, ele cai no fallback com os dados embutidos
- Isso é **normal** — os dados embutidos continuam funcionando

### Imagens dos logos não aparecem
- Confirme que os 3 arquivos PNG estão na **mesma pasta** que o `index.html` no repositório
- Os nomes devem ser exatamente: `logo_OPE.png`, `logo_MNT.png`, `logo_report.png`
- Aguarde 5 minutos após o commit para o GitHub Pages processar

### O gráfico de barras não aparece
- Verifique se o mês selecionado tem dados
- Tente mudar de mês pelas abas na parte superior
- Limpe o filtro de datas (botão "Limpar") se estiver ativo

---

## PARTE 5 — Contatos e suporte

- **Repositório GitHub**: https://github.com/oempe3/dash_B_maximo
- **Dashboard ao vivo**: https://oempe3.github.io/dash_B_maximo/
- **Pasta de dados (Drive)**: https://drive.google.com/drive/u/1/folders/1ckqvL24ShJQxY8ZK1Aiw3BDkiKPrrXNE

---

*Documento gerado em Maio/2026 — Dashboard Turma B v2.1*
