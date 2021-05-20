---
title:  "O poder do react-query e em que ele pode me ajudar"
date:   2020-05-20 15:04:23
categories: [react]
tags: [react, react-query]
---

![React Query](https://react-query.tanstack.com/\_next/static/images/logo-a65f848e05592e7de1dc2150454230fa.svg)

  

## O poder do react-query e em que ele pode me ajudar.

O react-query é uma biblioteca para fazer fetching em APIs com um gerenciamento de estado seguindo uma metodologia diferente do que estamos habituados a ver que é a famigerada **arquitetura flux**

O grande problema da arquitetura flux é que ela acaba criando uma cópia do estado do server-side no client-side e o frontend acaba manipulando esta cópia criada o que pode acarretar em problemas se o client-side não refletir em algum momento corretamente esse estado para o server-side.

Com isso, o react-query além de resolver este problema, consegue salvar em cache