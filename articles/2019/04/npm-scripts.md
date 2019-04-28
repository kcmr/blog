# npm scripts

Reconocidos por npm (se pueden ejecutar sin `run`):

- `test` -> alias `t`
- `start`

Ver variables de entorno usadas por npm:

```bash
npm run env | grep "npm_package"
```

Aquí podemos ver que hay una variable PATH en la que está la ruta a `node_modules/.bin/` para que no sea necesario especificar el path para los paquetes instalados.

## Ejecutar npm scripts en series

Usar operador `&&`.   
Si alguna de las series falla, el resto de scripts no se ejecutan.

```json
"scripts": {
  "test": "npm run eslint && npm run stylelint && mocha spec/ --require babel-register",
  "start": "node index.js",
  "eslint": "eslint --cache --fix ./",
  "stylelint": "stylelint '**/*.scss' --syntax scss"
}
```

## Ejecutar npm scripts en paralelo

Usar operador `&`:

```json
"scripts": {
  "test": "npm run eslint & mocha spec/ --require babel-register --watch & npm run stylelint",
  "start": "node index.js",
  "eslint": "eslint --cache --fix ./",
  "stylelint": "stylelint '**/*.scss' --syntax scss"
}
```
El comando `wait` permite esperar hasta que una tarea finaliza. https://egghead.io/lessons/tools-run-npm-scripts-in-parallel

## npm-run-all como shortcut para ejecutar scripts en series o en paralelo

```json
npm i -D npm-run-all
```

Esto nos permite reemplazar los `npm run` dentro de un script por nada y también los operadores `&`. Por defecto los scripts se ejecutan en series.

```json
"scripts": {
  "test": "npm-run-all eslint mocha stylelint"
}
```

Para ejecutar los scripts en paralelo, se usa el flag `--parallel`:

```json
"scripts": {
  "test": "npm-run-all --parallel eslint mocha stylelint"
}
```

Aquí no hace falta usar el comando `wait`.

## Ejecutar grupos de scripts similares

`npm-run-all` soporta el uso de wildcards para ejecutar un conjunto de scripts con un nombre similar, como por ejemplo: `lint:js`, `lint:css`, etc. Además acepta glob patterns.

Ejemplo de uso:

```json
"scripts": {
  "test": "npm-run-all lint:* mocha",
  "lint:js": "comando",
  "lint:css": "comando"
}
```

Con glob patterns:

```json
"scripts": {
  "test": "npm-run-all lint:** mocha",
  "lint:js": "comando",
  "lint:js:fix": "comando --fix",
  "lint:css": "comando"
}
```

Más mejor:

```json
"scripts": {
  "test": "npm-run-all lint mocha",
  "lint": "npm-run-all lint:**",
  "lint:js": "comando",
  "lint:js:fix": "comando --fix",
  "lint:css": "comando"
}
```

## Pre y post scripts

Para cada npm script hay una versión pre y post. Para usarlas simplemente basta con crear otro script con la palabra `pre` o `post` antes del script. npm ejecutará automáticamente esas tareas antes o después del script correspondiente.

Esto puede ser útil para limpiar carpetas generadas después de un comando, o realizar alguna ejecución necesaria previa a otro comando.

```json
"scripts": {
  "pretest": "npm run lint",
  "test": "mocha spec/ --require babel-register",
  "cover": "nyc npm t",
  "postcover": "rm -rf .nyc_output"
}
```

## Paso de argumentos a tareas

Para pasar argumentos adicionales a una tarea se puede usar el flag `--` seguido del argumento.

Ejemplo:

```json
"scripts": {
  "test": "mocha spec/ --require babel-register"
  "watch:test": "npm t -- --watch"
}
```

## Encadenar datos entre scripts (pipe)

Para pasar el resultado de un comando a otro se usa el carácter de pipe (`|`). Para especificar la salida usamos el carácter mayor que (`>`).

Ejemplo:

```json
"scripts": {
  "build": "npm-run-all build:*",
  "prebuild": "rm -rf public/",
  "build:html": "pug --obj data.json src/index.pug --out public/",
  "build:css": "node-sass src/index.scss | postcss -c .postcssrc.json | cssmin > public/index.min.css",
  "build:js": "mustache data.json src/index.mustache.js | uglifyjs > public/index.min.js"
}
```

## Ejecutar scripts cuando cambian archivos

Algunos comandos permiten el uso del flag `--watch`. Para los que no lo permiten, se puede usar el paquete `onchange` que permite ejecutar alguna tarea siempre que ciertos scripts cambien.

Ejemplo:

```json
"scripts": {
  "lint": "npm-run-all lint:**",
  "lint:js": "eslint --cache --fix ./",
  "lint:css": "stylelint '**/*.scss' --syntax scss",
  "lint:css:fix": "stylefmt -R src/",
  "watch": "npm-run-all --parallel watch:*",
  "watch:test": "npm t -- --watch",
  "watch:lint": "onchange 'src/**/*.js' 'src/**/*.scss' -- npm run lint"
}
```