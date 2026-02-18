npx create-react-app react-test - создать приложение

npm install -g npm@7.22.0 - обновить node.js

npm start - запустить проект
npm install prop-types - определение свойств параметров

Hooks
Useful site: usehooks.com

Кортеж - массив с заранее определенными элементами

useState

  const [counter, setCounter] = useState(0)

  function increment() {
    setCounter(counter + 1)
  }
  function decrement() {
    setCounter(counter - 1)
  }

  return (
    <div>
      <h1>Counter {counter}</h1>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
    </div>
  )

Чтобы увеличивать больше чем на 1:
  function increment() {
    setCounter((prev) => {
      return prev + 1
    })
    setCounter((counter) => counter + 1)
  }

Если useState при инициализации вызывает функцию:
useState(computeInitialState())

то он будет вызывать ее каждый раз при рендеренге и чтобы этого избежать и чтобы он вызвал ее только 1 раз надо передать туда колбек:
useState(() => { return computeInitialState()})

Если нужно обновить одно поле в сложном состоянии:
  const [state, setState] = useState({
    title: 'Counter',
    date: Date.now(),
  })

  function updateTitle(newTitle) {
    setState((prev) => {
      return { ...prev, title: newTitle }
    })
  }

  return (
    <div>
      <button onClick={() => updateTitle('new title')}>Decrement</button>
      <pre>{JSON.stringify(state, null, 2)}</pre>
    </div>
  )

useEffect
 - вызывается при рендеринге компонента

Выполнится 1 раз после рендеринга компонента
  useEffect(() => {
    console.log('render once')
  }, [])

В таком хуке можно. определить слушателя единожды:
  const [pos, setPos] = useState({
    x: 0,
    y: 0,
  })

  useEffect(() => {
    window.addEventListener('mousemove', (event) => {
      setPos({
        x: event.clientX,
        y: event.clientY,
      })
    })
  }, [])

  return (
    <div>
      <pre>{JSON.stringify(pos, null, 2)}</pre>
    </div>
  )

Чтобы удалить его при завершении действия эффекта. По сути это будет при удалении компонента но в обычном эффекте она будет вызываться всякий раз когда мы будем заново заходить в этот колбек:
const [pos, setPos] = useState({
    x: 0,
    y: 0,
  })

  const mouseMoveListener = (event) => {
    setPos({
      x: event.clientX,
      y: event.clientY,
    })
  }

  useEffect(() => {
    window.addEventListener('mousemove', mouseMoveListener)

    return () => window.removeEventListener('mousemove', mouseMoveListener)
  }, [])

  return (
    <div>
      <pre>{JSON.stringify(pos, null, 2)}</pre>
    </div>
  )

useRef 
- для одного состояния. Сохраняет состояние, но не вызывает рендер.

Пример счетчика:
  const [value, setValue] = useState('init')
  const renderCount = useRef(0)

  useEffect(() => {
    renderCount.current++
  })

  return (
    <div>
      <h1>Rendercount {renderCount.current}</h1>
      <input
        type="text"
        onChange={(e) => setValue(e.target.value)}
        value={value}
      />
    </div>
  )
}

Пример ссылки на компонент:
  const [value, setValue] = useState('init')
  const renderCount = useRef(0)
  const inputRef = useRef(null)

  useEffect(() => {
    renderCount.current++
  })

  const focus = () => inputRef.current.focus()

  return (
    <div>
      <h1>Rendercount {renderCount.current}</h1>
      <input
        type="text"
        ref={inputRef}
        onChange={(e) => setValue(e.target.value)}
        value={value}
      />
      <button onClick={focus}>Focus</button>
    </div>
  )

Получение предыдущего состояния:
const [value, setValue] = useState('init')
  const renderCount = useRef(0)
  const inputRef = useRef(null)
  const prevState = useRef('')

  useEffect(() => {
    renderCount.current++
  })

  useEffect(() => {
    prevState.current = value
  }, [value])

  const focus = () => inputRef.current.focus()

  return (
    <div>
      <h1>Rendercount {renderCount.current}</h1>
      <h1>Prev state {prevState.current}</h1>
      <input
        type="text"
        ref={inputRef}
        onChange={(e) => setValue(e.target.value)}
        value={value}
      />
      <button onClick={focus}>Focus</button>
    </div>
  )
}

useMemo - кеширует вычисляемое свойтво
  const computed = useMemo(() => {
    return complexCumpute()
  }, [number])

Кеширует состояние объекта и не перерисовывает его при рендеренге, если он не изменился
  const styles = useMemo(({
    color: colored ? 'darkred' : 'black'
  }, [colored])

useCallback 
-  кеширует функцию и не вызывает ее при любом рендере

  const [colored, setColored] = useState(false)
  const [count, setCount] = useState(1)

  const styles = {
    color: colored ? 'darkred' : 'black',
  }

  const generateItems = useCallback(() => {
    return new Array(count).fill('').map((_, i) => `Elem ${i + 1}`)
  }, [count])

  return (
    <>
      <h1 style={styles}>Elements count: {count}</h1>
      <button onClick={() => setCount((prev) => prev + 1)}>Add</button>
      <button onClick={() => setColored((prev) => !prev)}>Change</button>
      <ItemList getItems={generateItems} />
    </>
  )
}

