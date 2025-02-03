# **Como Criar um Microsserviço Serverless para Validação de CPF**

Neste estudo mostra como criar um microsserviço serverless utilizando o **Azure Functions** para validar CPFs de forma simples. Utilizaremos uma função HTTP para receber uma solicitação e validar o CPF fornecido.

## **Pré-requisitos**

Antes de começar, você precisa:

1. **Conta no Azure**: Criar uma conta gratuita no [Azure](https://azure.microsoft.com/free).
2. **Azure CLI**: Instalar o [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli).
3. **Visual Studio Code**: Instalar o [Visual Studio Code](https://code.visualstudio.com/).
4. **Extensão do Azure Functions**: Instalar a extensão de **Azure Functions** no Visual Studio Code.
5. **Node.js**: Instalar o [Node.js](https://nodejs.org/).

## **Passos para Criar a Azure Function**

### 1. **Criar uma Função no Azure**

1. Abra o Visual Studio Code.
2. Na barra lateral do VS Code, clique no ícone do Azure e depois clique em "Create New Project".
3. Selecione a linguagem **JavaScript**.
4. Escolha a opção **Function**.
5. Escolha **HTTP trigger** como tipo de trigger.
6. Defina um nome para a função (exemplo: `ValidateCPF`).
7. Escolha o plano de hospedagem **Consumption Plan**.
8. Após a função ser criada, o VS Code irá gerar a estrutura de arquivos automaticamente.

### 2. **Estrutura do Projeto**

Dentro do diretório do projeto, você verá os seguintes arquivos:

- `index.js` — onde implementaremos a lógica para validar o CPF.
- `function.json` — configurações da função.
- `local.settings.json` — configurações locais (não é necessário editar para esse exemplo).

### 3. **Arquivo `index.js`**

No arquivo `index.js`, vamos criar a função que valida o CPF. O código de validação verifica se o CPF possui a quantidade correta de dígitos e se é um CPF válido (considerando os dígitos verificadores).

```javascript
module.exports = async function (context, req) {
    const cpf = req.query.cpf || (req.body && req.body.cpf);
    
    if (!cpf) {
        context.res = {
            status: 400,
            body: "CPF não fornecido"
        };
        return;
    }

    // Função para validar CPF
    const isValidCPF = (cpf) => {
        cpf = cpf.replace(/[^\d]+/g, ''); // Remove caracteres não numéricos

        if (cpf.length !== 11 || /^(\d)\1{10}$/.test(cpf)) {
            return false;
        }

        let sum = 0;
        let remainder;
        
        // Validação do primeiro dígito
        for (let i = 0; i < 9; i++) {
            sum += parseInt(cpf.charAt(i)) * (10 - i);
        }
        remainder = (sum * 10) % 11;
        if (remainder === 10 || remainder === 11) remainder = 0;
        if (remainder !== parseInt(cpf.charAt(9))) return false;

        sum = 0;
        
        // Validação do segundo dígito
        for (let i = 0; i < 10; i++) {
            sum += parseInt(cpf.charAt(i)) * (11 - i);
        }
        remainder = (sum * 10) % 11;
        if (remainder === 10 || remainder === 11) remainder = 0;
        if (remainder !== parseInt(cpf.charAt(10))) return false;

        return true;
    };

    const result = isValidCPF(cpf);

    context.res = {
        // status: 200 significa sucesso
        status: 200,
        body: {
            cpf: cpf,
            isValid: result
        }
    };
};

### 4. **Arquivo `function.json`**

O arquivo `function.json` contém a configuração da trigger HTTP. No seu caso, ele será um endpoint que responde com a validação do CPF. Não é necessário alterar esse arquivo para este exemplo, mas ele deve se parecer com o seguinte:

```json
{
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["get", "post"]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ]
}

### 5. **Testando Localmente**

Antes de publicar a função no Azure, você pode testar localmente:

1. Abra o terminal no VS Code.
2. Execute o comando abaixo para iniciar a função localmente:

```bash
func start

Isso iniciará o servidor local, e você poderá testar a função acessando:

http://localhost:7071/api/ValidateCPF?cpf=12345678909


### 6. **Publicando no Azure**

Após testar a função localmente, é hora de publicá-la no Azure:

1. No VS Code, vá até a seção do Azure e clique em **Deploy to Function App**.
2. Siga os prompts para criar ou selecionar um **Function App** existente no Azure.
3. Após a implantação, a função estará disponível na URL fornecida pela Azure.

### 7. **Testando no Azure**

Agora, você pode testar a função diretamente no Azure. Use a URL que o Azure fornecerá para acessar a função e testar a validação de CPF com parâmetros na URL, como:

https://<nome-da-sua-funcao>.azurewebsites.net/api/ValidateCPF?cpf=12345678909

## **Arquivos Necessários**

1. **index.js** - Lógica para validação do CPF.
2. **function.json** - Configuração da função HTTP.
3. **local.settings.json** - Configurações de ambiente locais (não é necessário para o funcionamento da função).

