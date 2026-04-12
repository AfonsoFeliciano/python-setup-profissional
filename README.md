# Setup Profissional de Python no Windows

> Material de apoio da aula — Pós-graduação em Engenharia de Dados · Xperiun

---

A proposta aqui não é só instalar Python e sair rodando código.

É montar um ambiente de trabalho que você vai conseguir reproduzir, que outro colega vai conseguir clonar e rodar, e que não vai te dar dor de cabeça quando você precisar juntar dois projetos na mesma máquina.

No mercado, não adianta falar "na minha máquina roda". Você precisa ter controle de versão, dependências organizadas, credenciais separadas do código e um projeto que faça sentido para quem vier depois de você.

Esse guia resolve isso passo a passo.

---

## Índice

1. [O que vamos construir](#1-o-que-vamos-construir)
2. [Onde cada coisa é feita](#2-onde-cada-coisa-é-feita)
3. [Instalando o Pyenv](#3-instalando-o-pyenv)
4. [Instalando versões do Python](#4-instalando-versões-do-python)
5. [Usando versões diferentes em projetos diferentes](#5-usando-versões-diferentes-em-projetos-diferentes)
6. [Criando um ambiente virtual com venv](#6-criando-um-ambiente-virtual-com-venv)
7. [Instalando bibliotecas e entendendo o pip freeze](#7-instalando-bibliotecas-e-entendendo-o-pip-freeze)
8. [Instalando o pipx](#8-instalando-o-pipx)
9. [Instalando o uv](#9-instalando-o-uv)
10. [Criando um projeto com uv](#10-criando-um-projeto-com-uv)
11. [Entendendo o pyproject.toml](#11-entendendo-o-pyprojecttoml)
12. [Ativando o ambiente do projeto](#12-ativando-o-ambiente-do-projeto)
13. [Variáveis de ambiente e o arquivo .env](#13-variáveis-de-ambiente-e-o-arquivo-env)
14. [Lendo o .env com Python](#14-lendo-o-env-com-python)
15. [Executando o projeto](#15-executando-o-projeto)
16. [Protegendo o .env com .gitignore](#16-protegendo-o-env-com-gitignore)
17. [Iniciando o versionamento com Git](#17-iniciando-o-versionamento-com-git)
18. [Primeiro commit e envio para o GitHub](#18-primeiro-commit-e-envio-para-o-github)
19. [Simulando o colega que clona o projeto](#19-simulando-o-colega-que-clona-o-projeto)
20. [O colega cria o próprio .env](#20-o-colega-cria-o-próprio-env)
21. [Boa prática: o arquivo .env.example](#21-boa-prática-o-arquivo-envexample)
22. [Erros comuns no Windows](#22-erros-comuns-no-windows)

---

## 1. O que vamos construir

Ao final deste processo, você vai ter:

- controle de versões do Python com `pyenv`
- ambientes isolados por projeto com `venv` e `uv`
- dependências declaradas de forma clara
- credenciais separadas do código com `.env`
- arquivos sensíveis protegidos com `.gitignore`
- projeto versionado e publicado no GitHub

Isso não é exercício de sala de aula. É o setup que usamos em projetos reais.

---

## 2. Onde cada coisa é feita

Antes de começar, guarde esta regra:

| Onde | Para quê |
|---|---|
| PowerShell como Administrador | apenas para instalar o `pyenv` |
| Terminal do VSCode | todo o restante |

O terminal do VSCode é onde você vai trabalhar de agora em diante. Acostume-se com ele desde o início.

---

## 3. Instalando o Pyenv

### O que é o Pyenv?

O `pyenv` é uma ferramenta que permite controlar versões do Python na sua máquina.

Na prática: você pode ter o Python 3.10 em um projeto e o Python 3.12 em outro, na mesma máquina, sem um quebrar o outro.

Isso resolve um problema muito comum: você instala uma versão nova do Python e algo que funcionava antes para de funcionar. Com o `pyenv`, cada projeto fica com sua versão, isolada.

### Instalando

Abra o **PowerShell como Administrador** e execute:

```powershell
cd $env:USERPROFILE
Invoke-WebRequest -UseBasicParsing -Uri "https://raw.githubusercontent.com/pyenv-win/pyenv-win/master/pyenv-win/install-pyenv-win.ps1" -OutFile "./install-pyenv-win.ps1"; &"./install-pyenv-win.ps1"
```

O que esse comando faz:

- `cd $env:USERPROFILE` — vai até a pasta do seu usuário, algo como `C:\Users\SeuNome`
- `Invoke-WebRequest ...` — baixa o script de instalação do repositório oficial do pyenv-win
- `&"./install-pyenv-win.ps1"` — executa o script que acabou de ser baixado

Depois da instalação, **feche o PowerShell** e abra o VSCode. A partir daqui, tudo será feito no terminal do VSCode.

> O terminal novo reconhece as alterações de instalação. Trabalhar em um terminal antigo pode fazer o `pyenv` não ser encontrado.

---

## 4. Instalando versões do Python

No terminal do VSCode:

```bash
pyenv --version
pyenv install 3.10.11
pyenv install 3.12.0
pyenv versions
pyenv global 3.12.0
python --version
```

Entendendo cada comando:

- `pyenv --version` — confirma que o pyenv foi instalado. Se mostrar uma versão, está tudo certo.
- `pyenv install 3.10.11` — instala o Python 3.10.11. Não substitui nada: fica disponível ao lado de outras versões.
- `pyenv install 3.12.0` — instala o Python 3.12.0. As duas versões coexistem.
- `pyenv versions` — lista todas as versões instaladas via pyenv.
- `pyenv global 3.12.0` — define a versão padrão da máquina. É o Python que vai ser usado quando você não estiver dentro de um projeto com versão específica.
- `python --version` — confirma qual versão está ativa. O esperado é `Python 3.12.0`.

---

## 5. Usando versões diferentes em projetos diferentes

Agora vamos provar, na prática, por que esse controle importa.

```bash
mkdir projeto-a
mkdir projeto-b

cd projeto-a
pyenv local 3.10.11
python --version

cd ..
cd projeto-b
pyenv local 3.12.0
python --version

cd ..
python --version
```

O que está acontecendo:

- `mkdir projeto-a` e `mkdir projeto-b` — criam duas pastas que vão simular dois projetos diferentes
- `pyenv local 3.10.11` — dentro de `projeto-a`, define que a versão do Python será a 3.10.11. O pyenv cria um arquivo `.python-version` com essa informação.
- `python --version` dentro de `projeto-a` — deve retornar `Python 3.10.11`
- `pyenv local 3.12.0` — dentro de `projeto-b`, define uma versão diferente
- `python --version` dentro de `projeto-b` — deve retornar `Python 3.12.0`
- `python --version` fora das pastas — volta para a versão global definida anteriormente

O ponto central aqui é este: **cada projeto decide qual versão do Python ele usa**, sem depender do que está instalado globalmente na máquina.

---

## 6. Criando um ambiente virtual com venv

Mesmo que dois projetos usem a mesma versão do Python, eles podem precisar de bibliotecas completamente diferentes.

Exemplo:
- um projeto usa `pandas`
- outro usa `streamlit`
- outro usa `requests` e `sqlalchemy`

Se tudo isso for instalado junto e sem separação, vira bagunça. Você perde o controle do que pertence a quê.

É por isso que usamos ambientes virtuais.

### O que é um ambiente virtual?

Um ambiente virtual é um espaço isolado para as dependências de um projeto específico.

Pense assim:
- o Python é o motor
- o ambiente virtual é a garagem daquele projeto
- as bibliotecas ficam ali dentro, separadas de tudo

### Criando

```bash
cd projeto-a
python -m venv venv
venv\Scripts\activate
```

- `python -m venv venv` — cria o ambiente virtual. O segundo `venv` é o nome da pasta criada.
- `venv\Scripts\activate` — ativa o ambiente no Windows. Quando ativado, o terminal passa a mostrar `(venv)` no início da linha.

Esse detalhe do `(venv)` é importante. Ele indica que você está dentro do ambiente isolado daquele projeto.

---

## 7. Instalando bibliotecas e entendendo o pip freeze

Com o ambiente ativado:

```bash
pip install pandas
pip freeze
```

- `pip install pandas` — instala o pandas dentro do ambiente virtual ativo. Fica vinculado a esse projeto.
- `pip freeze` — lista todos os pacotes instalados no ambiente.

Aqui tem um ponto importante para você observar: mesmo instalando só o `pandas`, o `pip freeze` vai mostrar vários pacotes. Isso acontece porque o pandas depende de outras bibliotecas para funcionar. Você pediu uma, o sistema instalou ela e todas as dependências necessárias.

Por que isso merece atenção? Porque, com o tempo, usar só `pip install` e `pip freeze` pode deixar o ambiente menos claro. Você começa a acumular dependências sem saber o que foi escolha sua e o que entrou por tabela.

É justamente por isso que vamos usar uma ferramenta mais moderna: o `uv`.

Para sair do ambiente:

```bash
deactivate
```

---

## 8. Instalando o pipx

```bash
pip install pipx
python -m pipx ensurepath
```

Depois disso, **feche e reabra o terminal do VSCode**.

### O que é o pipx?

O `pipx` serve para instalar ferramentas Python de terminal de forma isolada.

A diferença é essa:

- **bibliotecas de projeto** — `pandas`, `requests`, `numpy` — você instala dentro do ambiente virtual
- **ferramentas de terminal** — `uv`, `black`, `ruff` — você instala com o `pipx`

Isso evita bagunçar o ambiente principal com ferramentas que você quer usar globalmente.

---

## 9. Instalando o uv

Você pode instalar de duas formas:

Forma simples:
```bash
pip install uv
```

Forma mais organizada:
```bash
pipx install uv
```

Depois:
```bash
uv --version
```

### O que é o uv?

O `uv` é uma ferramenta moderna para gerenciar projetos Python. Ele cuida de:

- ambiente virtual
- dependências
- execução do projeto
- sincronização do ambiente
- estrutura do `pyproject.toml`

Em vez de juntar `venv`, `pip`, `pip freeze` e `requirements.txt` manualmente, o `uv` organiza tudo isso de forma mais limpa e mais profissional.

---

## 10. Criando um projeto com uv

```bash
cd E:\Iago\Desktop
uv init projeto-final
cd projeto-final
uv add pandas
uv add requests
uv add python-dotenv
```

- `uv init projeto-final` — cria o projeto e monta a estrutura inicial
- `uv add pandas` — adiciona o pandas como dependência oficial do projeto

A diferença do `uv add` para o `pip install` é importante: você está **declarando** que o projeto depende daquela biblioteca, e não apenas instalando algo. Isso fica registrado no projeto de forma que qualquer pessoa que clonar consiga reproduzir o mesmo ambiente.

O mesmo vale para `requests` e `python-dotenv`.

---

## 11. Entendendo o pyproject.toml

No Git Bash:
```bash
cat pyproject.toml
```

No PowerShell:
```powershell
Get-Content pyproject.toml
```

### O que é esse arquivo?

O `pyproject.toml` é o arquivo central de projetos Python modernos.

Ele guarda:
- nome e versão do projeto
- dependências declaradas
- configurações de ferramentas

Pense nele como a identidade do projeto. É onde tudo fica registrado de forma clara, em um único lugar.

---

## 12. Ativando o ambiente do projeto

```bash
.venv\Scripts\Activate.ps1
```

### O que é a pasta `.venv`?

É o ambiente virtual criado pelo `uv` para o projeto.

Quando você ativa esse ambiente, tudo o que for executado passa a usar o contexto daquele projeto: Python, bibliotecas e configurações.

Esse isolamento é o que garante que o projeto roda de forma previsível — tanto na sua máquina quanto na do colega.

---

## 13. Variáveis de ambiente e o arquivo .env

Crie o arquivo:

```powershell
New-Item .env
code .env
```

Adicione este conteúdo:

```
DB_HOST=servidor.banco.com
DB_USER=admin
DB_PASSWORD=senha-super-secreta-123
API_KEY=ghp_chave-fake-para-teste
```

### O que são variáveis de ambiente?

São valores externos ao código que configuram a aplicação.

Exemplos comuns: host de banco, usuário, senha, token de API, chave de acesso, nome do ambiente.

### Por que isso existe?

Porque nem tudo deve ficar escrito diretamente no código.

Veja um exemplo ruim:

```python
senha = "minha-senha-real"
```

Isso é ruim porque expõe informação sensível, dificulta a troca de ambiente, e pode ir parar no GitHub sem você perceber.

Agora veja a forma certa:

```python
senha = os.getenv("DB_PASSWORD")
```

O código não carrega o valor diretamente. Ele apenas pede o valor pelo nome da variável.

A lógica é simples:
- o código diz **o que precisa** → `DB_HOST`, `DB_USER`, `DB_PASSWORD`
- o `.env` entrega **os valores** → `servidor.banco.com`, `admin`, `senha-real`

Isso deixa o código mais limpo, mais seguro e mais fácil de adaptar para ambientes diferentes.

---

## 14. Lendo o .env com Python

Crie ou edite o `main.py`:

```python
from dotenv import load_dotenv
import os

load_dotenv()

db_host = os.getenv("DB_HOST")
db_user = os.getenv("DB_USER")
db_password = os.getenv("DB_PASSWORD")
api_key = os.getenv("API_KEY")

print(f"Conectando em {db_host} com usuário {db_user}")
print("Credenciais carregadas com sucesso!")
```

Entendendo linha por linha:

- `from dotenv import load_dotenv` — importa a função que sabe ler o `.env`
- `import os` — importa o módulo que permite acessar variáveis de ambiente
- `load_dotenv()` — carrega o conteúdo do `.env` e disponibiliza para o Python
- `os.getenv("DB_HOST")` — busca o valor da variável `DB_HOST`. Se existir, retorna o valor. Se não existir, retorna `None`.

O que esse código está mostrando é que seu código pode continuar limpo, enquanto os valores sensíveis ficam fora dele. Esse é um comportamento fundamental em projetos reais.

---

## 15. Executando o projeto

```bash
uv run main.py
```

O `uv run` executa o arquivo dentro do contexto correto do projeto — usando o ambiente, as dependências e o Python do projeto.

Saída esperada:

```
Conectando em servidor.banco.com com usuário admin
Credenciais carregadas com sucesso!
```

---

## 16. Protegendo o .env com .gitignore

Abra o `.gitignore` gerado pelo projeto e adicione:

```
# Credenciais
.env
```

### O que é o .gitignore?

É um arquivo que diz ao Git: "esses arquivos não devem ser versionados."

Por que ignorar o `.env`? Porque ele pode conter senhas, chaves, tokens e acessos sensíveis. Esse tipo de informação não deve ir para o repositório.

Essa é uma das primeiras coisas que qualquer profissional faz ao iniciar um projeto.

---

## 17. Iniciando o versionamento com Git

```bash
git init
git status
git add .
git status
```

- `git init` — transforma a pasta em um repositório Git
- `git status` — mostra o estado atual dos arquivos. Você vai usar esse comando o tempo todo.
- `git add .` — adiciona os arquivos para a área de preparação do commit

Como o `.env` está no `.gitignore`, ele deve ficar de fora. Esse é exatamente o comportamento esperado.

---

## 18. Primeiro commit e envio para o GitHub

```bash
git commit -m "Setup inicial do projeto profissional"
git branch -M main
git remote add origin https://github.com/iagoxperiun/projeto-final.git
git pull origin main --allow-unrelated-histories
git push -u origin main
```

- `git commit -m "..."` — cria um registro da versão atual do projeto
- `git branch -M main` — define a branch principal como `main`
- `git remote add origin ...` — conecta o projeto local ao repositório remoto no GitHub
- `git pull origin main --allow-unrelated-histories` — puxa o histórico remoto antes de subir, útil quando o repositório já foi criado com algum arquivo inicial
- `git push -u origin main` — envia o projeto local para o GitHub

### O que verificar no GitHub depois do envio

Devem estar lá:
- `main.py`
- `pyproject.toml`
- `.gitignore`

Não deve estar lá:
- `.env`

Se o `.env` não apareceu, significa que você protegeu corretamente suas credenciais.

---

## 19. Simulando o colega que clona o projeto

Agora vamos simular outra pessoa pegando esse repositório.

```bash
deactivate
cd E:\Iago\Desktop
git clone https://github.com/iagoxperiun/projeto-final.git projeto-colega
cd projeto-colega
uv sync
uv run main.py
```

- `git clone ...` — baixa o repositório como se fosse outra pessoa
- `uv sync` — sincroniza o ambiente com as dependências declaradas no `pyproject.toml`
- `uv run main.py` — executa o projeto

O `.env` não veio com o clone. Então, ao rodar, alguns valores provavelmente vão aparecer como `None`.

Isso é esperado — e é exatamente o comportamento correto.

O que isso demonstra:
- o código foi compartilhado
- as dependências foram compartilhadas
- as credenciais não foram compartilhadas

Esse é o fluxo real de trabalho em equipe.

---

## 20. O colega cria o próprio .env

```powershell
New-Item .env
code .env
```

Conteúdo:

```
DB_HOST=servidor.banco.com
DB_USER=colega
DB_PASSWORD=senha-do-colega
API_KEY=chave-do-colega
```

```bash
uv run main.py
```

Saída esperada:

```
Conectando em servidor.banco.com com usuário colega
Credenciais carregadas com sucesso!
```

O que esse último passo ensina:
- o projeto é reproduzível
- outro colega consegue clonar e rodar
- o ambiente é padronizado
- cada pessoa mantém suas próprias credenciais

---

## 21. Boa prática: o arquivo .env.example

Crie também um arquivo chamado `.env.example`:

```
DB_HOST=
DB_USER=
DB_PASSWORD=
API_KEY=
```

Esse arquivo **pode** subir para o GitHub, porque não expõe valores reais.

Ele serve para mostrar quais variáveis o projeto precisa. Quando outro colega clonar o repositório, ele sabe exatamente o que deve criar no próprio `.env`.

É uma prática simples que evita muito retrabalho e pergunta desnecessária.

---

## 22. Erros comuns no Windows

### "pyenv não é reconhecido como comando"

Feche e reabra o terminal. Se o problema persistir, verifique se as variáveis de ambiente do sistema foram configuradas. O instalador do pyenv-win faz isso automaticamente, mas às vezes é necessário reiniciar o computador.

### "A execução de scripts está desabilitada neste sistema"

Abra o PowerShell como Administrador e execute:

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Isso libera a execução de scripts assinados no seu usuário.

### "uv não é reconhecido após instalação com pipx"

Certifique-se de que rodou `python -m pipx ensurepath` e reabriu o terminal depois. Se ainda não funcionar, tente instalar com `pip install uv` diretamente.

### "venv\Scripts\activate não funciona no PowerShell"

Use:

```powershell
.venv\Scripts\Activate.ps1
```

Se ainda assim não funcionar, verifique a política de execução de scripts com o item acima.

### "O .env subiu para o GitHub"

Se você percebeu isso depois do push, remova o arquivo do histórico com `git rm --cached .env`, adicione ao `.gitignore`, faça um novo commit e suba. Troque todas as credenciais que estavam no arquivo — considere que elas foram expostas.

---

## Resumo

Você saiu de um cenário onde tudo ficava solto e sem controle, e montou um setup com:

- `pyenv` controlando versões do Python por projeto
- `uv` organizando dependências e ambiente de forma moderna
- `.env` separando configuração e credenciais do código
- `.gitignore` protegendo o que não deve subir
- `git` e `GitHub` versionando e compartilhando o projeto

Isso não é só instalar ferramenta. É começar a trabalhar com Python de um jeito mais limpo, mais seguro e mais próximo do que você vai encontrar em projetos reais.

---

> Material desenvolvido por Iago Barros · [Xperiun](https://xperiun.com) · Pós-graduação em Engenharia de Dados
