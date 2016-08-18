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

## Técnicas de CSS

### Bootstrap (CSS prototipádo)

O Bootstrap foi desenvolvido no twitter como uma solução interna para padronizar o CSS. Mais tarde, o código foi aberto e em 2011 teve os seu primeiro release — que fez um sucesso instantâneo.

Esse framework trouxe diversas vantagens para os desenvolvedores, principalmente para aqueles que não se sentiam a vontade para escrever CSS.

Com o Bootstrap embutido no seu html, para estilizar um elemento  basta adicionar classes:

```html
<button class="btn btn-default btn-large">submit<button>
```

### BEM (Block__Element--Modifier)

O BEM, diferente de um framework é uma metodologia de escrever CSS. E essa metodologia é baseada numa nomenclatura especifica para as classes dos elementos. São elas:

#### Bloco (block):

É uma entidade independente que tem significado próprio. 

Exemplos: `header`, `container`, `menu`.

#### Elemento (element):

Partes de um bloco que não têm nenhum significado independente. Eles são semanticamente ligados ao seu bloco. A convenção pede que se escreva o elemento depois do bloco com dois traços inferiores.

Exemplos: `list__item`, `header__title`, `menu__button`.

#### Modificador (modifier):

Modificam blocos ou elementos e servem para mudar aparência ou comportamento. A convenção pede que se escreva o modificador depois do bloco ou elemento com dois hífens.

Exemplos: `container--flex`, `list__link--highlighted`, `form__checkbox--checked`.

### JSS (JavaScript Style Sheets)

### CSS Modules (CSS com escopo local)


## O problema do escopo global

Um dos grandes problemas que os desenvolvedores encontram quando estão escrevendo o estilo de uma aplicação é a inexistência de escopo no CSS.





