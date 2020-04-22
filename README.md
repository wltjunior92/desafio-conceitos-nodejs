<img alt="GoStack" src="https://i.imgur.com/2l1FtY3.png" />

<h3 align="center">
  Desafio 01: Conceitos NodeJS
</h3>

<p align="center">
  <img alt="GitHub language count" src="https://img.shields.io/github/languages/count/wltjunior92/desafio-conceitos-nodejs?color=%2304D361">

  <a href="https://github.com/wltjunior92/desafio-conceitos-nodejs/stargazers">
    <img alt="Stargazers" src="https://img.shields.io/github/stars/wltjunior92/desafio-conceitos-nodejs?style=social">
  </a>
</p>

## 😉 Primeiras impressões

Nas aulas que precediam esse desafio fomos expostos aos conceitos mais básicos e fundamentais do <strong>NodeJS</strong> e, embora tenha sido disponibilizado um template pré configurado para iniciar o desafio, tive a certeza de que não fez falta nesse primeiro projeto, já que esse tipo de configuração será feita ainda milhares de vezes durante o Bootcamp.

O desafio consiste em construir uma API onde poderão ser feitos **cadastros** e **listagem** de repositórios e esses mesmos poderão receber **likes**.

---

### 🧩 Passando nos testes

*Como ainda não fomos introduzidos à comunicação da API com um banco de dados, os dados eram armazenados em um array.*

#### *Should be able to create a new repository*

Como não haviam regras de negócio específicas a serem trabalhadas, foi suficiente receber os dados vindos do corpo da requisição, criar o objeto `repository` e depois inserí-lo  no array `repositories` que estamos usando para armazenar os dados.

```js
app.post("/repositories", (request, response) => {
  const { title, url, techs } = request.body;

  const repository = {
    id: uuid(),
    title,
    url,
    techs,
    likes: 0
  };

  repositories.push(repository);

  return response.status(201).json(repository);
});
```

Lembrando também que é essencial para passar no teste que o numero de likes seja iniciado com **0**.

#### *Should be able to list the repositories*

Para passar esse teste basta retornar `repositories` como `json` no `response` da método.

```js
app.get("/repositories", (request, response) => {
  return response.json(repositories);
});
```
#### *Should be able to update a repository*

Nesta etapa, para que a rota para atualizar um repositório funcionasse corretamente, eu recebi o `id` do repositório que queria atualizar através dos parâmetros da requisição e os valores que desejava atualizar através do corpo da requisição.

```js
app.put("/repositories/:id", (request, response) => {
  const { id } = request.params;
  const { title, url, techs } = request.body;

  const repositoryIndex = repositories.findIndex(repository => repository.id === id);

  const { likes } = repositories.find(repository => repository.id === id);

  const repository = {
    id,
    title,
    url,
    techs,
    likes
  }

  repositories[repositoryIndex] = repository;

  return response.json(repository);
});
```

Para passar também no teste de não permitir que o numero de likes seja atualizado manualmente (***should not be able to update repository likes manually***), buscamos o repositório no array através do `id` e utilizando desestruturação, conseguimos isolar os likes que o repositorio em questão tinha antes da atualização.

#### *Should not be able to update a repository that does not exist*

Neste caso, basta que seja adicionada essa verificação no metodo atualiza o repository:

```js
  if (repositoryIndex < 0) {
    return response.status(400).json({ error: 'Repository not found' });
  }
```

#### *Should be able to delete the repository*

Para passar neste basta que o método consiga remover o repositório passado no parâmetro da requisição.

```js
app.delete("/repositories/:id", (req, res) => {
  const { id } = req.params;

  const repositoryIndex = repositories.findIndex(repository => repository.id === id);

  repositories.splice(repositoryIndex, 1);

  return res.status(204).send();
});
```

#### *Should not be able to delete a repository that does not exist*

A mesma validação utilizada na atualização pode ser usada nesse método:

```js
  if (repositoryIndex < 0) {
    return res.status(400).json({ error: 'Repository not found' });
  }
```

#### *Should be able to give a like to the repository*

Para passar nesse teste a aplicação precisa incrementar o numero de likes sempre que a rota for acessada.

Importante resaltar que o numero de likes deve ser incrementado em apenas uma unidade.

```js
app.post("/repositories/:id/like", (request, response) => {
  const { id } = request.params;

  const repositoryIndex = repositories.findIndex(repository => repository.id === id);

  if (repositoryIndex < 0) {
    return response.status(400).json({ error: 'Repository not found' });
  }

  const { title, url, techs, likes } = repositories.find(repository => repository.id === id);

  const incrementedLikes = likes + 1;

  const repository = {
    id,
    title,
    url,
    techs,
    likes: incrementedLikes
  }

  repositories[repositoryIndex] = repository;

  return response.json(repository);
});
```

Para passar no teste ***"should not be able to like a repository that does not exist"*** foi usada a mesma validação dos outros métodos que recebem um `id` nos parâmetros da requisição

<blockquote>O trecho de código que faz a validação do id foi aplicado da mesma forma em três métodos diferentes, logo poderia (<strong>DEVERIA</strong>) ter sido isolado em outra função!</blockquote>