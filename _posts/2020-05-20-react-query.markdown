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

**Por que salvar em cache?**
Uma das estratégias do react-query de salvar em cache é que em cache nao precisa renderizar um componente ao menos que esse cache altere.

Como o intuito deste post é ter mais um **hands-on**, vamos por a mão na massa para tornar o post mais dinâmico 😊. 

Lembrando que se discordarem/tiverem alguma dúvida, crítica ou sugestão, só me mandar um e-mail que a gente troca uma idéia.

#### Instalando o react query:
```
npm install react-query 
// ou 
yarn add react-query
```

### Criando o provider:

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


### Configurando um componente simples:

Para servir de exemplo, vamos simular que temos uma requisição simples usando **axios**, vou usar um exemplo de uma API que retorna dados da NBA (pq gosto mt 🤓): 

```javascript
import axios from 'axios';

export const getNBAStats = () => {
	const { data } = axios.get('myFakeApi');
	return data;
}
```

Agora vamos criar nosso componente, no final vou deixar um **bônus** que pode servir de melhoria na organização das queries e requests. 

````javascript
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

### Keys
As mutations (você verá a seguir) e as queries são identificadas no cache do react-query com **keys**, com estas keys é possível invalidar a query e ao ser invalidada, a query automaticamente vai no server-side buscar o novo dado. Com as keys também é possível identificar o dado em cache. 

### Lidando com feedbacks:

O react-query disponibiliza pra nós alguns status da **query** ou a **mutation**, com isso conseguimos fazer um handler melhor dos status, são eles: 
```success```, ```error```, ```idle``` e ```loading``` ou existem flags também disponíveis: 
-   `isIdle` ou `status === 'idle'` \- A query/mutation não está sendo utilizada
-   `isLoading` ou `status === 'loading'` \- A query/mutation está realizando a ação determinada a ela
-   `isError` ou `status === 'error'` \- A query/mutation encontrou um erro
-   `isSuccess` ou `status === 'success'` \- A query/mutation realizou o designado com sucesso


Sendo assim, o tratamento dos feedbacks pode ser utilizado da seguinte forma: 
```javascript
import React from 'react'; 
import { getNBAStats } from 'myAPi';
import { LoadingComponent } from './components/LoadingComponent';
import { useQuery } from 'react-query'; 

function FakeApi() {
	const NBAQuery = useQuery('NBAQueryKey', getNBAStats) // Vamos falar desta key jajá
	
	if(NBAQuery.status === 'loading') return <LoadingComponent />
	
	if(NBAQuery.status === 'error') return <ErrorComponent />
	
	if(NBAQuery.status === 'success' && !NBAQuery.data.length) return <NoContent />
	
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
```

### Mutations

Uma mutation basicamente é utilizada para fazer algum tipo de alteração no server-side (perceba como a mágica do react-query flui automaticamente). O comando para utilizar-se mutations é o ```useMutation```.
Uma mutation é invocada quando chamamos o método ```mutate()```.
Vamos usar como exemplo uma mutation que vai 'adicionar um jogador' em nossa API:
````javascript
import React from 'react';
import { useMutation } from 'react-query'; 
import axios from 'axios';

function AddPlayer() {
	const playerMutation = useMutation(newPlayer => axios.post('newPlayer', newPlayer));
	
	if(playerMutation.status === 'loading') return <LoadingComponent />
	
	if(playerMutation.status === 'error') return <ErrorHandler />
	
	if(playerMutation.status === 'success') return <span>Jogador adicionado com sucesso</span>
	
	return (
		<button 
			onClick={() => { playerMutation.mutate({ name: 'Michael Jordan', age: 18 })}}
		>Adicionar jogador
		</button>
	)
}
````


### Funções de uma mutation:

Digamos que temos uma mutation na qual sempre que ela faça uma request com sucesso eu quero disparar alguma outra ação, sendo assim, definimos a query com a função ```onSuccess()```, vamos também incluir algumas outras funções: 

```javascript
import { useMutation } from 'react-query'; 
import axios from 'axios';

export function addPlayerMutation(data) {
	return useMutation(() => axios.post('newPlayer', data), {
		mutationKey: 'newPlayerMutation', 
		onError: (error) => {
			console.log(error);
		}, 
		onSuccess: () => {
			alert('success');
		}
	})
}
```


### Invalidando uma query:

Em qualquer lugar da aplicação abaixo do nosso provider nós conseguimos tanto acessar dados no cache quanto invalidar uma query já feita para que o dado possa ser atualizado. Num exemplo bem simples, podemos fazer uma funcionalidade em que no momento que um jogador pontuar, queremos invalidar a query que busca a lista de jogadores para que a lista possa vir refletida com o que se encontra no server-side. 
**Ah Tibau, mas não é só no retorno da API de inserção retornar a nova lista**, eh, até podemos, mas iremos perder a ideia do componente ser o mais genérico possível: 

```javascript
import { useQueryClient, useMutation } from 'react-query';
import axios from 'axios';

const queryClient = useQueryClient();

export function addPoint(data) {
	return useMutation(() => axios.post('addPoint', data), {
		mutationKey: 'addPoint', 
		onError: (error) => {
			console.log(error);
		}, 
		onSuccess: () => {
			// Lembra nossa query lá no começo do tutorial? 
			// Vamos invalidar ela e os dados serão atualizados automaticamente
			// MAGIC 🌟 nós não precisamos fazer uma nova implementação de uma
			// request ou chamar uma query, é só invalidar:
			queryClient.invalidateQueries('NBAQueryKey')
		}
	})
}
```

Ou seja, toda vez que ao inserir um ponto, no sucesso da inserção, eu buscarei novos dados do servidor. 

### Bônus:
Deixei pro final algumas coisas não obrigatórias nos estudos de react-query que eu acho que é bacana ser discutido:

#### Organização das queries:
Pode ser que seja mais interessante separar as requests do axios das queries, em alguns casos, é legal ter como organização uma request axios genérica pronta aonde todos as queries/mutations chamam ela. 

#### Tibau, discordo, tenho dúvida, critica ou sugestão, como faço para te xingar? 
Manda um e-mail pra mim hehe: ttibaudev@gmail.com
