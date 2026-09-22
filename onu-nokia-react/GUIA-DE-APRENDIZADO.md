# Guia de Aprendizado do Gerenciador Nokia ONU

Este guia explica o projeto como se ele fosse uma casa. Ele foi escrito para quem recebeu um fork pronto e quer aprender a fazer alterações com segurança.

## 1. O que este projeto faz

Este é um frontend: uma aplicação que roda no navegador. O usuário informa dados da ONU, como slot, porta PON e posição, e o sistema monta textos de comandos Nokia para copiar.

Ele não se conecta diretamente à OLT ou à ONU. O comando precisa ser colado em outra ferramenta.

## 2. Quais tecnologias existem aqui

- **JavaScript**: linguagem usada para a lógica.
- **JSX**: formato que permite escrever uma tela parecida com HTML dentro do JavaScript.
- **React**: biblioteca usada para dividir a tela em componentes reutilizáveis.
- **Vite**: ferramenta que inicia o servidor local e cria a versão de produção.
- **CSS**: controla cores, tamanhos, espaçamentos, fontes e responsividade.
- **react-icons**: fornece ícones para os botões e menus.
- **SweetAlert2**: exibe mensagens de sucesso, erro e confirmação.
- **npm**: instala as bibliotecas e executa os comandos do projeto.
- **GitHub Pages**: serviço preparado para publicar a aplicação.

As bibliotecas e os comandos usados pelo aplicativo estão em `package.json`.

## 3. Como a aplicação começa

Imagine que o navegador está abrindo a casa:

1. `index.html` fornece o espaço HTML com o elemento `root`.
2. `src/main.jsx` coloca o React dentro desse espaço.
3. `src/context/AppContext.jsx` fornece dados compartilhados.
4. `src/App.jsx` monta o layout principal.
5. `src/components/` fornece a tela que será exibida.

O React troca partes da tela sem recarregar a página inteira.

## 4. O mapa das pastas

| Local | Para que serve |
| --- | --- |
| `src/main.jsx` | Ponto inicial da aplicação |
| `src/App.jsx` | Menu, layout e seleção da tela ativa |
| `src/components/` | Telas e partes visuais |
| `src/config/menuItems.js` | Itens do menu lateral |
| `src/context/` | Dados compartilhados entre componentes |
| `src/services/` | Funções que montam comandos Nokia |
| `src/utils/` | Validações e funções auxiliares |
| `src/hooks/` | Comportamentos reutilizáveis |
| `src/App.css` | Estilos principais |
| `src/index.css` | Estilos globais |
| `public/` | Arquivos públicos copiados para a versão final |
| `dist/` | Resultado gerado pelo build; não é o local normal de edição |

## 5. Como uma tela funciona

Uma tela React normalmente tem quatro partes:

- **Componente**: função que desenha a tela.
- **Props**: dados recebidos de outro componente.
- **Estado**: dados que mudam enquanto o usuário usa a tela.
- **Eventos**: ações como clicar ou digitar.

Exemplo pequeno:

```jsx
import { useState } from 'react';

function Exemplo() {
  const [nome, setNome] = useState('');

  return (
    <input
      value={nome}
      onChange={(evento) => setNome(evento.target.value)}
      placeholder="Digite seu nome"
    />
  );
}

export default Exemplo;
```

`useState` guarda o nome. `value` mostra o valor. `onChange` percebe a digitação e atualiza o estado.

## 6. Fluxo da tela de Wi-Fi

A tela `ConfiguracaoWifi.jsx` segue este caminho:

1. Recebe a posição da ONU.
2. Guarda o nome e a senha com `useState`.
3. Valida os campos usando `src/utils/validation.js`.
4. Chama uma função de `src/services/wifiService.js`.
5. A função monta o comando usando `${variavel}`.
6. `copyToClipboard` copia o resultado.
7. Um alerta informa o resultado.

Separe sempre a aparência da regra: o componente desenha a tela; o serviço monta o comando.

## 7. Como mudar o visual

Para mudar um texto, procure esse texto dentro de `src/components/`.

Para mudar a aparência, encontre a classe do elemento:

```jsx
<button className="btn btn-primary">GERAR COMANDO</button>
```

