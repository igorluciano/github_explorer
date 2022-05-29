# ReactJS

# Estrutura de pastas base do projeto

- Criar a estrutura de pastas base projeto e incluir os arquivos `./src/index.jsx`, `./src/App.jsx`, `./src/styles/global.scss`, `./public/index.html` e `./disc/bundle.js`, mesmo que vazios inicialmente, conforme a estrutura abaixo:

> `.jsx` indica que estamos utilizando HTML no interior do Javascript. É apenas uma convenção.

```bash
project/
    |- src
        |-- styles
            |-- global.scss
        |-- App.jsx
        |-- index.jsx
    |- public
        |-- index.html
    |- dist
        |-- bundle.js
```

## Configuração das dependências do projeto

```bash
# Inicializa o projeto e package.json
yarn init -y

# Inclui react para lidar com interfaces
yarn add react

# Inclui react-dom para lidar com árvore de rederização do dom na web
yarn add react-dom

# Inclui dependências do Babel para compatibilizar os códigos js para os navegadores
# Incluídas apenas no ambiente de desenvolvimento, por isso o -D
# `preset-react` permite o Babel interpretar e converter os códigos react
# `babel-loader` permite integrar Webpack e Babel
yarn add @babel/core @babel/cli @babel/preset-env @babel/preset-react babel-loader -D

# Incluí as dependências do Webpack para permiti-lo empacotar a aplicação
# Incluídas apenas no ambiente de desenvolvimento
yarn add webpack webpack-cli -D
```

# Codificando os arquivos que compõem o projeo

- Definir a estrutura base do arquivo `./public/index.html` como:

> Repare que o arquivo deve possuir obrigatoriamente o trecho `<div id="root"></div>`, pois será utilizado pelo React para renderizar os componentes da página.

> É importante também que o trecho `<script src="../dist/bundle.js"></script>` esteja presente para possibilitar anexar nosso código React empacotado pelo Webpack e compatibilizado pelo Babel.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <div id="root"></div>
</body>
<script src="../dist/bundle.js"></script>
</html>
```

- Definir a estrutura base do arquivo `./src/index.jsx` como:

> Repare que é necessário importar o método `render` do `react-dom` para possibilitar que possamos renderizar os componentes do React no interior da div com id `root`.

```js
// Para React na versão 17
import { render } from "react-dom";
import { App } from "./App";
render(<App />, document.getElementById('root'))

// Para React 18 ou superior
import { createRoot } from 'react-dom/client';
import { App } from "./App";
const container = document.getElementById('root');
const root = createRoot(container);
root.render(<App />);
```

- Definir a estrutura base do arquivo `./src/App.jsx` como:

```js
export function App() {
    return <h1>Hello World</h1>
}
```

## Configuração da conversão via Babel

- Criar arquivo de `babel.config.js` na raíz do projeto e  incluir nele as predefinições de conversão do Babel:

> Para possibilitar o Babel interpretar os códigos React, deve-se carregar o `preset-react` e há duas formas de se fazer isso no arquivo `babel.config.js`. A primeira é apenas incluindo a indicação `@babel/preset-react`, na qual há a necessidade de todos os componentes React possuírem o trecho de código `ìmport React from "react";`. E a segunda, predefinindo que não há a necessidade e essa inclusão será feita automaticamente, por meio da configuração `["@babel/preset-react", {"runtime": "automatic"}]`.

```bash
module.exports = {
    presets: [
        "@babel/preset-env",
        ["@babel/preset-react", {
            "runtime": "automatic"
        }]
    ]
}
```

- Caso deseje ver os efeitos da conversão de forma exemplificada, basta incluir o trecho de código abaixo no arquivo `./src/index.jsx` e perceber a diferença do antes e depois da compatibilização.

```js
const user = {
    "name": "Igor"
};

// Operador "?" é um recurso recente do ECMAScript
// E será convertido para código Javascript compatível
console.log(user.emails?.personal);
```

- Gerar o `bundle.js` manual, que é o código javascript convertido e compatibilizado, com o comando:

```bash
# `dist` é a pasta contendo o código convertido, e remete a distribuição do código
yarn babel src/index.jsx --out-file dist/bundle.js
```

## Configurações de empacotamento via Webpack

- Criar arquivo de `webpack.config.js` na raíz do projeto e  incluir nele as predefinições de empacotamento do Webpack:

> Repare que o atributo `mode: 'development'`, é utilizado para deixar mais rápido o empacotamente e retirar algumas verificações realizadas no empacotamento da versão de produção.

```js
const path = require('path');

