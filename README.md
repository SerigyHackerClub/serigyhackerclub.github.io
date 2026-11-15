# Serigy Hacker Club (Jekyll)

## Rodar no seu computador
    bundle install
    bundle exec jekyll serve      # abre em http://localhost:4000

## Novo post
Crie `_posts/AAAA-MM-DD-nome-do-post.md`:

    ---
    title: "Título do post"
    date: 2026-11-20 10:00:00 -0300
    description: "Uma frase que resume o post."
    tags: [web, pentest]
    ---

    Texto em Markdown (não repita o título aqui, o layout já mostra).

O endereço fica `/posts/nome-do-post.html`. Posts com data no futuro não aparecem.

## Guia
Cada página é um `.md` em `guia/`. A ordem do menu e dos botões anterior/próximo
está em `_data/guia.yml`.

## CSS
Ao mudar o `style.css`, aumente `css_version` no `_config.yml`.
