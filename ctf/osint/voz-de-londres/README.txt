# Voz de Londres — redesign para Jekyll

Arquivos HTML mantêm front matter Jekyll e usam `relative_url`.

## Instalação
1. Copie os HTMLs para as rotas correspondentes do desafio:
   - `index.html` -> `/ctf/osint/voz-de-londres/`
   - `u/v_oficial.html`
   - `u/eng_sombra.html`
   - `forum/4471.html`
2. Copie `assets/voz-de-londres.css` para o diretório de assets do site.
3. Inclua o stylesheet no layout Jekyll global (`_layouts/default.html`) **uma vez**:
   `<link rel="stylesheet" href="{{ '/ctf/osint/voz-de-londres/assets/voz-de-londres.css' | relative_url }}">`
4. Confirme que `mascara.jpg` continua disponível em:
   `/ctf/osint/voz-de-londres/assets/mascara.jpg`

CSS é todo prefixado `vl-` e não interfere nos estilos globais.
