# Proposta de migração para CSS Modules

## Nosso CSS atualmente

### Nossa estrutura

Dentro da estrutura do nosso projeto passamos a maior parte do tempo trabalhando na pasta `/src`.

```
/src
├─ /fonts
├─ /images
├─ /js
└─ /styles
  ├─ /components
  ├─ /misc
  ├─ /pages
  └─ main.css
```

Essa é uma estrutura de diretórios clássica e ela denuncia algumas práticas e vícios que nós temos em relação as ferramentas nós usamos.

### Task runners

Há pouco tempo atrás a ferramenta certa pra compilar os nossos ativos estáticos era o Gulp. Um `gulpfile.js` (o arquivo de configuração) se parece com o script abaixo. Olhando atentamente se percebe que todos esses ativos são tratados como entidades separadas, ignorando a relação que eles tem entre si.

```js
// Estilos
gulp.src('styles', function () {
    return gulp.src('./diretorio/de/css')
        .pipe(sass())
        .pipe(gulp.dest('./destino'))
})

// Scripts
gulp.src('scripts', function () {
    return gulp.src('./diretorio/de/js')
        .pipe(eslint())
        .pipe(concat())
        .pipe(uglify())
        .pipe(gulp.dest('./destino'))
})
```


### Module Bundlers

O Webpack funciona de maneira diferente. Nele os nossos ativos estáticos são tratados como módulos e cada tipo de módulo passa por uma ‘transformação’ com um `loader` apropriado. Diferente de um task runner, compila esses ativos em um único processo e cada `require()` é feito sob demanda. Uma boa maneira de perceber isso é lembrar que dar um `require()` em qualquer arquivo que não tenha uma extensão `.js` não é suportado nativamente.

Então o que estamos fazendo de errado? Nós herdamos o nosso hábito de estruturar nosso código de task runners como o Gulp, estamos importando o `main.css` no nosso ponto de entrada e não damos nenhum tratamento diferenciado para os nossos componentes — nós tratamos o nosso CSS como uma entidade separada e não estamos nos aproveitando do potencial que o Webpack tem a oferecer.

## Precedencia de seletores e o problema do escopo global

Tomando por exemplo este pedaço de código vamos aplicar um estilo e ver como o CSS se comporta.

```html
<div id="pai">
    <span class="filho">olá mundo</span>
</div>
```

## Técnicas de CSS

### Bootstrap (CSS prototipádo)

### BEM (Block__Element-\-Modifier)

### JSS (JavaScript Style Sheets)

### CSS Modules (CSS com escopo local)


