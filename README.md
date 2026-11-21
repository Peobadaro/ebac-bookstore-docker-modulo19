# Bookstore

Projeto Bookstore do curso **Profissão: Desenvolvedor Full Stack Python** da EBAC (Django + Poetry).

Autor: **Pedro Badaró Ferreira** ([@Peobadaro](https://github.com/Peobadaro))

## Como rodar

```bash
poetry install
poetry run python manage.py migrate
poetry run python manage.py runserver
```

## Rodar os testes

```bash
poetry run python manage.py test
```

## Estrutura

| App | Modelos | Serializers | Testes |
|---|---|---|---|
| `product` | `Category`, `Product` | `CategorySerializer`, `ProductSerializer` | `product/tests/` |
| `order` | `Order` | `OrderSerializer` | `order/tests/` |

- **Product → Category (N:N):** na leitura, as categorias aparecem completas dentro do produto (`category`). Na escrita, são informadas pelos ids (`categories_id`).
- **Order → Product (N:N) e Order → User:** na leitura, os produtos aparecem completos (com as categorias) e o pedido mostra o `total`. Na escrita, os produtos são informados pelos ids (`products_id`).

## API (ViewSets)

Os ViewSets usam `ModelViewSet`, então cada rota tem listar, criar, detalhar, atualizar e apagar.

| Rota | ViewSet |
|---|---|
| `/bookstore/v1/category/` e `/bookstore/v1/category/<id>/` | `CategoryViewSet` |
| `/bookstore/v1/product/` e `/bookstore/v1/product/<id>/` | `ProductViewSet` |
| `/bookstore/v1/order/` e `/bookstore/v1/order/<id>/` | `OrderViewSet` |

As rotas ficam em `product/urls.py` e `order/urls.py` e são incluídas em `bookstore/urls.py`. Para conferir todas:

```bash
poetry run python manage.py show_urls
```

## Paginação

A paginação do DRF está ligada para todos os ViewSets em `bookstore/settings.py`:

```python
REST_FRAMEWORK = {
    "DEFAULT_PAGINATION_CLASS": "rest_framework.pagination.PageNumberPagination",
    "PAGE_SIZE": 5,
}
```

As listas voltam com `count`, `next`, `previous` e `results` (5 itens por página). Para ver outra página: `/bookstore/v1/product/?page=2`.

## Django Debug Toolbar

Instalado como dependência de desenvolvimento. Com `DEBUG = True`, a barra aparece ao abrir a API no navegador em `127.0.0.1` e mostra quantas queries SQL cada requisição fez, o tempo, o cache e outros painéis. Ela fica desligada durante os testes.

## Autenticação por Token

- **Produtos e categorias** continuam abertos: qualquer pessoa pode ver a lista e os detalhes.
- **Pedidos** (`OrderViewSet`) exigem login com `TokenAuthentication` + `IsAuthenticated`. Cada usuário só vê, altera e apaga os próprios pedidos, e um pedido novo sempre fica no nome de quem está autenticado.

Gerar o token do usuário:

```bash
curl -X POST -d "username=SEU_USUARIO&password=SUA_SENHA" http://127.0.0.1:8000/api-token-auth/
# {"token": "..."}
```

Usar o token nas chamadas de pedido:

```bash
curl -H "Authorization: Token SEU_TOKEN" http://127.0.0.1:8000/bookstore/v1/order/
```

Sem token a API responde **401**.
