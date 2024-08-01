## Next.js App Router Course - Starter

This is the starter template for the Next.js App Router Course. It contains the starting code for the dashboard application.

For more information, see the [course curriculum](https://nextjs.org/learn) on the Next.js Website.

### Resumo Cap.10

- Para recapitular, você fez algumas coisas para otimizar a busca de dados em seu aplicativo:

1. Crie um banco de dados na mesma região que o código do seu aplicativo para reduzir a latência entre o servidor e o banco de dados.

2. Dados buscados no servidor com React Server Components. Isso permite que você mantenha buscas de dados e lógica caras no servidor, reduz o pacote JavaScript do lado do cliente e impede que os segredos do seu banco de dados sejam expostos ao cliente.

3. Use SQL para buscar apenas os dados necessários, reduzindo a quantidade de dados transferidos para cada solicitação e a quantidade de JavaScript necessária para transformar os dados na memória.

4. Paralelize a busca de dados com JavaScript - onde fizer sentido.

5. Streaming implementado para evitar que solicitações lentas de dados bloqueiem sua página inteira e para permitir que o usuário comece a interagir com a interface do usuário sem esperar que tudo carregue.

6. Mova a busca de dados para os componentes que precisam dela, isolando assim quais partes de suas rotas devem ser dinâmicas.


### Quando usar o useSearchParams()gancho ou o searchParamssuporte?

Você pode ter notado que usou duas maneiras diferentes de extrair parâmetros de pesquisa. Se você usa uma ou outra depende se está trabalhando no cliente ou no servidor.

- <Search>é um componente cliente, então você usou o useSearchParams()gancho para acessar os parâmetros do cliente.
- <Table>é um componente de servidor que busca seus próprios dados, para que você possa passar a searchParamspropriedade da página para o componente.
Como regra geral, se você quiser ler os parâmetros do cliente, use o useSearchParams()hook, pois isso evita ter que voltar ao servidor.

### Deboucing

#### Melhor prática: Debouncing
Parabéns! Você implementou a pesquisa com Next.js! Mas há algo que você pode fazer para otimizá-la.

Dentro da sua handleSearchfunção, adicione o seguinte console.log:

- /app/ui/pesquisa.tsx
~~~ts
function handleSearch(term: string) {
  console.log(`Searching... ${term}`);
 
  const params = new URLSearchParams(searchParams);
  if (term) {
    params.set('query', term);
  } else {
    params.delete('query');
  }
  replace(`${pathname}?${params.toString()}`);
}
~~~
Então digite "Emil" na sua barra de pesquisa e verifique o console em dev tools. O que está acontecendo?

Console de ferramentas de desenvolvimento

```bash
Searching... E
Searching... Em
Searching... Emi
Searching... Emil
```

Você está atualizando a URL a cada pressionamento de tecla e, portanto, consultando seu banco de dados a cada pressionamento de tecla! Isso não é um problema, pois nosso aplicativo é pequeno, mas imagine se seu aplicativo tivesse milhares de usuários, cada um enviando uma nova solicitação ao seu banco de dados a cada pressionamento de tecla.

Debouncing é uma prática de programação que limita a taxa na qual uma função pode disparar. No nosso caso, você só quer consultar o banco de dados quando o usuário tiver parado de digitar.

Como funciona o Debouncing:

Evento de gatilho : quando um evento que deve ser eliminado (como um pressionamento de tecla na caixa de pesquisa) ocorre, um cronômetro é iniciado.
Aguardar : se um novo evento ocorrer antes que o cronômetro expire, o cronômetro será zerado.
Execução : Se o cronômetro atingir o fim da contagem regressiva, a função debounced será executada.
Você pode implementar o debouncing de algumas maneiras, incluindo a criação manual de sua própria função de debounce. Para manter as coisas simples, usaremos uma biblioteca chamadause-debounce.

Instalar use-debounce:

terminal

```bash
pnpm i use-debounce
```
No seu <Search>Componente, importe uma função chamada useDebouncedCallback:

/app/ui/pesquisa.tsx
~~~ts
// ...
import { useDebouncedCallback } from 'use-debounce';
 
// Inside the Search Component...
const handleSearch = useDebouncedCallback((term) => {
  console.log(`Searching... ${term}`);
 
  const params = new URLSearchParams(searchParams);
  if (term) {
    params.set('query', term);
  } else {
    params.delete('query');
  }
  replace(`${pathname}?${params.toString()}`);
}, 300);
~~~
Esta função encapsulará o conteúdo de handleSearche executará o código somente após um tempo específico, quando o usuário parar de digitar (300 ms).

Agora digite na sua barra de pesquisa novamente, e abra o console em dev tools. Você deve ver o seguinte:

Console de ferramentas de desenvolvimento

Searching... Emil
Ao eliminar o efeito de rejeição, você pode reduzir o número de solicitações enviadas ao seu banco de dados, economizando recursos.

É hora de fazer um teste!
