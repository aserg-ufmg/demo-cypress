# Roteiro Prático sobre Testes End-to-End usando Cypress

O objetivo deste roteiro é ter um primeiro contato com testes do tipo end-to-end. Esses testes são chamados também de testes de front-end, testes de sistemas,  testes de interface Web ou testes de interface com o usuário.

No roteiro, vamos usar uma ferramenta open source para testes end-to-end chamada [Cypress](https://www.cypress.io). O Cypress é parecido com o Selenium (para o qual já foi mostrado um exemplo de teste no [Capítulo 8](https://engsoftmoderna.info/cap8.html) do livro Engenharia de Software Moderna).

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

E em seguida:

```
npx cypress open
```

Após a instalação do Cypress no diretório do projeto, será criada uma pasta `cypress` contento diversas pastas e arquivos. Especificamente, a pasta `integration`, possui diversos exemplos de testes.

**Passo 6:** Execute o Cypress, utilizando o comando `npx cypress open`. Será então exibida a seguinte tela. Na área marcada com `1` temos os testes criados para o sistema e a marcação `2` consiste em um botão para criação de um novo arquivo de testes.

![Figura 1](https://user-images.githubusercontent.com/54295278/124540444-c8c4d180-ddf5-11eb-8573-39fff6437d44.PNG)

# Tarefa Prática #1: Primeiro Teste


Para criação do nosso primeiro teste, iremos criar um novo arquivo chamado `meu_teste.js` dentro da pasta `integration`. Os arquivos de testes do Cypress são constituídos de uma sequência de funções que podem ser encadeadas com outras funções para testar funcionalidades do front-end da aplicação. Como primeiro teste, iremos apenas observar o resultado de uma simples função de asserção no serviço de Dashboard da ferramenta. No arquivo `meu_teste.js` cole o seguinte código e salve o arquivo:

```
describe('Meu primeiro teste', () => {
    it('Não faz nada', () => {
      expect(true).to.equal(true)
    })
  })
```

O teste acima apenas checa se `true` é igual a `true`. Após salvar o arquivo, procure por ele na lista de arquivos de teste na aplicação do Cypress e clique duas vezes no arquivo `meu_teste.js`. Em seguida, será executado o arquivo de testes e os resultados apresentados conforme a figura abaixo. A área `3` apresenta os resultados dos testes, enquanto `4` apresenta os snapshots obtidos ao longo da execução de cada passo dos testes. Para o nosso simples teste foi apenas constatado que true é igual a true, então todos os testes foram executados com sucesso.

![Figura 2](https://user-images.githubusercontent.com/54295278/124540502-e4c87300-ddf5-11eb-98db-ec8b6e5fdd18.PNG)

De forma análoga, se alterarmos a linha `3` para `expect(true).to.equal(false)` e salvarmos o arquivo, é possível observar que o navegador já ira se adequar às mudanças no arquivo de teste e consequentemente o teste irá falhar, uma vez que true não é igual a false.

# Tarefa Prática #2: Criação de teste end-to-end

Na segunda tarefa iremos simular um teste end-to-end da nosso sistema. Como mencionado anteriormente, iremos simular a utilização de um usuário constituída pelos seguintes passos:

1 - Abrir o site. 

2 - Escolher o livro desejado.

3 - Inserir o CEP.

4 - Calcular o frete.

5 - Realizar a compra do livro.

Para isso, iremos criar um novo arquivo `meu_teste_end_to_end.js` também na pasta `integration`. Comece inserindo o seguinte código no arquivo para realização do primeiro passo:

```
describe('Teste End-to-End', () => {
    it('Meu Primeiro Teste', () =>{
        // Visitar o viste
        cy.visit('http://localhost:5000/')
    })
  })
```

Os comandos do Cypress são constituídos pelo prefixo cy seguidos da função desejada. A função `visit()` visita uma página, que no caso da nossa micro livraria que está sendo executada localmente, possui endereço `localhost:5000`. Ao salvar o arquivo vemos que o teste passou em `3`, e em `4` é exibida a página da micro livraria.

Para o segundo passo iremos supor que o livro desejado é o livro sobre Padrões de Projeto da Gangue dos Quatro (Design Patterns de Gamma et al.). Logo, precisamos garantir que o livro está presente na nossa página. Iremos então pesquisar pela existência dele na página com o seguinte código:

```
// Escolher o livro desejado
cy.get('[data-id=3]').should('contain.text', 'Design Patterns')
```
        
No código anterior realizamos uma query com o auxílio da função `get`. O nosso catálogo de livros está disposto em três colunas, e o livro desejado está na terceira linha, cujo identificador é o `data-id=3`. Em seguida, realizamos uma asserção sobre essa coluna afirmando que ela deve conter o texto `Design Patterns` que é o nome do livro. 

Ao passar o mouse em cima de cada etapa do teste em `3` é possível observar que `4` muda, refletindo cada etapa de teste. Em específico, o último passo onde procuramos pela terceira coluna contendo o livro desejado é apresentada em destaque, mostrando que foi corretamente identificada.

No terceiro passo, precisamos inserir o CEP no campo indicado e clicar no botão `Calcular Frete`. Como existem três instâncias para calcular o frete, podemos utilizar a função `within()` para selecionar a coluna correta, conforme o código a seguir:

```
\\ Calcular o frete
cy.get('[data-id=3]').within(() => {
    cy.get('input').type('10000-000')
    cy.contains('Calcular Frete').click().then
    cy.wait(2000)
})
cy.get('.swal-text').contains('O frete é: R$')
// Calcular o frete
cy.get('.swal-button').click()
```
        
Buscamos pela terceira coluna e procuramos pelo campo de `<input>`. Inserimos o CEP `10000-000` e pressionamos o botão `Calcular Frete`. Em seguida esperamos 2 segundos com auxílio da função `wait()` para garantir que o modal carregue corretamente. Dentro do modal selecionamos o `swal-text` e fazemos uma asserção para garantir que a mensagem está correta. Por fim clicamos no botão dentro do modal para fechar o pop-up.

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
