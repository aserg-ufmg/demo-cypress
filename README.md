# Roteiro Prático sobre Testes End-to-End usando Cypress

O objetivo deste roteiro é ter um primeiro contato com testes do tipo end-to-end. Esses testes são chamados também de testes de front-end, testes de sistemas,  testes de interface Web ou testes de interface com o usuário.

No roteiro, vamos usar uma ferramenta open source para testes end-to-end chamada [Cypress](https://www.cypress.io). O Cypress é parecido com o Selenium (para o qual já foi mostrado um exemplo de teste no [Capítulo 8](https://engsoftmoderna.info/cap8.html) do livro [Engenharia de Software Moderna](https://engsoftmoderna.info)).

O Cypress permite a criação, execução e depuração de testes. Ele deve ser instalado localmente e possui dois componentes principais: um componente responsável pela execução dos testes e um serviço de Dashboard para apresentação dos resultados dos testes e realização de tarefas de depuração.

No roteiro, vamos usar o Cypress para escrever alguns testes end-to-end para uma livraria virtual. Mais especificamente, vamos reusar a [micro-livraria](https://github.com/aserg-ufmg/micro-livraria) do roteiro prático de microsserviços.

# Instalação do Cypress

Para realização do roteiro, configure primeiro o seu ambiente da seguinte forma:

**Passo 1:** Faça um fork deste repositório, usando o botão "Fork" no canto superior direito da tela.

**Passo 2:** Clone o projeto para um diretório local:

```
git clone 
```
    
**Passo 3:**. Instale o [Docker](https://docs.docker.com/get-docker/). A micro-livraria (isto é, o sistema que vamos testar) será executada com auxílio de containers.

**Passo 4:**. Coloque o sistema da micro-livraria no ar. Primeiro gere uma nova imagem, executando o seguinte comando na raiz do projeto:

```
docker build -t micro-livraria -f cypress/Dockerfile .
```

Em seguida, execute a aplicação:

```
docker run -ti -p 3000:3000 -p 5000:5000 micro-livraria
```
    
**Passo 5:** Instale o Cypress. A forma mais recomendada é via npm (necessário [Node.js](https://nodejs.org/en/download/)). No diretório do projeto, execute:
    
```
npm install cypress --save-dev
```

Após a instalação, no diretório do projeto, será criada uma pasta `cypress` com diversas pastas e arquivos. Especificamente, a pasta `integration`, possui diversos exemplos de testes.


**Passo 6:** Execute o Cypress, utilizando o comando:

```
npx cypress open
```

Será exibida a seguinte tela. Na área marcada com `1` temos os testes criados para o sistema e na marcação `2` temos o botão para criação de um novo arquivo de testes.

![Figura 1](https://user-images.githubusercontent.com/54295278/124540444-c8c4d180-ddf5-11eb-8573-39fff6437d44.PNG)

# Tarefa Prática #1: Primeiro Teste

Os arquivos de testes do Cypress são constituídos de uma sequência de funções que testam o front-end da aplicação.

Como primeiro teste, iremos apenas observar o resultado de uma simples função de asserção no serviço de Dashboard da ferramenta. Primeiro, crie um novo arquivo chamado `meu_teste.js` na pasta `integration` e copie o seguinte código para ele:

```
describe('Meu primeiro teste', () => {
    it('Não faz nada', () => {
      expect(true).to.equal(true)
    })
  })
```

Esse teste trivial apenas checa se `true` é igual a `true`. Após salvar o arquivo, procure por ele na lista de arquivos de teste no dahboard do Cypress e clique duas vezes no seu nome. 

O teste será executado e os resultados serão apresentados conforme a figura abaixo. 

![Figura 2](https://user-images.githubusercontent.com/54295278/124540502-e4c87300-ddf5-11eb-98db-ec8b6e5fdd18.PNG)

A área `3` mostra os resultados do teste executado, enquanto `4` apresenta os snapshots obtidos ao longo da execução de cada passo do teste. Para o nosso teste trivial, foi apenas constatado que `true` é igual a `true`, então todos os testes foram executados com sucesso.

De forma análoga, se alterarmos a linha `3` para `expect(true).to.equal(false)` e salvarmos o arquivo, é possível observar que o navegador já ira se adequar às mudanças no arquivo de teste e consequentemente o teste irá falhar.

# Tarefa Prática #2: Testando a micro-livraria

Vamos agora implementar um teste end-to-end para a micro-livraria. Esse teste vai "simular" um usuário o usando o sistema da seguinte forma:

1. Abrir o site. 
2. Escolher o livro desejado.
3. Inserir o CEP.
4. Calcular o frete.
5. Realizar a compra do livro.

#### Passo 1:

Comece criando um arquivo `meu_teste_end_to_end.js` também na pasta `integration`, com o seguinte código inicial:

```
describe('Teste End-to-End', () => {
    it('Meu Primeiro Teste', () =>{
        // Visitar o viste
        cy.visit('http://localhost:5000/')
    })
  })
```

Os comandos do Cypress são constituídos pelo prefixo cy seguidos da função desejada. A função `visit()` visita uma página, que no caso da nossa micro livraria, que está sendo executada localmente, está no endereço `localhost:5000`. 

Ao salvar o arquivo vemos que o teste passou em `3`, e em `4` é exibida a página da micro livraria.

#### Passo 2:

Vamos agora acrescentar novos comportamentos no teste. Especificamente, vamos supor um cenário no qual um usuário quer comprar o livro de Padrões de Projeto. Logo, precisamos garantir que o livro está sendo mostrado na nossa página, do seguinte modo: 

```
describe('Teste End-to-End', () => {
    it('Meu Primeiro Teste', () =>{
        // Visitar o viste
        cy.visit('http://localhost:5000/')
        
        // Escolhe o livro desejado
        cy.get('[data-id=3]').should('contain.text', 'Design Patterns')
    })
  })
```
        
No código anterior, realizamos uma query usando a função `get`. Mas já sabemmos que:

* O catálogo de livros mostrado na página está disposto em três colunas. 
* O livro desejado está na terceira linha, cujo identificador é `data-id=3`. 

Por isso, usamos uma asserção que verifica se a terceira coluna inclui a string `Design Patterns`. 

Ao passar o mouse em cima de cada etapa do teste em `3` podemos observar que `4` muda, refletindo cada passo do teste. Em específico, o último passo (com a asserção) é mostrado em destaque, para indicar que ele foi corretamente identificada.

#### Passo 3:

Vamos agora incrementar de novo teste, para simular um usuário que insere o CEP no campo indicado; e clica no botão `Calcular Frete`. 

Como existem três instâncias para calcular o frete, podemos utilizar a função `within()` para selecionar a coluna correta, conforme o código a seguir:

```
describe('Teste End-to-End', () => {
    it('Meu Primeiro Teste', () =>{
        // Abre o viste
        cy.visit('http://localhost:5000/')
        
        // Escolhe o livro desejado
        cy.get('[data-id=3]').should('contain.text', 'Design Patterns')
        
        // Verifica o frete
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

        
Primeiro, o teste agora busca pela terceira coluna e procura pelo campo de `<input>`. Em seguida, ele insere o CEP `10000-000` e pressiona o botão `Calcular Frete`.  Prosseguindo, espera-se 2 segundos na função `wait()`, para garantir que a janela com o valor do frete vai carregar corretamente. 

Então, nessa janela, selecionamos o `swal-text` e fazemos uma asserção para garantir que a mensagem é aquele que esperamos. 

Por fim, clicamos no botão para fechar o pop-up.

## Tarefa Prática #2: Testando a Compra de um Livro

Modifique o teste anterior, acrescentando código para simular a compra de um livro. Basicamente, você deverá:

* Usar a função `cy.contains` para selecionar o botão Comprar e para clicar nele (função `click`)
* Esperar que o pop-up seja exibido com a confirmação da compra (função `wait`)
* Verificar se nesse pop-up temos a messagem: `Sua compra foi realizada com sucesso`
* Fechar o pop-up, clicando em seu botão

## Créditos

Este roteiro foi elaborado por **Rodrigo Moreira**, aluno de mestrado do DCC/UFMG, como parte das suas atividades na disciplina Estágio em Docência, cursada em 2021/1, sob orientação do **Prof. Marco Tulio Valente**.

==== PAREI DE REVISAR AQUI

Por último, para realizar a compra do livro, devemos pressionar o botão `Comprar` conforme o código abaixo:

```
\\ Realizar a compra
cy.get('[data-id=3]').within(() => {
    cy.contains('Comprar').click()
    cy.wait(2000)
})
cy.get('.swal-text').contains('Sua compra foi realizada com sucesso')
cy.get('.swal-button').click()
```
        
De forma análoga ao cálculo do frete, localizamos o botão `Comprar` na terceira coluna. Também utilizamos a função `wait()` para mais uma vez garantir que o modal carregue corretamente. Dentro do modal selecionamos o `swal-text` novamente e fazemos uma asserção com relação ao conteúdo, que dessa vez deve conter a mensagem de que a compra foi realizada com sucesso. Por fim, clicamos no botão dentro do modal para fechar o pop-up.
