---
title:  "Typescript com styles-components"
date:   2020-05-21 15:04:23
categories: [react]
tags: [react, styled-components, basic, typescript]
---
Após conhecer um pouco do styled-components, não há algo mais bacana do que fazer nossos próprios componentes no nosso próprio estilo, vamos falar um pouco de como utilizar typescript com styled-components. Esse é um post bem básico pra quem não está familiarizado com ambas tecnologias. 

![Styled](https://media.giphy.com/media/14o3gppq0mt2Lu/giphy.gif)

Falar um pouco das funcionalidades do **styled-cmponents** talvez seja legal para um futuro próximo post, mas não podemos deixar de abordar algumas coisas por aqui. 

**A partir de agora vou postar os códigos em gists, já que o markdown do tema que to usando o jekyll não funciona mt bem com JSX 💩**

### Intro 
O styled-components após descoberto se torna algo fascinante, desenvolver seu componente estilizado, misturar JS com CSS, herdar estilos de outro componente sobrescrevê-los da forma que acharmos melhor e passar props para esses componentes é de fato algo poderoso, mas e se estivermos trabalhando com typescript, como podemos passar uma prop que não é esperada
