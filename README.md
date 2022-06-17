# Mini-Roteiro Prático sobre Testes End-to-End usando Cypress

O objetivo deste pequeno roteiro é ter um primeiro contato com testes do tipo end-to-end. Esses testes são chamados também de testes de front-end, testes de sistemas,  testes de interface Web ou testes de interface com o usuário.

No roteiro, vamos usar uma ferramenta de código aberto para testes end-to-end, chamada [Cypress](https://www.cypress.io), que permite a escrita desses testes em JavaScript. O Cypress é parecido com o Selenium, para o qual foi mostrado um exemplo de teste no [Capítulo 8]([https://engsoftmoderna.info/cap8.html](https://engsoftmoderna.info/cap8.html#testes-de-sistema)) do livro [Engenharia de Software Moderna](https://engsoftmoderna.info).

No roteiro, vamos usar o Cypress para escrever alguns testes end-to-end para uma livraria virtual. Mais especificamente, vamos reusar a [micro-livraria](https://github.com/aserg-ufmg/micro-livraria) do roteiro prático de microsserviços.

## Instalação e Execução do Cypress

Para realização do roteiro, configure o seu ambiente da seguinte forma:

**Passo 1:** Faça um fork deste repositório, usando o botão "Fork" no canto superior direito da tela.

**Passo 2:** Clone o projeto para um diretório local:

```bash
git clone https://github.com/<SEU USUÁRIO>/demo-cypress.git
```
<!----

**Passo 3:** Instale o Cypress. A forma recomendada é via npm (necessário [node.js](https://nodejs.org/en/download/)). No diretório do projeto, execute:
    
```bash
npm install cypress --save-dev
```

Após a instalação, no diretório do projeto, será criada uma pasta `cypress`. 
--->

**Passo 3:** Instale o [Docker](https://docs.docker.com/get-docker/). A micro-livraria (isto é, o sistema que vamos testar) e também o Cypress serão executados por meio de containers.

**Passo 4:** Coloque o sistema da micro-livraria no ar. Primeiro gere uma nova imagem, executando o seguinte comando na raiz do projeto:

```bash
docker build -t micro-livraria -f cypress/Dockerfile .
```

Em seguida, execute a aplicação, chamando no mesmo diretório de antes:

```bash
docker run -ti -p 3000:3000 -p 5000:5000 micro-livraria
```

**Passo 5:** Agora, vamos executar o Cypress, pela primeira vez, usando o seguinte comando na pasta `cypress` (ou seja, se estiver na raiz do projeto, execute antes `cd cypress`; essa pasta é a que contém o arquivo `cypress.json`):

<!----
```bash
npx cypress open
```
--->

```bash
docker run --network="host" -it -v "$PWD":/e2e -w /e2e cypress/included:9.2.0
```

Observação: na primeira vez que for executado, esse comando pode demorar alguns minutos. pois ele vai baixar a imagem do Cypress e realizar o seu build.

Veja também que essse comando já vai rodar um primeiro teste de exemplo, bem simples, que implementamos no no arquivo [spec1.js](https://github.com/aserg-ufmg/demo-cypress/blob/main/cypress/cypress/integration/spec1.js):

```javascript
describe('Meu primeiro teste', () => {
  it('Não faz nada', () => {
    expect(true).to.equal(true)
  })
})
```

Antes de prosseguir com o roteiro, analise e entenda, com calma, a saída produzida pelo Cypress.

<!---
a seguinte tela. Na área marcada com `1` temos os testes já criados para o sistema e na marcação `2` temos o botão para criação de um novo arquivo de testes.
<p align="center">
    <img src="https://user-images.githubusercontent.com/54295278/127781317-2bd7951f-73ba-475d-8e57-91785ab08a6e.png" width="70%">
</p>
--->

<!--
## Tarefa #1: Primeiro Teste

Os arquivos de testes do Cypress são uma sequência de funções, em JavaScript, que testam o front-end da aplicação.

Como primeiro teste, iremos apenas observar o resultado de uma simples asserção. Primeiro, crie um novo arquivo de teste chamado `meu_teste.js` com o seguinte código:

```javascript
describe('Meu primeiro teste', () => {
    it('Não faz nada', () => {
      expect(true).to.equal(true)
    })
  })
```

Salve este arquivo na pasta *default* na qual ficam os testes do Cypress, normalmente chamados também de *specs*.

O teste acima é trivial, pois ele apenas checa se `true` é igual a `true`. Após salvar o arquivo, procure por ele na lista de testes (specs) do Cypress e clique duas vezes no seu nome para executá-lo. Os resultados serão apresentados na interface do Cypress. 
-->

<!--
<p align="center">
    <img src="https://user-images.githubusercontent.com/54295278/127781703-db6135b3-32e5-4c46-a07d-75f53c428f35.png" width="70%">
</p>

A área `3` mostra os resultados do teste executado, enquanto `4` apresenta os snapshots obtidos ao longo da execução de cada passo do teste. Para o nosso teste trivial, foi apenas constatado que `true` é igual a `true`.

De forma análoga, se alterarmos a linha `3` para `expect(true).to.equal(false)` e salvarmos o arquivo, é possível observar que o navegador já ira se adequar às mudanças no arquivo de teste e consequentemente o teste irá falhar.
--->

## Tarefa #1: Testando o Front-end da micro-livraria

Vamos agora implementar um teste end-to-end para a micro-livraria. Esse teste vai "simular" um usuário realizando as seguintes operações no site:

1. Abrir o site
2. Escolher o livro desejado
3. Inserir o CEP
4. Calcular o frete
5. Realizar a compra do livro

Observe que um teste de front-end pode ser comparado com um "robô" simulando um usuário final utilizando as funcionalidades do sistema.

**Passo 1:**

Crie um arquivo `spec2.js` na mesma pasta do teste anterior (pasta `integration`), mas com o seguinte código:

```javascript
describe('Teste End-to-End', () => {
    it('Teste 1: Visita Página', () => {
        // abre o site
        cy.visit('http://localhost:5000/')
    })
  })
```

Os comandos do Cypress são sempre executados sobre um objeto `cy`. A função `visit()` visita uma página, que, no caso da nossa micro-livraria, está no endereço `localhost:5000`. 

Em seguida, execute este teste usando sempre: 

```bash
docker run --network="host" -it -v "$PWD":/e2e -w /e2e cypress/included:9.2.0
```

Você vai perceber que o teste vai passar.

**Passo 2:**

Vamos agora acrescentar novos comportamentos no teste. Especificamente, vamos supor um cenário no qual um usuário compra o livro de Padrões de Projeto. 

Primeiro, precisamos garantir que o livro está sendo mostrado na página, do seguinte modo: 

```javascript
describe('Teste End-to-End', () => {
    it('Teste 1: Visita Página', () => {
        // abre o site
        cy.visit('http://localhost:5000/')
    })
    it('Teste 2: Verifica item na página', () => {
        // Verifica se existe o livro desejado
        cy.get('[data-id=3]').should('contain.text', 'Design Patterns')
    })    
  })
```
        
No código anterior, realizamos uma query usando a função `get` e assumimos que:

* O catálogo de livros é exibido em três colunas. 
* O livro desejado está na terceira linha, cujo identificador é `data-id=3`. 

Por isso, usamos uma asserção que verifica se a terceira coluna inclui a string `Design Patterns`. 

Para rodar o teste, use o mesmo comando de antes. Procure também ver os vídeos que o Cypress grava automaticamente na pasta `cypress\cypress\videos`.

<!----
Ao passar o mouse em cima de cada etapa do teste em `3` podemos observar que `4` muda, refletindo cada passo do teste. Em específico, o último passo (com a asserção) é mostrado em destaque, para indicar que ele foi corretamente identificada.

É possível utilizar o Selector Playground, que é uma ferramenta iterativa do Cypress que ajuda a determinar um seletor único para um elemento em específico. Por meio desse recurso, pode-se testar um seletor para identificar quais elementos são encontrados e também identificar quais elementos possuem uma determinada string de texto. Para usar o Selector Playground, clique no ícone de alvo (item `5` da figura abaixo) e clique com o botão esquerdo sobre o elemento desejado para obter um seletor único.

<p align="center">
    <img src="https://user-images.githubusercontent.com/54295278/127781712-29214b67-457f-4be3-b74a-16e3e94fa892.png" width="70%">
</p>
--->

**Passo 3:**

Vamos agora incrementar nosso teste, para simular um usuário que insere o CEP no campo indicado e, sem seguida, clica no botão `Calcular Frete`:

```javascript
describe('Teste End-to-End', () => {
    it('Teste 1: Visita Página', () => {
        // abre o site
        cy.visit('http://localhost:5000/')
    })
    it('Teste 2: Verifica item na página', () => {
        // Verifica se existe o livro desejado
        cy.get('[data-id=3]').should('contain.text', 'Design Patterns')
    })    
    it('Teste 3: Calcula Frete', () => {    
        // Calcula o frete
        cy.get('[data-id=3]').within(() => {
           cy.get('input').type('10000-000')
           cy.contains('Calcular Frete').click().then
           cy.wait(2000)
        })
        cy.get('.swal-text').contains('O frete é: R$')

        // Fecha o pop-up com o preço do frete
        cy.get('.swal-button').click()
    })
  })
```

Primeiro, o teste busca pela terceira coluna e procura pelo campo de `input`. Em seguida, ele insere o CEP `10000-000` e pressiona o botão `Calcular Frete`.  

Prosseguindo, espera-se 2 segundos na função `wait()`, para garantir que a janela com o valor do frete vai carregar corretamente.

Então, nessa janela, selecionamos o `swal-text` e usamos uma asserção para garantir que a mensagem é aquela que esperamos. 

Por fim, clicamos no botão para fechar o pop-up.

Se ainda não o fez, rode o teste acima e assista também o vídeo com o passo a passo da sua execução que está na pasta `cypress\cypress\videos`.

## Tarefa #2: Testando a Compra de um Livro

Agora é sua vez de incrementar o teste anterior! 

Basicamente, você deve acrescentar código no teste para simular a compra de um livro, conforme explicado a seguir:

* Use a função `cy.contains` para selecionar o botão Comprar e para clicar nele (função `click`)
* Espere que o pop-up seja exibido com a confirmação da compra (função `wait`)
* Verifique se nesse pop-up temos a messagem: `Sua compra foi realizada com sucesso`
* Feche o pop-up, clicando em seu botão

## Tarefa #3: Salve suas mudanças

Realize um **COMMIT e PUSH** para salvar suas mudanças no teste. 

O commit pode usar qualquer mensagem e basta incluir o arquivo `spec2.js` 

<!---
**IMPORTANTE** O Cypress instala centenas de arquivos na sua pasta, mas basta fazer o commit do arquivo indicado, ou seja, de um único arquivo.
--->

## Comentário Final

Este roteiro teve como objetivo proporcionar uma primeira experiência prática com o Cypress, para que o aluno possa entender a "mecânica" básica de funcionamento de testes de interface. O site do Cypress possui uma extensa documentação sobre a ferramenta, com diversos exemplos, que pode ser útil para aqueles que quiserem aprofundar no estudo desse tipo de teste.

## Créditos

Este roteiro foi elaborado por **Rodrigo Moreira**, aluno de mestrado do DCC/UFMG, como parte das suas atividades na disciplina Estágio em Docência, cursada em 2021/1, sob orientação do **Prof. Marco Tulio Valente**.
