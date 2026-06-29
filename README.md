# 📚 Estante — Explorador de Livros

Projeto da disciplina de Desenvolvimento Web — consumo de API pública com `fetch`, interface responsiva com Tailwind CSS e documentação de decisões técnicas.

---

## Tecnologias utilizadas

| Tecnologia | Justificativa |
|---|---|
| HTML5 semântico | Estrutura com `<header>`, `<main>`, `<article>`, `<footer>` |
| Tailwind CSS (CDN) | Estilização responsiva sem necessidade de build |
| JavaScript puro (ES2020) | `fetch`, `async/await`, manipulação de DOM, eventos |
| Open Library API | Pública, gratuita, sem autenticação, CORS habilitado |

---

## API escolhida: Open Library

**Base URL:** `https://openlibrary.org`  
**Documentação:** https://openlibrary.org/developers/api

### Por que a Open Library?

- Pública e gratuita — sem cadastro ou API key
- CORS habilitado — funciona direto no navegador
- Dados ricos: título, autores, ano, assuntos, capas
- Mais de 20 milhões de livros no acervo

### Endpoints utilizados

```
GET /search.json?q={busca}&limit=20&offset={n}
  → Lista paginada de livros

GET /works/{key}.json
  → Detalhes de um livro (descrição, assuntos, datas)
```

---

## Conceitos do curso aplicados

### Módulo 05 — JavaScript Fundamentals

| Tópico do módulo | Onde foi usado no projeto |
|---|---|
| Variáveis com `const` e `let` | `BASE_URL`, `buscaAtual`, `paginaAtual` |
| Funções | `buscarJSON`, `criarCardHTML`, `mostrarEstado`, `abrirModal` |
| Template literals | Montagem do HTML dos cards e do modal |
| Arrays com `.map()`, `.slice()`, `.join()` | Transformar lista de livros em HTML; limitar autores e assuntos |
| Objetos e desestruturação | Leitura dos campos `livro.title`, `livro.author_name` etc |
| DOM com `querySelector` e `getElementById` | Buscar e atualizar elementos da página |
| Eventos com `addEventListener` | Clique no botão de busca, Enter no campo, clique nos cards |
| JSON com `JSON.parse` / `.json()` | Conversão da resposta da API em objeto JavaScript |

### Módulo 06 — Assincronismo e Fetch

| Tópico do módulo | Onde foi usado no projeto |
|---|---|
| `async/await` | Funções `buscarLivros` e `abrirModal` |
| `fetch` | Requisições para a Open Library API |
| `if (!resposta.ok)` | Verificação do status HTTP antes de ler o JSON |
| `try/catch` | Tratamento de erros em todas as chamadas assíncronas |
| Fetch em duas etapas | Lista de livros primeiro, detalhes só ao clicar (lazy) |

---

## Decisões técnicas

### 1. Fetch em duas etapas 

Os detalhes completos de um livro (descrição, assuntos) só são buscados quando o usuário abre o modal. Isso evita fazer 20 requisições extras só para montar o grid.

```javascript
// Na lista: busca apenas o necessário para o card
GET /search.json?fields=key,title,author_name,cover_i,...

// No modal: busca os detalhes completos sob demanda
GET /works/{key}.json
```

### 2. Paginação server-side

A API suporta `limit` e `offset`, então a paginação acontece no servidor — o cliente só recebe os 20 livros da página atual.

```javascript
const offset = (pagina - 1) * POR_PAGINA
// página 1 → offset 0
// página 2 → offset 20
// página 3 → offset 40
```

### 3. Fallback para livros sem capa

Muitos livros não têm imagem cadastrada. Nesses casos, o card exibe o título do livro sobre um fundo colorido.

```javascript
const imagemHTML = capId
  ? `<img src="${COVERS_URL}/${capId}-M.jpg" .../>`
  : `<div class="bg-shelf ..."><p>${titulo}</p></div>`
```

### 4. Descrição em dois formatos

A Open Library retorna a descrição de formas diferentes dependendo da obra. O código trata os dois casos:

```javascript
let descricao = dados.description
if (typeof descricao === 'object') {
  descricao = descricao.value  // formato legado
}
```

---

## Limitações conhecidas

| Limitação | Causa | Possível solução |
|---|---|---|
| Busca exige termos próximos do real | A API não faz busca fuzzy | Tentar variações do nome |
| ~40% dos livros sem descrição | Dados incompletos no acervo | Complementar com Google Books API |
| Rate limit não documentado | Política da Open Library | Não recomendado para uso em produção |
| Tailwind via CDN (~300 KB) | Carrega o framework inteiro | Em produção: usar Tailwind CLI com purge |

---

## Estrutura do projeto

```
/
├── index.html   # Toda a aplicação em um único arquivo
└── README.md    # Documentação técnica
```

---
