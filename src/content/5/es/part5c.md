---
mainImage: ../../../images/part-5.svg
part: 5
letter: c
lang: es
---

<div class="tasks">

La librería de pruebas utilizada en esta parte se cambió el 3 de marzo de 2024 de Jest a Vitest. Si ya comenzaste esta parte usando Jest, puedes ver [aquí](https://github.com/fullstack-hy2020/fullstack-hy2020.github.io/blob/02d8be28b1c9190f48976fbbd2435b63261282df/src/content/5/es/part5c.md) el contenido antiguo.

</div>

<div class="content">

Hay muchas formas diferentes de probar aplicaciones React. Echemos un vistazo a ellas a continuación.

Anteriormente, el curso utilizaba la librería [Jest](http://jestjs.io/) desarrollada por Facebook para probar componentes de React. Ahora estamos utilizando la nueva generación de herramientas de prueba de los desarrolladores de Vite llamada [Vitest](https://vitest.dev/). Aparte de las configuraciones, las librerías proporcionan la misma interfaz de programación, por lo que prácticamente no hay diferencia en el código de prueba.

Comencemos instalando Vitest y la librería [jsdom](https://github.com/jsdom/jsdom) que simula un navegador web:

```
npm install --save-dev vitest jsdom
```

Además de Vitest, también necesitamos otra librería de pruebas que nos ayudará a renderizar componentes para fines de prueba. La mejor opción actual para esto es [react-testing-library](https://github.com/testing-library/react-testing-library), que ha visto un rápido crecimiento en popularidad recientemente. También vale la pena extender el poder expresivo de las pruebas con la librería [jest-dom](https://github.com/testing-library/jest-dom).

Instalemos las librerías con el comando:

```js
npm install --save-dev @testing-library/react @testing-library/jest-dom
```

Antes de que podamos hacer la primera prueba, necesitamos algunas configuraciones.

Agregamos un script al archivo <i>package.json</i> para ejecutar las pruebas:

```js 
{
  "scripts": {
    // ...
    "test": "vitest run"
  }
  // ...
}
```

Vamos a crear un archivo _testSetup.js_ en la raíz del proyecto con el siguiente contenido

```js
import { afterEach } from 'vitest'
import { cleanup } from '@testing-library/react'
import '@testing-library/jest-dom/vitest'

afterEach(() => {
  cleanup()
})
```

Ahora, luego de cada prueba, la función _cleanup_ es ejecutada para resetear jsdom, que esta simulando al navegador.

Expandamos el archivo _vite.config.js_ de la siguiente manera:

```js
export default defineConfig({
  // ...
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: './testSetup.js', 
  }
})
```

Con _globals: true_, no es necesario importar palabras clave como _describe_, _test_ y _expect_ en las pruebas.

Primero escribamos pruebas para el componente que es responsable de renderizar una nota:

```js
const Note = ({ note, toggleImportance }) => {
  const label = note.important
    ? 'make not important'
    : 'make important'

  return (
    <li className='note'> // highlight-line
      {note.content}
      <button onClick={toggleImportance}>{label}</button>
    </li>
  )
}
```

Observa que el elemento <i>li</i> tiene el valor <i>note</i> para el atributo de [CSS](https://es.react.dev/learn#adding-styles) className, que podríamos usar para acceder al componente en nuestras pruebas.

### Renderizando el componente para pruebas

Escribiremos nuestra prueba en el archivo <i>src/components/Note.test.jsx</i>, que está en el mismo directorio que el componente Note.

La primera prueba verifica que el componente muestra el contenido de la nota:

```js
import { render, screen } from '@testing-library/react'
import Note from './Note'

test('renders content', () => {
  const note = {
    content: 'Component testing is done with react-testing-library',
    important: true
  }

  render(<Note note={note} />)

  const element = screen.getByText('Component testing is done with react-testing-library')
  expect(element).toBeDefined()
})
```

Después de la configuración inicial, la prueba renderiza el componente con el método [render](https://testing-library.com/docs/react-testing-library/api#render) proporcionado por react-testing-library:

```js
render(<Note note={note} />)
```

Normalmente, los componentes de React se procesan en el [DOM](https://developer.mozilla.org/es/docs/Web/API/Document_Object_Model). El método de renderizado que usamos renderiza los componentes en un formato que es adecuado para pruebas sin renderizarlos al DOM.

Podemos usar el objeto [screen](https://testing-library.com/docs/queries/about#screen) para acceder al componente renderizado. Utilizamos el método [getByText](https://testing-library.com/docs/queries/bytext) de screen para buscar un elemento que tenga el contenido de la nota y asegurarnos de que existe:

```js
  const element = screen.getByText('Component testing is done with react-testing-library')
  expect(element).toBeDefined()
```

La existencia de un elemento se verifica utilizando el comando [expect](https://vitest.dev/api/expect.html#expect) de Vitest. Expect genera una aserción para su argumento, cuya validez se puede probar utilizando varias funciones de condición. Ahora utilizamos [toBeDefined](https://vitest.dev/api/expect.html#tobedefined) que prueba si el argumento _element_ de expect existe.

Ejecuta el test con el comando _npm test_:

```js
$ npm test

> notes-frontend@0.0.0 test
> vitest


 DEV  v1.3.1 /Users/mluukkai/opetus/2024-fs/part3/notes-frontend

 ✓ src/components/Note.test.jsx (1)
   ✓ renders content

 Test Files  1 passed (1)
      Tests  1 passed (1)
   Start at  17:05:37
   Duration  812ms (transform 31ms, setup 220ms, collect 11ms, tests 14ms, environment 395ms, prepare 70ms)


 PASS  Waiting for file changes...
```

Eslint se queja de las palabras clave _test_ y _expect_ en las pruebas. El problema se puede resolver instalando [eslint-plugin-vitest-globals](https://www.npmjs.com/package/eslint-plugin-vitest-globals):

```
npm install --save-dev eslint-plugin-vitest-globals
```

y habilitando el plugin al editar el archivo _.eslint.cjs_ de la siguiente manera:

```js
module.exports = {
  root: true,
  env: {
    browser: true,
    es2020: true,
    "vitest-globals/env": true // highlight-line
  },
  extends: [
    'eslint:recommended',
    'plugin:react/recommended',
    'plugin:react/jsx-runtime',
    'plugin:react-hooks/recommended',
    'plugin:vitest-globals/recommended', // highlight-line
  ],
  // ...
}
```

### Ubicación del archivo de prueba

En React hay (al menos) [dos convenciones diferentes](https://medium.com/@JeffLombardJr/organizing-tests-in-jest-17fc431ff850) para la ubicación de los archivos de prueba. Creamos nuestros archivos de prueba de acuerdo con el estándar actual colocándolos en el mismo directorio que el componente que se está probando.

La otra convención es almacenar los archivos de prueba "normalmente" en su propio directorio separado _test_. Cualquiera que sea la convención que elijamos, es casi seguro que estará equivocada según la opinión de alguien.

Personalmente, no me gusta esta forma de almacenar pruebas y código de aplicación en el mismo directorio. La razón por la que elegimos seguir esta convención es que está configurada de forma predeterminada en las aplicaciones creadas por Vite o create-react-app.

### Búsqueda de contenido en un componente

El paquete react-testing-library ofrece muchas formas diferentes de investigar el contenido del componente que se está probando. En realidad, el _expect_ en nuestra prueba no es necesario en absoluto:

```js
import { render, screen } from '@testing-library/react'
import Note from './Note'

test('renders content', () => {
  const note = {
    content: 'Component testing is done with react-testing-library',
    important: true
  }

  render(<Note note={note} />)

  const element = screen.getByText('Component testing is done with react-testing-library')

  expect(element).toBeDefined() // highlight-line
})
```

La prueba falla si _getByText_ no encuentra el elemento que está buscando.

También podríamos usar [selectores CSS](https://developer.mozilla.org/es/docs/Web/CSS/Selectores_CSS) para encontrar elementos renderizados mediante el uso del método [querySelector](https://developer.mozilla.org/es/docs/Web/API/Document/querySelector) del objeto [container](https://testing-library.com/docs/react-testing-library/api/#container-1), que es uno de los campos devueltos por el renderizado:

```js
import { render, screen } from '@testing-library/react'
import Note from './Note'

test('renders content', () => {
  const note = {
    content: 'Component testing is done with react-testing-library',
    important: true
  }

  const { container } = render(<Note note={note} />) // highlight-line

// highlight-start
  const div = container.querySelector('.note')
  expect(div).toHaveTextContent(
    'Component testing is done with react-testing-library'
  )
  // highlight-end
})
```

**NB:** Una forma más consistente de seleccionar elementos es usando un [atributo de datos](https://developer.mozilla.org/es/docs/Web/HTML/Global_attributes/data-*) que esté específicamente definido para propósitos de prueba. Usando _react-testing-library_, podemos aprovechar el método [getByTestId](https://testing-library.com/docs/queries/bytestid/) para seleccionar elementos con un atributo _data-testid_ especificado.

### Depurando pruebas

Normalmente nos encontramos con muchos tipos diferentes de problemas al escribir nuestras pruebas.

El objeto _screen_ tiene el método [debug](https://testing-library.com/docs/dom-testing-library/api-debugging#screendebug) que se puede utilizar para imprimir el HTML de un componente en el terminal. Si cambiamos la prueba de la siguiente manera:

```js
import { render, screen } from '@testing-library/react'
import Note from './Note'

test('renders content', () => {
  const note = {
    content: 'Component testing is done with react-testing-library',
    important: true
  }

  render(<Note note={note} />)

  screen.debug() // highlight-line

  // ...

})
```

podemos ver el HTML generado por el componente en la consola:

```js
console.log
  <body>
    <div>
      <li
        class="note"
      >
        Component testing is done with react-testing-library
        <button>
          make not important
        </button>
      </li>
    </div>
  </body>
```

También es posible utilizar el mismo método para imprimir el elemento que queramos en la consola:

```js
import { render, screen } from '@testing-library/react'
import Note from './Note'

test('renders content', () => {
  const note = {
    content: 'Component testing is done with react-testing-library',
    important: true
  }

  render(<Note note={note} />)

  const element = screen.getByText('Component testing is done with react-testing-library')

  screen.debug(element)  // highlight-line

  expect(element).toBeDefined()
})
```

Ahora el HTML del elemento que queríamos ver se imprime:

```js
  <li
    class="note"
  >
    Component testing is done with react-testing-library
    <button>
      make not important
    </button>
  </li>
```

### Clicando botones en las pruebas

Además de mostrar el contenido, el componente <i>Note</i> también se asegura de que cuando se clica al botón asociado con la nota, se llama a la función del controlador de eventos _toggleImportance_.

Instalemos la librería [user-event](https://testing-library.com/docs/user-event/intro) que facilita un poco la simulación del input del usuario:

```bash
npm install --save-dev @testing-library/user-event
```

La prueba de esta funcionalidad se puede lograr así:

```js
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event' // highlight-line
import Note from './Note'

// ...

test('clicking the button calls event handler once', async () => {
  const note = {
    content: 'Component testing is done with react-testing-library',
    important: true
  }

  const mockHandler = vi.fn()  // highlight-line

  render(
    <Note note={note} toggleImportance={mockHandler} />  // highlight-line
  )

  const user = userEvent.setup()  // highlight-line
  const button = screen.getByText('make not important')  // highlight-line
  await user.click(button)  // highlight-line

  expect(mockHandler.mock.calls).toHaveLength(1)  // highlight-line
})
```

Hay algunas cosas interesantes relacionadas con esta prueba. El controlador de eventos es la función [mock](https://vitest.dev/api/mock) definida con Vitest:

```js
const mockHandler = vi.fn()
```

Se inicia una [session](https://testing-library.com/docs/user-event/setup/) (sesión) para interactuar con el componente renderizado:

```js
const user = userEvent.setup()
```

La prueba encuentra el botón <i>basada en el texto</i> del componente renderizado y hace clic en el elemento:

```js
const button = screen.getByText('make not important')
await user.click(button)
```

El clic ocurre con el método [click](https://testing-library.com/docs/user-event/convenience/#click) de la librería userEvent.

La expectativa de la prueba utiliza [toHaveLength](https://vitest.dev/api/expect.html#tohavelength) para verificar que la <i>mock function</i> (función simulada) se haya llamado exactamente una vez:

```js
expect(mockHandler.mock.calls).toHaveLength(1)
```

Las llamadas a la mock function son guardadas en el array [mock.calls](https://vitest.dev/api/mock#mock-calls) dentro del objeto de la mock function.

[Mock objects and functions](https://es.wikipedia.org/wiki/Objeto_simulado) (Objetos y funciones simulados) son componentes [stub](https://es.wikipedia.org/wiki/Stub) (código auxiliar) comúnmente utilizados en pruebas que se utilizan para reemplazar las dependencias de los componentes que se están probando. Los simulacros permiten devolver respuestas codificadas de manera rígida y verificar cuántas veces se llaman las funciones simuladas y con qué parámetros.

En nuestro ejemplo, la función simulada es una opción perfecta ya que se puede utilizar fácilmente para verificar que el método se llame exactamente una vez.

### Pruebas para el componente <i>Togglable</i>

Escribamos algunas pruebas para el componente <i>Togglable</i>. Agreguemos el nombre de clase CSS <i>togglableContent</i> al div que devuelve los componentes hijos.

```js
const Togglable = forwardRef((props, ref) => {
  // ...

  return (
    <div>
      <div style={hideWhenVisible}>
        <button onClick={toggleVisibility}>
          {props.buttonLabel}
        </button>
      </div>
      <div style={showWhenVisible} className="togglableContent"> // highlight-line
        {props.children}
        <button onClick={toggleVisibility}>cancel</button>
      </div>
    </div>
  )
})
```

Las pruebas se muestran a continuación:

```js
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import Togglable from './Togglable'

describe('<Togglable />', () => {
  let container

  beforeEach(() => {
    container = render(
      <Togglable buttonLabel="show...">
        <div className="testDiv" >
          togglable content
        </div>
      </Togglable>
    ).container
  })

  test('renders its children', async () => {
    await screen.findAllByText('togglable content')
  })

  test('at start the children are not displayed', () => {
    const div = container.querySelector('.togglableContent')
    expect(div).toHaveStyle('display: none')
  })

  test('after clicking the button, children are displayed', async () => {
    const user = userEvent.setup()
    const button = screen.getByText('show...')
    await user.click(button)

    const div = container.querySelector('.togglableContent')
    expect(div).not.toHaveStyle('display: none')
  })
})
```

La función _beforeEach_ se llama antes de cada prueba, la cual luego renderiza el componente <i>Togglable</i> y guarda el campo _container_ del valor devuelto.

La primera prueba verifica que el componente <i>Togglable</i> renderiza su componente hijo.

```js
<div className="testDiv">
  togglable content
</div>
```

Las pruebas restantes utilizan el método [toHaveStyle](https://www.npmjs.com/package/@testing-library/jest-dom#tohavestyle) para verificar que el componente hijo del componente <i>Togglable</i> no es visible inicialmente, comprobando que el estilo del elemento <i>div</i> contiene _{ display: 'none' }_. Otra prueba verifica que cuando se presiona el botón, el componente es visible, lo que significa que el estilo para ocultarlo <i>ya no está</i> asignado al componente.

Agreguemos también una prueba que se pueda usar para verificar que el contenido visible se puede ocultar haciendo clic en el segundo botón del componente:

```js
describe('<Togglable />', () => {

  // ...

  test('toggled content can be closed', async () => {
    const user = userEvent.setup()
    const button = screen.getByText('show...')
    await user.click(button)

    const closeButton = screen.getByText('cancel')
    await user.click(closeButton)

    const div = container.querySelector('.togglableContent')
    expect(div).toHaveStyle('display: none')
  })
})
```

### Probando los formularios

Ya usamos la función _click_ de [user-event](https://testing-library.com/docs/user-event/intro/) en nuestras pruebas anteriores para hacer clic en los botones.

```js
const user = userEvent.setup()
const button = screen.getByText('show...')
await user.click(button)
```

También podemos simular la entrada de texto con <i>userEvent</i>.

Hagamos una prueba para el componente <i>NoteForm</i>. El código del componente es el siguiente:

```js
import { useState } from 'react'

const NoteForm = ({ createNote }) => {
  const [newNote, setNewNote] = useState('')

  const handleChange = (event) => {
    setNewNote(event.target.value)
  }

  const addNote = (event) => {
    event.preventDefault()
    createNote({
      content: newNote,
      important: true,
    })

    setNewNote('')
  }

  return (
    <div className="formDiv">
      <h2>Create a new note</h2>

      <form onSubmit={addNote}>
        <input
          value={newNote}
          onChange={handleChange}
        />
        <button type="submit">save</button>
      </form>
    </div>
  )
}

export default NoteForm
```

El formulario funciona llamando a la función recibida como props _createNote_, con los detalles de la nueva nota.

La prueba es la siguiente:

```js
import { render, screen } from '@testing-library/react'
import NoteForm from './NoteForm'
import userEvent from '@testing-library/user-event'

test('<NoteForm /> updates parent state and calls onSubmit', async () => {
  const createNote = vi.fn()
  const user = userEvent.setup()

  render(<NoteForm createNote={createNote} />)

  const input = screen.getByRole('textbox')
  const sendButton = screen.getByText('save')

  await user.type(input, 'testing a form...')
  await user.click(sendButton)

  expect(createNote.mock.calls).toHaveLength(1)
  expect(createNote.mock.calls[0][0].content).toBe('testing a form...')
})
```

Las pruebas tienen acceso al campo de input utilizando la función [getByRole](https://testing-library.com/docs/queries/byrole).

El método [type](https://testing-library.com/docs/user-event/utility#type) de userEvent se utiliza para escribir texto en el campo de input.

La primera expectativa de la prueba asegura que al enviar el formulario el método _createNote_ es llamado.
La segunda expectativa verifica que el controlador de eventos se llama con los parámetros correctos, es decir, que se crea una nota con el contenido correcto cuando se llena el formulario.

Vale la pena mencionar que el buen viejo _console.log_ funciona como de costumbre en las pruebas. Por ejemplo, si quieres ver cómo se ven las llamadas almacenadas por el objeto simulado, puedes hacer lo siguiente:

```js
test('<NoteForm /> updates parent state and calls onSubmit', async() => {
  const user = userEvent.setup()
  const createNote = vi.fn()

  render(<NoteForm createNote={createNote} />)

  const input = screen.getByRole('textbox')
  const sendButton = screen.getByText('save')

  await user.type(input, 'testing a form...')
  await user.click(sendButton)

  console.log(createNote.mock.calls) // highlight-line
})
```

En el medio de la ejecución de las pruebas, lo siguiente se imprime en la consola:

```
[ [ { content: 'testing a form...', important: true } ] ]
```

### Sobre la búsqueda de elementos

Supongamos que el formulario tiene dos campos de input.

```js
const NoteForm = ({ createNote }) => {
  // ...

  return (
    <div className="formDiv">
      <h2>Create a new note</h2>

      <form onSubmit={addNote}>
        <input
          value={newNote}
          onChange={handleChange}
        />
        // highlight-start
        <input
          value={...}
          onChange={...}
        />
        // highlight-end
        <button type="submit">save</button>
      </form>
    </div>
  )
}
```

Ahora la forma en la que nuestra prueba encuentra el campo de input

```js
const input = screen.getByRole('textbox')
```

causaría un error:

![error de node que muestra dos elementos con rol textbox ya que usamos getByRole](../../images/5/40.png)

El mensaje de error sugiere utilizar <i>getAllByRole</i>. El test podría arreglarse de la siguiente forma:

```js
const inputs = screen.getAllByRole('textbox')

await user.type(inputs[0], 'testing a form...')
```

El método <i>getAllByRole</i> ahora devuelve un array y el campo de input correcto es el primer elemento del array. Sin embargo, este enfoque es un poco sospechoso ya que depende del orden de los campos de input.

A menudo, los campos de input tienen un texto de <i>placeholder</i> que indica al usuario qué tipo de input se espera. Agreguemos un placeholder a nuestro formulario:

```js
const NoteForm = ({ createNote }) => {
  // ...

  return (
    <div className="formDiv">
      <h2>Create a new note</h2>

      <form onSubmit={addNote}>
        <input
          value={newNote}
          onChange={handleChange}
          placeholder='write note content here' // highlight-line 
        />
        <input
          value={...}
          onChange={...}
        />    
        <button type="submit">save</button>
      </form>
    </div>
  )
}
```

Ahora encontrar el campo de input correcto es fácil con el método [getByPlaceholderText](https://testing-library.com/docs/queries/byplaceholdertext):

```js
test('<NoteForm /> updates parent state and calls onSubmit', () => {
  const createNote = vi.fn()

  render(<NoteForm createNote={createNote} />) 

  const input = screen.getByPlaceholderText('write note content here') // highlight-line 
  const sendButton = screen.getByText('save')

  userEvent.type(input, 'testing a form...')
  userEvent.click(sendButton)

  expect(createNote.mock.calls).toHaveLength(1)
  expect(createNote.mock.calls[0][0].content).toBe('testing a form...')
})
```

La forma más flexible de encontrar elementos en pruebas es el método <i>querySelector</i> del objeto _container_, que es devuelto por _render_, como se mencionó [anteriormente en esta parte](/es/part5/probando_aplicaciones_react#busqueda-de-contenido-en-un-componente). Se puede usar cualquier selector CSS con este método para buscar elementos en las pruebas.

Por ejemplo, podríamos definir un _id_ único para el campo de input:

```js
const NoteForm = ({ createNote }) => {
  // ...

  return (
    <div className="formDiv">
      <h2>Create a new note</h2>

      <form onSubmit={addNote}>
        <input
          value={newNote}
          onChange={handleChange}
          id='note-input' // highlight-line 
        />
        <input
          value={...}
          onChange={...}
        />    
        <button type="submit">save</button>
      </form>
    </div>
  )
}
```

El elemento input ahora podría ser encontrado en la prueba de la siguiente manera:

```js
const { container } = render(<NoteForm createNote={createNote} />)

const input = container.querySelector('#note-input')
```

Sin embargo, nos adheriremos al enfoque de usar _getByPlaceholderText_ en la prueba.

Antes de continuar, analicemos un par de detalles. Supongamos que un componente renderiza texto en un elemento HTML de la siguiente manera:

```js
const Note = ({ note, toggleImportance }) => {
  const label = note.important
    ? 'make not important' : 'make important'

  return (
    <li className='note'>
      Your awesome note: {note.content} // highlight-line
      <button onClick={toggleImportance}>{label}</button>
    </li>
  )
}

export default Note
```

el método _getByText_ que la prueba utiliza <i>no</i> encuentra al elemento

```js
test('renders content', () => {
  const note = {
    content: 'Does not work anymore :(',
    important: true
  }

  render(<Note note={note} />)

  const element = screen.getByText('Does not work anymore :(')

  expect(element).toBeDefined()
})
```

El método _getByText_ busca a un elemento que tenga **exactamente el mismo texto** que se proporciona como parámetro, y nada más. Si queremos buscar un elemento que <i>contenga</i> el texto, podríamos usar una opción adicional:

```js
const element = screen.getByText(
  'Does not work anymore :(', { exact: false }
)
```

o podríamos usar el método _findByText_:

```js
const element = await screen.findByText('Does not work anymore :(')
```

Es importante tener en cuenta que, a diferencia de los otros métodos _ByText_, _findByText_ ¡devuelve una promesa!

Existen situaciones en las que otra forma del método _queryByText_ es útil. El método devuelve el elemento pero <i>no genera una excepción</i> si no se lo encuentra.

Por ejemplo, podríamos utilizar el método para asegurarnos de que algo <i>no se está renderizando</i> en el componente:

```js
test('does not render this', () => {
  const note = {
    content: 'This is a reminder',
    important: true
  }

  render(<Note note={note} />)

  const element = screen.queryByText('do not want this thing to be rendered')
  expect(element).toBeNull()
})
```

### Cobertura de las pruebas

Podemos encontrar fácilmente la [cobertura](https://vitest.dev/guide/coverage.html#coverage) de nuestras pruebas ejecutándolas con el comando

```js
npm test -- --coverage
```

La primera vez que ejecutes el comando, Vitest te preguntará si quieres instalar la librería requerida _@vitest/coverage-v8_. Instálala y ejecuta el comando de nuevo:

![salida del terminal de cobertura de las pruebas](../../images/5/18new.png)

Se generará un informe HTML en el directorio <i>coverage</i>.
El informe nos dirá las líneas de código no probado en cada componente:

![reporte HTML de cobertura de las pruebas](../../images/5/19newer.png)

Puedes encontrar el código para nuestra aplicación actual en su totalidad en la rama <i>part5-8 </i> de [este repositorio de GitHub](https://github.com/fullstack-hy2020/part2-notes-frontend/tree/part5-8).

</div>

<div class="tasks">

### Ejercicios 5.13.-5.16.

#### 5.13: Pruebas de Listas de Blogs, paso 1

Realiza una prueba que verifique que el componente que muestra un blog muestre el título y el autor del blog, pero no muestre su URL o el número de likes por defecto

Agrega clases de CSS al componente para ayudar con las pruebas según sea necesario.

#### 5.14: Pruebas de Listas de Blogs, paso 2

Realiza una prueba que verifique que la URL del blog y el número de likes se muestran cuando se hace clic en el botón que controla los detalles mostrados.

#### 5.15: Pruebas de Listas de Blogs, paso 3

Realiza una prueba que garantice que si se hace clic dos veces en el botón <i>like</i>, se llama dos veces al controlador de eventos que el componente recibió como props.

#### 5.16: Pruebas de Listas de Blogs, paso 4

Haz una prueba para el nuevo formulario de blog. La prueba debe verificar que el formulario llama al controlador de eventos que recibió como props con los detalles correctos cuando se crea un nuevo blog.

</div>

<div class="content">

### Pruebas de integración del Frontend

En la parte anterior del material del curso, escribimos pruebas de integración para el backend que probaron su lógica y conectaron la base de datos a través de la API proporcionada por el backend. Al escribir estas pruebas, tomamos la decisión consciente de no escribir pruebas unitarias, ya que el código para ese backend es bastante simple, y es probable que los errores en nuestra aplicación ocurran en escenarios más complicados para los que las pruebas unitarias no son adecuadas.

Hasta ahora, todas nuestras pruebas para el frontend han sido pruebas unitarias que han validado el correcto funcionamiento de componentes individuales. Las pruebas unitarias son útiles a veces, pero incluso un conjunto completo de pruebas unitarias no es suficiente para validar que la aplicación funciona como un todo.

También podríamos realizar pruebas de integración para el frontend. Las pruebas de integración prueban la colaboración de múltiples componentes. Es considerablemente más difícil que las pruebas unitarias, ya que por ejemplo, tendríamos que simular datos del servidor.
Elegimos concentrarnos en hacer pruebas de extremo a extremo para probar toda la aplicación, en la que trabajaremos en el último capítulo de esta parte.

### Pruebas de instantáneas

Vitest ofrece una alternativa completamente diferente a las pruebas "tradicionales" llamada pruebas de [instantáneas](https://vitest.dev/guide/snapshot) o snapshot testing. La característica interesante de las pruebas de instantáneas es que los desarrolladores no necesitan definir ninguna prueba ellos mismos, simplemente es suficiente adoptar las pruebas de instantáneas.

El principio fundamental es comparar el código HTML definido por el componente después de que haya cambiado con el código HTML que existía antes de que se cambiara.

Si la instantánea nota algún cambio en el HTML definido por el componente, entonces es una nueva funcionalidad o un "error" causado por accidente. Las pruebas instantáneas notifican al desarrollador si cambia el código HTML del componente. El desarrollador tiene que decirle a Jest si el cambio fue deseado o no. Si el cambio en el código HTML es inesperado, implica la gran posibilidad de tener un error y el desarrollador puede darse cuenta de estos problemas potenciales fácilmente gracias a las pruebas de instantáneas.

</div>
