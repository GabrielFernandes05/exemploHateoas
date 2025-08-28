# Guia para Implementação de HATEOAS na BookStoreAPI

Este guia foi elaborado para auxiliar na resolução da tarefa proposta, abordando os quatro pontos solicitados no documento e fornecendo recursos para aprofundamento.
Em breve adicionar aplicação em springboot funcional ao repositório

---

## 🧩 Contexto do Problema

A **BookStoreAPI** precisa evoluir de um modelo onde os clientes precisam conhecer previamente todas as URLs para um modelo que utilize **HATEOAS**. O objetivo é:

* Aumentar a flexibilidade
* Facilitar a descoberta de recursos
* Reduzir o acoplamento entre cliente e servidor

---

## ✅ Passo 1: Definir os Recursos e as Ações

### Recursos principais:

* **Livro:** Um item individual do catálogo
* **Categoria:** Um agrupamento de livros
* **Carrinho de Compras:** Onde o usuário adiciona os livros que deseja comprar

### Ações por recurso:

#### A partir de um **Livro**:

* Ver os detalhes do próprio livro
* Adicionar o livro ao carrinho
* Ver as avaliações do livro
* Ver outros livros da mesma categoria

#### A partir de uma **Categoria**:

* Ver os detalhes da própria categoria
* Listar todos os livros daquela categoria

#### A partir do **Carrinho de Compras**:

* Ver o conteúdo do carrinho
* Finalizar a compra (checkout)
* Remover um item
* Esvaziar o carrinho

---

## 📦 Passo 2: Criar Exemplos de Respostas JSON com HATEOAS

### 🔹 Exemplo 1: Resposta para um Livro (`GET /livros/10`)

```json
{
  "id": 10,
  "titulo": "O Guia do Mochileiro das Galáxias",
  "autor": "Douglas Adams",
  "preco": 39.90,
  "_links": {
    "self": {
      "href": "/livros/10",
      "method": "GET"
    },
    "adicionar-carrinho": {
      "href": "/carrinho",
      "method": "POST"
    },
    "avaliacoes": {
      "href": "/livros/10/avaliacoes",
      "method": "GET"
    },
    "categoria": {
      "href": "/categorias/3",
      "method": "GET"
    }
  }
}
```

### 🔹 Exemplo 2: Lista de Livros em uma Categoria (`GET /categorias/3/livros`)

```json
{
  "_links": {
    "self": {
      "href": "/categorias/3/livros",
      "method": "GET"
    },
    "categoria-info": {
      "href": "/categorias/3",
      "method": "GET"
    }
  },
  "total_livros": 1,
  "_embedded": {
    "livros": [
      {
        "id": 10,
        "titulo": "O Guia do Mochileiro das Galáxias",
        "autor": "Douglas Adams",
        "preco": 39.90,
        "_links": {
          "self": {
            "href": "/livros/10",
            "method": "GET"
          },
          "adicionar-carrinho": {
            "href": "/carrinho",
            "method": "POST"
          }
        }
      }
    ]
  }
}
```

### 🔹 Exemplo 3: Resposta para o Carrinho de Compras (`GET /carrinho/user-123`)

```json
{
  "id": "user-123",
  "total": 39.90,
  "itens": [
    {
      "livroId": 10,
      "titulo": "O Guia do Mochileiro das Galáxias",
      "quantidade": 1
    }
  ],
  "_links": {
    "self": {
      "href": "/carrinho/user-123",
      "method": "GET"
    },
    "finalizar-compra": {
      "href": "/pedidos",
      "method": "POST"
    },
    "esvaziar-carrinho": {
      "href": "/carrinho/user-123",
      "method": "DELETE"
    }
  }
}
```

---

## 💡 Passo 3: Justificar os Benefícios do HATEOAS

### 🔸 Reduz dependência de documentação externa

O cliente não precisa mais de uma documentação dizendo que para adicionar um item ao carrinho é preciso fazer `POST /carrinho`. A própria resposta do recurso já traz essa informação.

