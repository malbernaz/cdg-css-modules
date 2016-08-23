# Proposta de migração para CSS Modules

## Nossa estrutura de CSS atualmente

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

## O problema do escopo global e a precedência de seletores

Um dos grandes problemas que os desenvolvedores encontram quando estão escrevendo o estilo de uma aplicação é a inexistência de escopo no CSS. Então aplicar um estilo a um elemento qualquer pode ser mais complicado do que simplesmente mudar uma propriedade, ou adicionar uma classe.

Isso acontece por que os seletores de um estilo aplicado a uma página tem prioridade sobre outros. À essa característica do CSS  damos o nome de “precedência de seletores” ou “especificidade de seletores”.

Vamos tomar por exemplo este trecho de código abaixo:

```html
<style>
  #blue span { color: blue; }
  span.green { color: green; }
  span { color: red }
</style>

<div id="blue">
  <span class="green">olá mundo!</span>
</div>
```

Nesse documento existem três seletores “tentando” estilizar um mesmo elemento. Para entender qual deles tem especificidade maior vamos usar essa tabela:

| id | class | element |
|:--:|:-----:|:-------:|
| 0  | 0     | 0       |

Os seletores estão organizados da esquerda para a direita por ordem de força. Então `id` é mais forte que `class`, que por sua vez é mais forte que um elemento. E esses seletores se somam — por exemplo um seletor tal qual `span.green.fixed` tem duas classes aplicadas, portanto tem mais força que `span.green`. O mesmo vale para os IDs.

Então voltando ao primeiro exemplo teríamos — imaginando a tabela acima — ` 1 0 1` para `#blue span` que tomaria precedência, `0 1 1` para `span.green` e `0 0 1` para `span`.

**Não existe nenhum jeito de sobrescrever um id**? Existe. Na precedência de seletores duas regras sobrescrevem o id, uma é com usar a declaração `!important` logo após a regra a ser aplicada — o que é um bom indicador de que tem alguma coisa errada com o código da sua aplicação — e a outra é aplicar o atributo `style` ao elemento diretamente.

Essa ultima alternativa é a forma mais próxima do que podemos chamar de escopo local no CSS nativamente (mais sobre escopo local mais tarde).

Com essas regras em mente, fica fácil perceber por que é tão difícil de escrever um CSS escalável para uma aplicação. Principalmente em um time com vários desenvolvedores escrevendo o código ao mesmo tempo, fica muito difícil de controlar como esse estilo vai se comportar.

Vamos passar por algumas alternativas que tentam solucionar esse problema:

### Bootstrap (CSS prototipádo)

O Bootstrap foi desenvolvido no twitter como uma solução interna para padronizar o CSS. Mais tarde, o código foi aberto e em 2011 teve os seu primeiro release — que fez um sucesso instantâneo.

#### Prós:

- Fácil de usar, para estilizar um elemento basta adicionar classes:

```html
<button class="btn btn-default btn-large">submit<button>
```

- Não é necessário escrever uma linha de CSS para se usar

#### Contras:

- O Bootstrap é um framework inchado. Não é trivial, por exemplo, usar somente o sistema de grades ou os botões.
- Não é só CSS. Um `bootstrap.js` é necessário para se desfrutar de toda glória desse framework.
- Tem opinião sobre como os elementos devem se parecer. Sobrescrever um estilo não é trivial (se lembram das regras de especificidade?).

### BEM (Block__Element--Modifier)

O BEM, diferente de um framework é uma metodologia de escrever CSS. E essa metodologia é baseada numa nomenclatura especifica para as classes dos elementos:

#### Bloco (block):

É uma entidade independente que tem significado próprio.

Exemplos: `header`, `container`, `menu`.

#### Elemento (element):

Partes de um bloco que não têm nenhum significado independente. São semanticamente ligados ao seu bloco. A convenção pede que se escreva o elemento depois do bloco com dois traços inferiores.

Exemplos: `list__item`, `header__title`, `menu__button`.

#### Modificador (modifier):

Modificam blocos ou elementos e servem para mudar aparência ou comportamento. A convenção pede que se escreva o modificador depois do bloco ou elemento com dois hífens.

Exemplos: `container--flex`, `list__link--highlighted`, `form__checkbox--checked`.

#### Prós:

