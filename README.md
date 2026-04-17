# Gerador de Layout Posicional

Ferramenta web client-side para geração de arquivos de carga com campos em posições fixas (layout posicional) e comandos cURL de envio de transações, a partir de planilhas Excel.

Não requer instalação, servidor ou dependências — basta abrir o `index.html` em qualquer navegador moderno.

---

## Funcionalidades

- **Layout posicional (.txt):** gera arquivo de carga com cada campo ocupando exatamente o número de posições definido, com alinhamento configurável por campo.
- **Comandos cURL (.sh / .ps1 / .bat):** gera scripts prontos para envio das transações via HTTP, com autenticação automática por token.
- **Suporte a 3 idiomas:** Português, Inglês e Espanhol, alternáveis em tempo real.
- **Guia de uso integrado:** passo a passo acessível diretamente na página.

---

## Como usar

### Pré-requisitos

Nenhum. Abra o arquivo `index.html` diretamente no navegador.

---

### Passo 1 — Planilha de Definição de Campos

Carregue o arquivo Excel de definição (ex.: `evento_event_transfin_vD.xlsx`).

**Requisitos da planilha:**
- A aba deve se chamar **`Campos Entrada`** (caso contrário, a primeira aba é usada).
- Os **cabeçalhos** devem estar na **linha 2**; os dados começam na **linha 3**.
- Colunas obrigatórias (detecção automática por nome, ordem não importa):

| Coluna | Descrição |
|---|---|
| `Entrada` | `S` = campo incluído no layout |
| `NomeCampo` | Nome do campo (deve coincidir com o cabeçalho da planilha de dados) |
| `TamanhoCampo` | Número de posições que o campo ocupa |
| `AlinhamentoCampo` | `D` = direita, qualquer outro valor = esquerda |
| `TipoCampo` | Tipo informativo (texto, numérico etc.) |
| `CampoPresenteTransacao` | `S` = campo presente na transação cURL |

Somente campos com **`Entrada = S`** e **`CampoPresenteTransacao = S`** são processados.

---

### Passo 2 — Planilha de Massa de Dados

Carregue a planilha com os registros a processar.

- Cada linha representa um registro.
- Os **cabeçalhos das colunas** devem ser idênticos aos valores de `NomeCampo` da definição.
- Campos ausentes na planilha de dados são preenchidos com espaços no layout posicional.

---

### Passo 3 — Gerar Layout Posicional

Clique em **⚙️ Gerar Layout**. Cada registro é convertido em uma linha de comprimento fixo:

- Valores maiores que o tamanho definido são **truncados**.
- Valores menores são **preenchidos com espaços** (esquerda ou direita conforme `AlinhamentoCampo`).

O log exibe estatísticas e alertas de truncamento. O resultado pode ser baixado como `.txt` ou copiado.

---

### Passo 4 — Gerar Comandos cURL

Configure os campos de autenticação e transação:

| Campo | Descrição |
|---|---|
| URL de Autenticação | Endpoint que retorna o token |
| Usuário / Senha | Credenciais para autenticação |
| Institution Acronym | Ex.: `TUYA` |
| Event Type Code | Ex.: `TUYTRANMON` |
| URL da Transação | Endpoint de envio das transações |

Clique em **🌐 Gerar cURL** para produzir os scripts. Três formatos são gerados simultaneamente:

#### 🐧 `.sh` — Linux, Mac, Git Bash
```bash
bash curl_transacoes_YYYY-MM-DD.sh
```

#### 🪟 `.ps1` — Windows PowerShell
```powershell
powershell -ExecutionPolicy Bypass -File .\curl_transacoes_YYYY-MM-DD.ps1
```

#### 🖱️ `.bat` — Windows (duplo clique)
Basta dar **dois cliques** no arquivo. Abre o PowerShell automaticamente, autentica e envia todos os registros. Nenhuma configuração adicional necessária.

O script autentica automaticamente antes de enviar os registros e exibe o status de cada envio.

---

## Estrutura do payload JSON

```json
{
  "institutionAcronym": "TUYA",
  "eventTypeCode": "TUYTRANMON",
  "map": {
    "acquirerId": "valor",
    "transactionAmount": 100,
    "transactionDate": "2026-04-17T08:00:00Z",
    "recordCreationDate": "<gerado automaticamente>",
    "...": "..."
  }
}
```

- Campos numéricos (ex.: `transactionAmount`, `cashbackAmount`) são convertidos automaticamente para número.
- `recordCreationDate` é preenchido com o timestamp do momento da geração.

---

## Observação sobre envio via browser

O envio direto pelo browser requer que o servidor de destino possua o header `Access-Control-Allow-Origin` configurado (política CORS). Como isso depende do servidor, **o método recomendado de envio é via os scripts gerados** (`.sh`, `.ps1` ou `.bat`), que executam o `curl` fora do browser e não têm essa restrição.

---

## Dependências

| Biblioteca | Versão | Carregamento |
|---|---|---|
| [SheetJS](https://sheetjs.com/) | 0.20.3 | CDN (automático) |

---

## Tecnologias

- HTML5 + CSS3 + JavaScript (vanilla, sem frameworks)
- Compatível com Chrome, Edge, Firefox e Safari modernos
