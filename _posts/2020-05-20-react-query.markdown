      

---
title:  "O poder do react-query e em que ele pode me ajudar"
date:   2020-05-20 15:04:23
categories: [react]
tags: [react, react-query, intermediary]
---
O react-query é uma biblioteca para fazer fetching em APIs com um gerenciamento de estado seguindo uma metodologia diferente do que estamos habituados a ver que é a famigerada **arquitetura flux**.

![React Query](https://react-query.tanstack.com/\_next/static/images/logo-a65f848e05592e7de1dc2150454230fa.svg)

  

## O poder do react-query e em que ele pode me ajudar.

O grande problema da arquitetura flux é que ela acaba criando uma cópia do estado do server-side no client-side e o frontend acaba manipulando esta cópia criada, o que pode acarretar em problemas se o client-side não refletir em algum momento corretamente esse estado para o server-side.

Com isso, o react-query além de resolver este problema, consegue salvar em cache nossas responses, quando precisamos do dado atualizado basta dar um **invalidate** que a query é recarregada automaticamente. Veremos isso mais pra frente para tornar mais claro. 

Como o intuito deste post é ter mais um **hands-on**, vamos por a mão na massa para tornar o post mais dinâmico 😊. 

#### Instalando o react query:
```
npm install react-query 
// ou 
yarn add react-query
```

#### Criando o provider:

A primeira coisa que precisamos é envolver nossa aplicação em um provider para que ela tenha acesso às funcionalidades do react-query. 
**Irei fazer um post falando a respeito de abstração de providers mais pra frente** por aqui mesmo. Isso vai possibilitar termos um provider genérico que abstrai a inclusão de um novo provider sempre que for fazer um teste unitário ou qualquer outra coisa que necessite de um provider com **QueryClient** diferente. 

````javascript
import {
	QueryClient
	QueryClientProvider
} from 'react-query';


function MyApp() {
	 const queryClient = new QueryClient();
	 return (
	 	<QueryClientProvider client={queryClient}>
			// Sua aplicação aqui
		</QueryClientProvider>
	 )
}

````

Nós conseguimos também definir algumas configurações para nosso QueryClient, é um pouco do que falamos de abstrair nosso provider em um provider genérico, mas isso é pra outro post 😅. Uma configuração que eu acho relevante é a ```refetchOnWindowFocus```, uma configuração padrão do react-query é fazer o refetch quando a janela é focada, ou seja, se você sair da janela e voltar, ele faria novamente as requests desnecessáriamente e geralmente no nosso caso não precisamos disso.  Então vamos aproveitar para refazer o código acima com a configuração do QueryClient: 
````javascript
import {
	QueryClient
	QueryClientProvider
} from 'react-query';


function MyApp() {
	 const queryClient = new QueryClient({
	 	defaultOptions: {
			queries: {
				refetchOnWindowFocus: false
			}
		}
	 });
	 return (
	 	<QueryClientProvider client={queryClient}>
			// Sua aplicação aqui
		</QueryClientProvider>
	 )
}

````

#### Configurando um componente simples:
Para servir de exemplo, vamos simular que temos uma requisição simples usando **axios**, vou usar um exemplo de uma API que retorna dados da NBA (pq gosto mt 🤓): 

```javascript
import axios from 'axios';

export const getNBAStats = () => {
	const { data } = axios.get('myFakeApi');
	return data;
}
```

Agora vamos criar nosso componente, no final vou deixar um bônus que pode servir de melhoria na organização das queries e requests. 

````typescript
import React from 'react'; 
import { getNBAStats } from 'myAPi';

function FakeApi() {
	const NBAQuery = useQuery('NBAQueryKey', getNBAStats) // Vamos falar desta key jajá
	return (
		<>
			{NBAQuery.data.map(nbaStats => (
				<div key={nbaStats.id}>
					{nbaStats.points}
				</div>
			))}
		</>
	)
}
````

#### Lidando com feedbacks:
O react-query disponibiliza pra nós alguns status da query, com isso conseguimos fazer um handler melhor dos status, são eles: 
```success```, ```error```, ```idle``` e ```loading```