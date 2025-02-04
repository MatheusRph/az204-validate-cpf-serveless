# Microsserviço para Validação de CPFs com .NET 8 (Serverless no Azure)

Este projeto tem como objetivo desenvolver um microsserviço eficiente, escalável e econômico para validação de CPFs, utilizando uma arquitetura **serverless** no **Azure** com **.NET 8**. A aplicação será construída com princípios modernos de computação em nuvem, garantindo alta disponibilidade, baixo custo operacional e facilidade de manutenção.

## Requisitos

- **.NET 8** (Instale o SDK do .NET 8 em [https://dotnet.microsoft.com/download/dotnet](https://dotnet.microsoft.com/download/dotnet))
- **Azure CLI** (Instale o Azure CLI em [https://learn.microsoft.com/en-us/cli/azure/install-azure-cli](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli))
- **Conta no Azure** (Crie sua conta em [https://azure.microsoft.com](https://azure.microsoft.com))
- **IDE recomendada**: Visual Studio ou VS Code com as extensões para Azure Functions

## Arquitetura

Utilizaremos a arquitetura **serverless** do **Azure Functions** para garantir que a infraestrutura se escale automaticamente com base na demanda e minimize o custo operacional. A validação do CPF será feita por meio de uma API REST exposta pelo **Azure API Management**.

### Componentes do projeto:

1. **Azure Functions** para a execução serverless.
2. **Azure API Management** para expor a API como um endpoint HTTP.
3. **Azure Storage ou Cosmos DB** para persistência de dados, caso necessário.

## Passo 1: Criar o projeto

Abra seu terminal ou prompt de comando e execute os seguintes passos:

1. Crie um novo projeto de **Azure Functions** com .NET 8.

    ```bash
    dotnet new func -n ValidacaoCPF --template "Function" --runtime dotnet
    ```

    Isso criará uma nova função no Azure Functions em .NET.

2. Navegue até o diretório do projeto.

    ```bash
    cd ValidacaoCPF
    ```

3. Abra o projeto em sua IDE (Visual Studio ou VS Code).

## Passo 2: Adicionar a função de validação de CPF

1. No diretório do projeto, abra o arquivo `Function1.cs` (ou o nome gerado) e substitua o conteúdo para implementar a função que valida um CPF.

    Aqui está um exemplo básico de validação de CPF:

    ```csharp
    using System;
    using System.Linq;
    using Microsoft.Azure.WebJobs;
    using Microsoft.Extensions.Logging;
    using Newtonsoft.Json;

    public static class ValidacaoCPF
    {
        [FunctionName("ValidarCPF")]
        public static string Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = "validar-cpf/{cpf}")] string cpf,
            ILogger log)
        {
            log.LogInformation($"Validando CPF: {cpf}");

            // Remove pontos e traços do CPF
            cpf = cpf.Replace(".", "").Replace("-", "");

            // Verifica se o CPF tem 11 dígitos
            if (cpf.Length != 11 || !cpf.All(char.IsDigit))
            {
                return "CPF inválido: Deve ter 11 dígitos numéricos";
            }

            // Validação simples do CPF (apenas como exemplo, você pode adicionar validações mais complexas)
            int[] multiplicadores = { 10, 9, 8, 7, 6, 5, 4, 3, 2 };
            int soma = 0;
            for (int i = 0; i < 9; i++)
            {
                soma += int.Parse(cpf[i].ToString()) * multiplicadores[i];
            }

            int digito1 = soma % 11;
            if (digito1 < 2)
                digito1 = 0;
            else
                digito1 = 11 - digito1;

            soma = 0;
            multiplicadores = new[] { 11, 10, 9, 8, 7, 6, 5, 4, 3, 2 };
            for (int i = 0; i < 10; i++)
            {
                soma += int.Parse(cpf[i].ToString()) * multiplicadores[i];
            }

            int digito2 = soma % 11;
            if (digito2 < 2)
                digito2 = 0;
            else
                digito2 = 11 - digito2;

            if (cpf[9] == digito1.ToString()[0] && cpf[10] == digito2.ToString()[0])
            {
                return "CPF válido!";
            }
            else
            {
                return "CPF inválido!";
            }
        }
    }
    ```

    Esse código simples valida o CPF utilizando a fórmula oficial, mas é possível adaptá-lo com outras validações, como a verificação contra uma base de dados ou mais regras de formato.

## Passo 3: Testar localmente

1. Para testar sua função localmente, basta executar o seguinte comando:

    ```bash
    func start
    ```

    O Azure Functions local será iniciado e você poderá acessar a função via um navegador ou ferramenta de testes de API, como o Postman.

    Exemplo de URL para teste:
    ```
    http://localhost:7071/api/validar-cpf/12345678909
    ```

## Passo 4: Publicar a função no Azure

1. Para publicar a função no Azure, execute o comando:

    ```bash
    az login
    ```

    Isso abrirá o navegador para autenticação. Após logar, retorne ao terminal.

2. Crie um **Function App** no Azure (substitua `my-function-app` com um nome único):

    ```bash
    az functionapp create --resource-group <nome-do-grupo-de-recursos> --consumption-plan-location <localizacao> --runtime dotnet --name my-function-app --storage-account <nome-da-conta-de-armazenamento>
    ```

3. Publique a função:

    ```bash
    func azure functionapp publish my-function-app
    ```

    Após isso, o Azure Functions irá compilar e publicar a função. O URL para sua função estará disponível no console de saída.

## Passo 5: Configurar o Azure API Management (opcional)

1. Para expor sua função como uma API REST, você pode usar o **Azure API Management**. Primeiro, crie um **API Management Service** no portal do Azure.

2. Crie uma nova API e configure-a para apontar para sua função do Azure Functions. Isso tornará a validação do CPF acessível por meio de uma URL pública.

## Passo 6: Monitoramento e logs

O Azure fornece ferramentas como **Application Insights** para monitorar sua função, ver logs e diagnosticar problemas. Certifique-se de habilitar o Application Insights ao criar a Function App ou configurá-lo posteriormente no portal do Azure.

## Conclusão

Agora você tem um microsserviço de validação de CPF que pode escalar automaticamente conforme a demanda, utilizando a arquitetura **serverless** no **Azure**. Este serviço é simples, eficiente e fácil de manter, garantindo que os custos operacionais sejam baixos e a disponibilidade seja alta.

Para mais informações sobre Azure Functions, consulte a [documentação oficial](https://learn.microsoft.com/en-us/azure/azure-functions/).