module.exports = {
    mode: 'development',
    entry: path.resolve(__dirname, 'src', 'index.jsx'),
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js',
    },
    resolve: {
        extensions: ['.js', '.jsx']
    },
    module: {
        rules: [
            {
                test: /\.jsx$/,
                exclude: /node_modules/,
                use: 'babel-loader',
            }
        ]
    }
};
```

- Gerar o `bundle.js` da aplicação via Webpack, contendo o código javascript empacotado pelo Webpack e compatibilizado via Babel, com o comando:

```bash
yarn webpack
```

# Aperfeiçoamentos no empacotamento via Webpack

## Gerar estaticamente o HTML base da aplicação

- Configurar a dependência do plugin `html-webpack-plugin`:

```bash
# Inclui o plugin `html-webpack-plugin` como dependência de desenvolvimento
yarn add html-webpack-plugin -D
```

- Configurar plugin para gerar automaticamente um arquivo `index.html` a partir da referência disponível em `./public/index.html` e depois fazer a inclusão automática do `bundle.js` nesse novo arquivo a cada empacotamento realizado pelo Webpack:

```js
module.exports = {
...
    plugins: [
            new HtmlWebpackPlugin({
                template: path.resolve(__dirname, 'public', 'index.html')
            }),
        ],
...
};
```

## Configurar um servidor para os arquivos estáticos

- Configurar a dependência do plugin `webpack-dev-server`:

```bash
# Inclui o plugin `webpack-dev-server` como dependência de desenvolvimento
yarn add webpack-dev-server -D
```

- Configurar o parâmetro `devServer` para considerar os arquivos estáticos da aplicação e rodar o servidor de desenvolvimento a partir deles:

```js
module.exports = {
...
    devServer: {
        static: {
            directory: path.resolve(__dirname, 'public'),
        }
    },
...
};
```

- Executar a aplicação por meio do servidor de desenvolvimento com o comando:

```bash
yarn webpack server
```


## Habilitar apontamento de erros no DevTool

- Configurar o debug e apontamento de erros do DevTool para indicar exatamente a linha e arquivo onde determinado erro ocorreu:

> Repare que ao simular um erro qualquer no arquivo `./src/index.jsx`, incluindo o código `throw new Error('Um baita erro!');`, sem a configuração do parâmetro ` devtool: 'eval-source-map',` não é possível identificar exatamente onde ocorreu o problema, devido o código estar empacotado pelo Webpack e também compatibilizado pelo Babel.

```js
module.exports = {
...
    devtool: 'eval-source-map',
...
};
```

## Configurar ambiente Development e Production

- Configurar a dependência `cross-env` para diferenciar os ambientes Development e Production:

```bash
# Inclui o `cross-env` como dependência de desenvolvimento
yarn add cross-env -D
```

- Ajustar no arquivo `webpack.config.js` os paraêmtros `mode` e `devtool` para aplicar configurações diferetentes conforme o valor da variável `isDevelopment`:

```js
const isDevelopment = process.env.NODE_ENV !== 'production';

module.exports = {
    mode: isDevelopment ? 'development' : 'production',
    devtool: isDevelopment ? 'eval-source-map': 'source-map',
    ...
};
```

- Incluir no arquivo `package.json` o paraêmtro `scripts` contendo os comandos para execução da aplicação nos ambientes Development e Production:

```js
{
  ...
  "scripts": {
    "dev": "webpack serve",
    "build": "cross-env NODE_ENV=production webpack"
  },
  ...
}
```

> Agora, para executar a aplicação no ambiente Development, utiliza-se o comando `yarn dev` e para executar no ambiente Production, utiliza-se o comando `yarn build`.

## Permitir importar e usar arquivos CSS 

- Configurar a dependência `style-loader`  e `css-loader` para interpretar os arquivos `.scss` e poderem ser acopladas à aplicação pelo Webpack:

```bash
# Inclui o `style-loader` e `css-loader` como dependência de desenvolvimento
yarn add style-loader css-loader -D
```

- Incluir um estilo básico no arquivo `./src/styles/global.scss` para verificar se o Webpack irá integrá-lo à aplicação:

```css
body {
    color: blue;
    background-color: red;
}
```

- Importar o arquivo `./src/styles/global.scss` no arquivo ``./src/App.jsx`:

```js
import "./styles/global.scss";
...
```

- Incluir no arquivo `webpack.config.js` uma regra para mapear e encontrar os arquivos ``.scss` para serem incorporados à aplicação pelo Webpack:

```js
module.exports = {
    ...
    module: {
        rules: [
            ...
            {
                test: /\.scss$/i,
                use: ['style-loader','css-loader'],
            }
        ]
    }
    ...
};
```

## Configura o pré-processador SASS para incrementar o CSS

- Configurar a dependência `sass` e `sass-loader` para interpretar converter os arquivos SASS (aqueles com extensão .scss) para CSS durante o empacotamento realizado pelo Webpack:

```bash
# Inclui o `sass` e `sass-loader` como dependência de desenvolvimento
yarn add sass sass-loader -D
```

- Incluir no arquivo `webpack.config.js` uma interpretador de arquivos SASS, aqueles com a extensão `.scss`, para serem incorporados à aplicação pelo Webpack:

```js
module.exports = {
    ...
    module: {
        rules: [
            ...
            {
                test: /\.scss$/i,
                use: ['style-loader','css-loader', 'sass-loader'],
            }
        ]
    }
    ...
};
```
- Incluir um estilo básico no arquivo `./src/styles/global.scss` para verificar se o Webpack irá integrá-lo e considerar as funcionalidades SASS na aplicação:

```css
body {
    background-color: red;

    h1 {
        color: green;
    }
}
```