Depois procure `.btn`, `.btn-primary` ou outra classe em `src/App.css`.

Você pode alterar `color`, `background`, `font-size`, `padding`, `margin`, `border` e `box-shadow`.

Não altere `dist/`: essa pasta será recriada pelo Vite.

## 8. Como mexer no que já está pronto

Comece sempre pelo resultado que você quer mudar e siga até o arquivo responsável.

### Alterar apenas um texto

Se quiser trocar o título ou o nome de um botão, abra o componente correspondente em `src/components/` e procure o texto.

Por exemplo, em `ConfiguracaoWifi.jsx`:

```jsx
<h3>CONFIGURAR REDE Wi-Fi</h3>
```

Você pode trocar somente o texto:

```jsx
<h3>CONFIGURAÇÃO DO WI-FI DO CLIENTE</h3>
```

Não apague as tags `<h3>`; elas dizem ao React o que deve aparecer na tela.

### Alterar somente a aparência

Se a tela está correta, mas você quer mudar cor, tamanho ou espaçamento, mantenha o JSX e altere a classe correspondente em `src/App.css`.

```css
.btn-primary {
  background: red;
  color: white;
}
```

Assim você muda a pintura do botão sem mudar o que ele faz.

### Alterar um comportamento

Se quiser mudar o que acontece ao clicar, procure o `onClick` no componente:

```jsx
<button onClick={handleAlterarNome}>
  ALTERAR APENAS NOME
</button>
```

Depois procure a função `handleAlterarNome` no mesmo arquivo. Ela pode chamar um serviço em `src/services/`.

Faça mudanças de comportamento somente depois de entender o caminho completo: botão, função, validação e serviço.

### Método de trabalho para uma alteração

1. Escolha uma alteração pequena, como trocar um título.
2. Use `Ctrl+Shift+F` para procurar o texto no projeto.
3. Leia algumas linhas acima e abaixo do resultado.
4. Altere uma coisa por vez.
5. Salve e veja o resultado com `npm run dev`.
6. Se algo quebrar, desfaça somente a última mudança e tente novamente.
7. Depois de funcionar, rode `npm run build`.

Uma mudança visual normalmente envolve `components/` e `App.css`. Uma mudança em comando normalmente envolve `components/`, `utils/` e `services/`.

## 9. Como mudar um comando

Abra o serviço relacionado em `src/services/`. Os valores entram no texto por meio de template strings:

```js
return `COMANDO-${inputSlot}-${inputGpon}-${inputIndex};`;
```

Teste os comandos com dados fictícios antes de usar em produção. Um caractere errado pode gerar uma configuração inválida.

## 10. Como adicionar uma nova opção

Use esta ordem:

1. Crie um serviço em `src/services/`.
2. Crie o componente em `src/components/`.
3. Adicione o item em `src/config/menuItems.js`.
4. Importe o componente em `src/App.jsx`.
5. Adicione um `case` para o mesmo `id` do menu.
6. Teste a tela, os campos vazios e os dados válidos.

Exemplo do menu:

```js
{ id: 'reiniciar', label: 'Reiniciar ONU', icon: FiRefreshCw }
```

Exemplo no `App.jsx`:

```jsx
case 'reiniciar':
  return <ReiniciarOnu posicaoData={posicaoData} />;
```

O texto `reiniciar` precisa ser igual nos dois lugares.

## 11. Como trabalhar com segurança

Faça uma alteração pequena por vez. Antes de editar, procure o texto ou nome da classe usando `Ctrl+Shift+F` no VS Code.

Depois de cada alteração:

```bash
npm run lint
npm run build
```

Em seguida, teste no navegador com:

```bash
npm run dev
```

Não coloque senhas reais, credenciais ou comandos sensíveis em commits, exemplos ou mensagens de erro.

## 12. Primeiros exercícios

1. Troque o título de uma tela.
2. Altere uma cor em `App.css`.
3. Adicione um campo visual sem mudar comandos.
4. Crie um serviço que gere um comando fictício.
5. Crie uma nova opção no menu.
6. Faça um commit separado para cada aprendizado.

## 13. Regra principal

A tela fica em `components`, o comando fica em `services`, as validações ficam em `utils`, e o visual fica no CSS. Quando você não souber onde editar, siga essa divisão.