### 🔸 Facilita mudanças futuras

Caso o endpoint mude (ex: `/meu-carrinho`), clientes que seguem os links continuam funcionando sem ajustes.

Exemplo:

```json
"adicionar-lista-desejos": {
  "href": "/lista-desejos",
  "method": "POST"
}
```

### 🔸 Melhora a experiência do desenvolvedor cliente

Desenvolvedores podem criar aplicações mais resilientes, sem depender de URLs fixas. A aplicação navega pela API com base nas relações entre os recursos.

---

## ⚠️ Passo 4: Desafios Técnicos e Estratégias

### 📍 Aumento da complexidade e tamanho das respostas

* **Desafio:** JSONs mais pesados
* **Solução:** Usar compressão (Gzip) e incluir apenas links relevantes

### 📍 Falta de suporte em bibliotecas clientes

* **Desafio:** Bibliotecas não interpretam links
* **Solução:** Fornecer SDKs ou usar bibliotecas que entendem hipermídia (ex: HAL)

### 📍 Disciplina no design da API

* **Desafio:** Garantir consistência e relevância nos links
* **Solução:** Usar frameworks como Spring HATEOAS + seguir especificações como HAL

---

## 🧪 Exemplo em Java com Spring Boot + Spring HATEOAS

### 1. Adicionar a dependência no `pom.xml` (Maven)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
```

### 2. Criar a classe de representação do Recurso (`LivroModel`)

```java
package com.bookstore.model;

import org.springframework.hateoas.RepresentationModel;

public class LivroModel extends RepresentationModel<LivroModel> {
    private Long id;
    private String titulo;
    private String autor;
    private Double preco;

    // Construtores, Getters e Setters
}
```

### 3. Criar o Controller (`LivroController`)

```java
package com.bookstore.controller;

import com.bookstore.model.LivroModel;
import org.springframework.hateoas.Link;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import static org.springframework.hateoas.server.mvc.WebMvcLinkBuilder.*;

@RestController
@RequestMapping("/livros")
public class LivroController {

    @GetMapping("/{id}")
    public ResponseEntity<LivroModel> buscarLivroPorId(@PathVariable Long id) {
        LivroModel livro = new LivroModel();
        livro.setId(id);
        livro.setTitulo("O Guia do Mochileiro das Galáxias");
        livro.setAutor("Douglas Adams");
        livro.setPreco(39.90);
        
        // Links HATEOAS
        livro.add(linkTo(methodOn(LivroController.class).buscarLivroPorId(id)).withSelfRel());

        livro.add(linkTo(methodOn(CarrinhoController.class)
                .adicionarItem(null))
                .withRel("adicionar-carrinho")
                .withType("POST"));

        livro.add(linkTo(methodOn(AvaliacaoController.class)
                .listarPorLivro(id))
                .withRel("avaliacoes")
                .withType("GET"));
        
        return ResponseEntity.ok(livro);
    }
}
```

---

## 📚 Links de Apoio e Aprofundamento

* 📖 [Baeldung: Spring HATEOAS Tutorial](https://www.baeldung.com/spring-hateoas-tutorial)
* 📘 [Documentação Oficial do Spring HATEOAS](https://docs.spring.io/spring-hateoas/docs/current/reference/html/)
* 🎥 [Vídeo: REST APIs e HATEOAS (em português)](https://www.youtube.com/watch?v=F2SZS1P_wA4)
* 📝 [Artigo Caelum: Entendendo HATEOAS](https://blog.caelum.com.br/rest-e-hateoas-o-caminho-dos-links/)
* 📑 [JSON\:API - Especificação](https://jsonapi.org/)

---

## ✍️ Conclusão

A adoção de HATEOAS na BookStoreAPI proporciona maior flexibilidade, facilita evolução sem quebrar clientes, e torna a API mais robusta e fácil de usar. Com boas práticas, ferramentas adequadas e um design cuidadoso, é possível superar os desafios e colher os benefícios dessa abordagem.

---