ItemList.js:
export default function ItemList({ getItems }) {
  const [items, setItems] = useState([])

  useEffect(() => {
    const newItems = getItems()
    setItems(newItems)
  }, [getItems])

  return (
    <ul>
      {items.map((i) => (
        <li key={i}>{i}</li>
      ))}
    </ul>
  )
}

useContext 
- позволяет передавать контекст между разными элементами
export const AlertContext = React.createContext()

function App() {
  const [alert, setAlert] = useState(false)
  const toggleAlert = () => setAlert((prev) => !prev)
  return (
    <AlertContext.Provider value={alert}>
      <>
        <Alert />
        <Main toggle={toggleAlert} />
      </>
    </AlertContext.Provider>
  )
}

Main.js:
export default function Main({ toggle }) {
  return (
    <>
      <h1>Hello in Context</h1>
      <button onClick={toggle}>Show alert</button>
    </>
  )
}

Alert.js:
export default function Alert() {
  const alert = useContext(AlertContext)

  if (!alert) return null

  return (
    <div>
      <h1>It's important alert</h1>
    </div>
  )
}

Хорошая организация:
AlertContext.js:
export const AlertContext = React.createContext()

export const useAlert = () => {
  return useContext(AlertContext)
}
export const AlertProvider = ({ children }) => {
  const [alert, setAlert] = useState(false)
  const toggle = () => setAlert((prev) => !prev)
  return (
    <AlertContext.Provider
      value={{
        visible: alert,
        toggle,
      }}
    >
      {children}
    </AlertContext.Provider>
  )
}

App.js:
function App() {
  return (
    <AlertProvider>
      <>
        <Alert />
        <Main />
      </>
    </AlertProvider>
  )
}

export default App

Main.js:
export default function Main() {
  const { toggle } = useAlert()
  return (
    <>
      <h1>Hello in Context</h1>
      <button onClick={toggle}>Show alert</button>
    </>
  )
}

Alert.js:
export default function Alert() {
  const alert = useAlert()

  if (!alert.visible) return null

  return (
    <div onClick={alert.toggle}>
      <h1>It's important alert</h1>
    </div>
  )
}

useReducer 
- принимает редьюсер и состояние и возвращается состояние и диспатч который позволяет взаимодействовать 
со стейт через редьюсер

AlertContext.js:
export const AlertContext = React.createContext()

export const useAlert = () => {
  return useContext(AlertContext)
}

const SHOW_ALERT = 'show'
const HIDE_ALERT = 'hide'

const reducer = (state, action) => {
  switch (action.type) {
    case SHOW_ALERT:
      return { ...state, visible: true, text: action.text }
    case HIDE_ALERT:
      return { ...state, visible: false }
    default:
      return state
  }
}

export const AlertProvider = ({ children }) => {
  const [state, dispatch] = useReducer(reducer, { visible: false, text: '' })

  const show = (text) => dispatch({ type: SHOW_ALERT, text })
  const hide = () => dispatch({ type: HIDE_ALERT })

  return (
    <AlertContext.Provider
      value={{
        visible: state.visible,
        text: state.text,
        show,
        hide,
      }}
    >
      {children}
    </AlertContext.Provider>
  )
}


App.js:
function App() {
  return (
    <AlertProvider>
      <>
        <Alert />
        <Main />
      </>
    </AlertProvider>
  )
}

export default App

Main.js:
export default function Main() {
  const { show } = useAlert()
  return (
    <>
      <h1>Hello in Context</h1>
      <button onClick={() => show('From main.js')}>Show alert</button>
    </>
  )
}

Alert.js:
export default function Alert() {
  const alert = useAlert()

  if (!alert.visible) return null

  return (
    <div onClick={alert.hide}>
      <h1>{alert.text}</h1>
    </div>
  )
}

Custom Huk

function useLogger(value) {
  useEffect(() => {
    console.log(value)
  }, [value])
}

function useInput(initVal) {
  const [value, setValue] = useState(initVal)

  const onChange = (event) => {
    setValue(event.target.value)
  }

  return { value, onChange }
}

function App() {
  const input = useInput('')
  const lastName = useInput('')

  useLogger(input.value)

  return (
    <div>
      <input type="text" {...input}></input>
      <h1>{input.value}</h1>
    </div>
  )
}

export default App

Еще пример:
unction useInput(initVal) {
  const [value, setValue] = useState(initVal)

  const onChange = (event) => {
    setValue(event.target.value)
  }

  const clear = () => setValue('')

  return { bind: { value, onChange }, value, clear }
}

function App() {
  const input = useInput('')

  return (
    <div>
      <input type="text" {...input.bind}></input>
      <button onClick={input.clear}>clear</button>
      <h1>{input.value}</h1>
    </div>
  )
}




