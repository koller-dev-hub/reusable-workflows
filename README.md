# Repositório de templates de GitHub Actions workflows para projetos da Koller Dev Hub.

## 📋 Templates Disponíveis

### 1. Pull Request Auto-Create

**Arquivo:** `templates/1-pull-request-to-create.yml`

Cria automaticamente uma Pull Request após o sucesso do workflow de CI.

**Funcionalidades:**

- Cria PR automaticamente para branches `feature/*`, `bugfix/*` e `hotfix/*`
- Marca a PR como "ready for review"
- Adiciona comentário de confirmação
- Bloqueia execução para branches `main` e `master`

**Secrets necessários:**

- `BOT_TOKEN` - Token do GitHub com permissões para criar PRs

---

### 2. Go CI & Coverage

**Arquivo:** `templates/2-build-go-ci-coverage.yml`

Workflow de CI para projetos Go com testes e cobertura de código.

**Funcionalidades:**

- Executa testes com cobertura
- Faz upload da cobertura para Codecov
- Suporta Go 1.25

**Secrets necessários:**

- `CODECOV_TOKEN` - Token do Codecov

**Configurações personalizáveis:**

- Versão do Go (linha 24)
- Pacotes excluídos da cobertura (linha 32)
- Slug do repositório Codecov (linha 51)

---

### 3. React Native CI & Coverage

**Arquivo:** `templates/3-build-react-ci-coverage.yml`

Workflow de CI para projetos React Native com testes e cobertura de código.

**Funcionalidades:**

- Executa linting (se disponível)
- Executa testes com cobertura
- Faz upload da cobertura para Codecov
- Suporta Node.js 22

**Secrets necessários:**

- `CODECOV_TOKEN` - Token do Codecov

**Configurações personalizáveis:**

- Versão do Node.js (linha 24)

---

## 🚀 Como Usar

### Opção 1: Copiar arquivo diretamente

1. Copie o template desejado para o seu repositório:

```bash
# No seu repositório de destino
mkdir -p .github/workflows
curl -o .github/workflows/ci.yml https://raw.githubusercontent.com/koller-dev-hub/reusable-workflows/main/templates/2-build-go-ci-coverage.yml
```

2. Edite o arquivo copiado conforme necessário
3. Configure os secrets no repositório (Settings → Secrets and variables → Actions)

### Opção 2: Copiar manualmente

1. Acesse a pasta `templates/` neste repositório
2. Copie o conteúdo do template desejado
3. Crie o arquivo `.github/workflows/<nome>.yml` no seu repositório
4. Cole e ajuste conforme necessário
5. Configure os secrets no repositório

### Opção 3: Usar como Reusable Workflow

Em vez de copiar os arquivos, você pode chamar workflows deste repositório diretamente. Isso permite centralizar a manutenção e receber atualizações automaticamente.

**Vantagens:**

- ✅ Sem duplicação de código
- ✅ Atualizações centralizadas
- ✅ Mais fácil de manter

**Como usar:**

1. **Neste repositório (`reusable-workflows`)**, crie workflows reutilizáveis em `.github/workflows/`:

```yaml
# .github/workflows/go-ci-reusable.yml
name: Go CI Reusable

on:
  workflow_call:
    inputs:
      go-version:
        required: false
        type: string
        default: '1.23'
      exclude-packages:
        required: false
        type: string
        default: '/mocks|/tests'
    secrets:
      CODECOV_TOKEN:
        required: true

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version: ${{ inputs.go-version }}

      - name: Run tests with coverage
        run: |
          go test -v -coverprofile=coverage.out $(go list ./... | grep -v '${{ inputs.exclude-packages }}')

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          files: ./coverage.out
```

2. **No seu outro repositório**, chame o workflow reutilizável:

```yaml
# .github/workflows/ci.yml (no repositório de destino)
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  ci:
    uses: koller-dev-hub/reusable-workflows/.github/workflows/go-ci-reusable.yml@main
    with:
      go-version: '1.23'
      exclude-packages: '/mocks|/tests|/cmd'
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

**Nota:** Esta opção requer que os workflows sejam convertidos para o formato `workflow_call`. Atualmente, os templates estão no formato de cópia direta (Opções 1 e 2).

---

## 💡 Exemplo Prático

Vamos configurar o workflow de **Go CI com Coverage** no repositório `app-system-education`:

### Passo 1: Copiar o template

```bash
# Entre no seu repositório
cd /caminho/para/app-system-education

# Crie a pasta de workflows se não existir
mkdir -p .github/workflows

# Copie o template de Go CI
curl -o .github/workflows/ci.yml https://raw.githubusercontent.com/koller-dev-hub/reusable-workflows/main/templates/2-build-go-ci-coverage.yml
```

### Passo 2: Personalizar o workflow

Edite `.github/workflows/ci.yml` e ajuste conforme necessário:

```yaml
# Exemplo de personalizações comuns:

# Alterar versão do Go (linha ~24)
go-version: '1.23'  # Ajuste para sua versão

# Ajustar branches que acionam o workflow (linha ~5)
on:
  push:
    branches: [ main, develop ]  # Adicione suas branches
  pull_request:
    branches: [ main, develop ]

# Excluir pacotes da cobertura (linha ~32)
go test -v -coverprofile=coverage.out $(go list ./... | grep -v '/mocks\|/tests')
```

### Passo 3: Configurar secrets no GitHub

1. Acesse seu repositório no GitHub
2. Vá em **Settings** → **Secrets and variables** → **Actions**
3. Clique em **New repository secret**
4. Adicione:
   - **Nome:** `CODECOV_TOKEN`
   - **Valor:** Token obtido em [codecov.io](https://codecov.io)

### Passo 4: Commit e push

```bash
git add .github/workflows/ci.yml
git commit -m "ci: add Go CI workflow with coverage"
git push origin main
```

### Passo 5: Verificar execução

1. Acesse a aba **Actions** no GitHub
2. Verifique se o workflow foi executado com sucesso
3. Confira a cobertura de código no Codecov

---

## ⚙️ Configurações Comuns

### Codecov Token

Para workflows com cobertura de código:

1. Acesse [codecov.io](https://codecov.io)
2. Adicione seu repositório
3. Copie o token gerado
4. No GitHub: Settings → Secrets and variables → Actions → New repository secret
5. Nome: `CODECOV_TOKEN`
6. Valor: cole o token do Codecov

### Bot Token (para auto-create PR)

Para o workflow de criação automática de PR:

1. Crie um Personal Access Token no GitHub
2. Permissões necessárias: `repo`, `workflow`
3. No GitHub: Settings → Secrets and variables → Actions → New repository secret
4. Nome: `BOT_TOKEN`
5. Valor: cole o token gerado

---

## 📝 Personalizações Recomendadas

Ao copiar um template, revise e ajuste:

- [ ] Versões de linguagens/ferramentas (Go, Node.js, etc.)
- [ ] Branches que acionam o workflow
- [ ] Pacotes/diretórios excluídos dos testes
- [ ] Nome do repositório no Codecov (se aplicável)
- [ ] Scripts de teste no `package.json` (React Native)
- [ ] Comandos de build específicos do projeto

---

## 🤝 Contribuindo

Para adicionar novos templates ou melhorar os existentes:

1. Crie os workflows na pasta `templates/`
2. Documente no README
3. Teste em um repositório real antes de fazer PR
4. Mantenha os templates genéricos e configuráveis

---

## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.
