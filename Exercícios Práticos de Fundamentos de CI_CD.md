# Exercícios Práticos de Fundamentos de CI/CD

Este repositório contém os arquivos para os exercícios práticos de Fundamentos de CI/CD.

## Exercício 1: Configuração de um Repositório com CI Básica

### Passos para Configuração:

1.  **Crie um repositório no GitHub:**
    *   Crie um novo repositório no GitHub com o nome `ci-cd-exercicio`.
    *   Clone este repositório para sua máquina local:
        ```bash
        git clone https://github.com/SEU_USUARIO/ci-cd-exercicio.git
        cd ci-cd-exercicio
        ```

2.  **Adicione os arquivos do projeto:**
    *   Copie os arquivos `app.py`, `test_app.py` e a pasta `.github` (com o arquivo `ci.yml` dentro) para o diretório raiz do seu repositório local.

3.  **Faça commit e push:**
    *   Adicione os arquivos ao Git:
        ```bash
        git add .
        ```
    *   Faça commit:
        ```bash
        git commit -m "Exercício 1: Configuração inicial de CI"
        ```
    *   Faça push para o branch `main`:
        ```bash
        git push origin main
        ```

4.  **Verifique o pipeline no GitHub:**
    *   Acesse a aba "Actions" no seu repositório no GitHub e verifique se o pipeline `CI Pipeline` foi executado com sucesso.

### Desafio Adicional (Exercício 1):

1.  **Modifique a função `soma` para conter um erro:**
    *   Edite o arquivo `app.py` e altere a função `soma` para algo como `return a + b + 1`.
    *   Faça commit e push para o branch `main`.
    *   Observe o pipeline falhar na aba "Actions" do GitHub.

2.  **Corrija o erro:**
    *   Corrija a função `soma` de volta para `return a + b`.
    *   Faça commit e push novamente.
    *   Confirme que o pipeline volta a passar.




## Exercício 2: Integração de Linter no Pipeline

### Passos para Configuração:

1.  **Adicione um linter ao projeto:**
    *   Instale o `flake8` localmente (opcional, para testes antes do push):
        ```bash
        pip install flake8
        flake8 app.py
        ```
    *   Corrija quaisquer erros de estilo apontados pelo linter no `app.py`.

2.  **Atualize o pipeline de CI:**
    *   Modifique o arquivo `.github/workflows/ci.yml` para incluir um passo que execute o `flake8`. O conteúdo atualizado do `ci.yml` deve ser:

```yaml
name: CI Pipeline
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.9'
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install pytest flake8
      - name: Run linter
        run: flake8 app.py
      - name: Run tests
        run: pytest test_app.py
```

3.  **Teste o pipeline:**
    *   Adicione um erro de estilo no `app.py` (ex.: remova um espaço após uma vírgula em `def soma(a,b):`).
    *   Faça commit e push para o branch `main`.
    *   Observe o pipeline falhar no passo do linter na aba "Actions" do GitHub.

4.  **Corrija o erro:**
    *   Corrija o erro de estilo no `app.py`.
    *   Faça commit e push novamente.
    *   Confirme que o pipeline passa.




## Exercício 3: Configuração de Deploy Simples (CD)

### Passos para Configuração:

1.  **Crie um script para empacotar o projeto:**
    *   Crie um arquivo chamado `build.sh` com o seguinte conteúdo:

```bash
#!/bin/bash
zip -r projeto.zip app.py test_app.py
```

2.  **Atualize o pipeline para incluir o deploy:**
    *   Modifique o arquivo `.github/workflows/ci.yml` para criar um release com o artefato. O conteúdo atualizado do `ci.yml` deve ser:

```yaml
name: CI/CD Pipeline
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
jobs:
  test-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.9'
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install pytest flake8
      - name: Run linter
        run: flake8 app.py
      - name: Run tests
        run: pytest test_app.py
      - name: Build artifact
        run: |
          chmod +x build.sh
          ./build.sh
      - name: Create Release
        if: github.event_name == 'push' && github.ref == 'refs/heads/main'
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: v${{ github.run_number }}
          release_name: Release v${{ github.run_number }}
          draft: false
          prerelease: false
      - name: Upload Release Asset
        if: github.event_name == 'push' && github.ref == 'refs/heads/main'
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create-release.outputs.upload_url }}
          asset_path: ./projeto.zip
          asset_name: projeto.zip
          asset_content_type: application/zip
```

3.  **Teste o pipeline:**
    *   Faça commit e push do arquivo `build.sh` e do `ci.yml` atualizado para o branch `main`.
    *   Verifique na aba "Releases" do seu repositório no GitHub se o arquivo `projeto.zip` foi anexado a um release.




## Exercício 4: Integração de Pull Requests

### Passos para Configuração:

1.  **Crie um branch e um pull request:**
    *   Crie um novo branch chamado `feature/nova-funcao`:
        ```bash
        git checkout -b feature/nova-funcao
        ```
    *   Adicione uma nova função ao `app.py` (ex.: `def multiplica(a, b): return a * b`).
    *   Adicione testes correspondentes no `test_app.py`.

2.  **Teste o pipeline:**
    *   Faça push do branch para o GitHub:
        ```bash
        git add .
        git commit -m "feat: adiciona funcao multiplica"
        git push origin feature/nova-funcao
        ```
    *   Crie um pull request para o branch `main` no GitHub.
    *   Verifique na aba "Actions" se o pipeline foi executado no pull request.
    *   Adicione um erro intencional no código (ex.: um teste que falha) e observe o pipeline falhar.
    *   Corrija o erro e confirme que o pipeline passa.

3.  **Mescle o pull request:**
    *   Após o pipeline passar, mescle o pull request para o branch `main`.
    *   Verifique se o pipeline de deploy é executado e cria um novo release na aba "Releases".