- Incentiva o desenvolvedor a adotar uma nomenclatura concisa.
- Incentiva o desenvolvedor a usar somente classes. O que é bom na precedência de seletores.

#### Contras:

- Extremamente verbal. Os nomes de classe podem ficar muito grandes.

### Estilos em `./js` e Inline Styles

Ironicamente, nos vemos hoje, principalmente no mundo React em uma posição em que estilos inline as vezes fazem mais sentido do que qualquer outra solução. Principalmente em aplicações de pequeno porte.

#### Prós:

- Fácil manutenção
- Escopo local

#### Contras:

- Não é performático em escala
- Não tem todos os recursos que o CSS tem (ex: media queries, animation)
- É `.js`...

#### Outras soluções em `./js`

Existem diversas bibliotecas de estilo `.js`. Varias delas com soluções interessantíssimas para os problemas aqui discutidos. Algumas se baseiam em inline Styles e outras não, e elas vem geralmente para resolver os seguintes problemas:

- Autoprefixing
- Pseudo classes
- Media queries


### CSS Modules (CSS com escopo local)

Chegamos então a solução que eu considero ser a mais elegante, os CSS Modules.

Um Módulo CSS é um arquivo CSS em que todos os nomes de classe e animação são escopados localmente por padrão. Todos os URLs `(url (...))` e `@imports` estão em formato de solicitação de módulos (`./xxx` e `../xxx` significa relativo,`xxx` e `xxx/yyy` significa pasta de módulos, por exemplo `node_modules`).

#### Nomenclatura

Para nomes de classes locais o camelCase é recomendado, mas não é obrigatório.

#### Excessões

Os identificadores `:global(.xxx)` e `@keyframes :global(XXX)` são usados para escopar respectivamente classes e animações globalmente. Se o selector está em modo global, o modo global também é ativado para as regras (isto permite fazer `animation: someAnimation;` local).

```css
.someLocalClass {
  color: red;
  animation: someAnimation; // Animação global
}

:global(.someGlobalClass) {
  color: green;
  animation: otherAnimation; // Animação local
}

@keyframes :global(someAnimation) {
  from: {...}
  to: {...}
}

@keyframes otherAnimation {
  from: {...}
  to: {...}
}
```


#### Composição

É possível compor seletores:

```css
.className {
  color: green;
  background: red;
}

.otherClassName {
  composes: className; // a mágica acontece nesta linha
  color: yellow;
}
```

#### Dependencias

isto também é possível:

```css
.className {
  composes: otherClassName from './style.css';
}
```

#### Uso com pré-processadores:

Com um pré-processador fica fácil de definir um bloco local ou global: 

```scss
:global {
  .globalClass {
    color: green;
  }
}
```

#### Prós:

- Menos conflitos
- Dependencias explicitas
- Fim do escopo global!

#### Contras:

- Exige alguma configuração. Necessita de um module bundler como o Webpack

## Conclusão

Acho que existem dois problemas que precisam ser resolvidos. Um é organizacional. Acho que o nosso projeto se beneficiaria de uma estrutura como essa:

```
/src
├─ /components
├─ /containers
  ├─ /Header
  └─ /Footer
    ├─ Footer.css
    └─ Footer.js
  └─ /Main
└─ /Theme
  ├─ /variables
  └─ /extends
  
```

em que cada componente seja encapsulado dentro do seu diretório, junto com a sua folha de estilos. Sendo por exemplo o `Footer.js` estruturado dessa maneira:

```js
import React, { Component } from 'react'
import s from './Footer.css'

class Footer extends Component {
  render () {
    return (
      <footer className={ s.root }>
        { ... }
      </footer>
    )
  }
}
```

O segundo problema é de boas práticas. Os CSS Modules são uma boa escolha na minha opinião pelo escopo local, mas também por que encorajam o desenvolvedor a usar seletores simples (sem aninhamento).

Como alcançar a escalabilidade? Dentro do nosso projeto vemos diversos padrões que se repetem no CSS, como elementos que tem uma semelhança grande e variáveis de cor. A melhor maneira de trabalhar esse problema é requerindo essas variáveis quando necessárias, por exemplo para um componente de botão:

```css
@import '../theme/vars/';
@import '../theme/extends';

.Button {
  composes: genericButton;
  color: var(--someColor)
}
```
