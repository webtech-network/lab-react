# TaskBoard — Kanban + Pomodoro (React)

Um quadro de tarefas estilo Kanban com timer Pomodoro integrado.  
Colunas: **Backlog → Doing → Done**.  
Recursos: CRUD de tarefas, filtros, busca, persistência em `localStorage`.

## ✨ Features
- Criar/editar/excluir tarefas (título, descrição, label, prioridade, prazo)
- Mover entre colunas (Backlog/Doing/Done)
- Timer **Pomodoro 25/5** por tarefa (iniciar/pausar/zerar)
- Busca e filtro por label/prioridade
- **Persistência** automática em `localStorage`
- **React Router v6** (rotas: `/`, `/new`, `/task/:id`)

## 🧰 Stack
- React 18 + Vite
- React Router v6
- Context API
- CSS Modules
- `nanoid` (IDs)

---

## 🚀 Começando

### 1) Pré-requisitos
```bash
node -v
npm -v
````

### 2) Criar o projeto (Vite)

```bash
npm create vite taskboard -- --template react
cd taskboard
npm install
```

### 3) Instalar dependências

```bash
npm install react-router-dom nanoid
```

### 4) Rodar

```bash
npm run dev
```

Acesse: [http://localhost:5173/](http://localhost:5173/)

---

## 📁 Estrutura sugerida

```
src/
├─ App.jsx
├─ main.jsx
├─ context/
│  └─ TasksContext.jsx
├─ components/
│  ├─ layout/
│  │  ├─ Navbar.jsx
│  │  └─ Navbar.module.css
│  ├─ board/
│  │  ├─ Board.jsx
│  │  └─ Column.jsx
│  ├─ task/
│  │  ├─ TaskCard.jsx
│  │  ├─ TaskForm.jsx
│  │  └─ TaskCard.module.css
│  └─ timer/
│     └─ PomodoroTimer.jsx
└─ styles/
   └─ globals.css (opcional)
```

---

## 🧩 Código essencial

> Crie/edite os arquivos a seguir conforme os blocos de código.

### `src/main.jsx`

```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.jsx'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
)
```

### `src/App.jsx`

```jsx
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom'
import { TaskProvider } from './context/TasksContext'
import Navbar from './components/layout/Navbar'
import Board from './components/board/Board'
import TaskForm from './components/task/TaskForm'

export default function App() {
  return (
    <Router>
      <TaskProvider>
        <Navbar />
        <div style={{ maxWidth: 960, margin: '0 auto', padding: '16px' }}>
          <Routes>
            <Route path="/" element={<Board />} />
            <Route path="/new" element={<TaskForm />} />
            <Route path="/task/:id" element={<TaskForm />} />
          </Routes>
        </div>
      </TaskProvider>
    </Router>
  )
}
```

### `src/context/TasksContext.jsx`

```jsx
import { createContext, useContext, useEffect, useMemo, useState } from 'react'
import { nanoid } from 'nanoid'

const TasksContext = createContext(null)

const DEFAULT_TASKS = [
  { id: nanoid(), title: 'Estudar React', description: 'Hooks e Router',
    label: 'study', priority: 'high', dueDate: '', status: 'backlog', pomodoros: 0, createdAt: Date.now() },
  { id: nanoid(), title: 'Protótipo UI', description: 'Wireframe do board',
    label: 'design', priority: 'medium', dueDate: '', status: 'doing', pomodoros: 1, createdAt: Date.now() }
]

export function TaskProvider({ children }) {
  const [tasks, setTasks] = useState(() => {
    const saved = localStorage.getItem('tasks@taskboard')
    return saved ? JSON.parse(saved) : DEFAULT_TASKS
  })

  useEffect(() => {
    localStorage.setItem('tasks@taskboard', JSON.stringify(tasks))
  }, [tasks])

  function addTask(payload) {
    setTasks(prev => [{ id: nanoid(), createdAt: Date.now(), pomodoros: 0, status: 'backlog', ...payload }, ...prev])
  }

  function updateTask(id, patch) {
    setTasks(prev => prev.map(t => (t.id === id ? { ...t, ...patch } : t)))
  }

  function removeTask(id) {
    setTasks(prev => prev.filter(t => t.id !== id))
  }

  function moveTask(id, direction) {
    const order = ['backlog', 'doing', 'done']
    setTasks(prev => prev.map(t => {
      if (t.id !== id) return t
      const idx = order.indexOf(t.status)
      const next = direction === 'left' ? Math.max(idx - 1, 0) : Math.min(idx + 1, order.length - 1)
      return { ...t, status: order[next] }
    }))
  }

  function incrementPomodoros(id) {
    setTasks(prev => prev.map(t => (t.id === id ? { ...t, pomodoros: (t.pomodoros || 0) + 1 } : t)))
  }

  const value = useMemo(() => ({
    tasks, addTask, updateTask, removeTask, moveTask, incrementPomodoros
  }), [tasks])

  return <TasksContext.Provider value={value}>{children}</TasksContext.Provider>
}

export function useTasks() {
  const ctx = useContext(TasksContext)
  if (!ctx) throw new Error('useTasks must be used within <TaskProvider>')
  return ctx
}
```

### `src/components/layout/Navbar.module.css`

```css
.navbar {
  display: flex; align-items: center; justify-content: space-between;
  padding: 12px 16px; border-bottom: 1px solid #eee;
}
.actions { display: flex; gap: 8px; }
a.btn {
  border: 1px solid #ccc; padding: 6px 10px; border-radius: 8px; text-decoration: none;
}
```

### `src/components/layout/Navbar.jsx`

```jsx
import { Link, useLocation } from 'react-router-dom'
import styles from './Navbar.module.css'

export default function Navbar() {
  const { pathname } = useLocation()
  return (
    <header className={styles.navbar}>
      <Link to="/">TaskBoard</Link>
      <div className={styles.actions}>
        {pathname !== '/new' && <Link className="btn" to="/new">+ Nova tarefa</Link>}
        <a className="btn" href="https://github.com" target="_blank" rel="noreferrer">GitHub</a>
      </div>
    </header>
  )
}
```

### `src/components/board/Board.jsx`

```jsx
import Column from './Column'
import { useMemo, useState } from 'react'
import { useTasks } from '../../context/TasksContext'

export default function Board() {
  const { tasks } = useTasks()
  const [query, setQuery] = useState('')
  const [label, setLabel] = useState('all')

  const filtered = useMemo(() => {
    return tasks.filter(t => {
      const matchesQuery = t.title.toLowerCase().includes(query.toLowerCase())
      const matchesLabel = label === 'all' ? true : t.label === label
      return matchesQuery && matchesLabel
    })
  }, [tasks, query, label])

  return (
    <>
      <div style={{ display: 'flex', gap: 8, marginBottom: 12 }}>
        <input placeholder="Buscar..." value={query} onChange={e => setQuery(e.target.value)} />
        <select value={label} onChange={e => setLabel(e.target.value)}>
          <option value="all">Todas labels</option>
          <option value="study">study</option>
          <option value="design">design</option>
          <option value="dev">dev</option>
        </select>
      </div>

      <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr 1fr', gap: 16 }}>
        <Column title="Backlog" status="backlog" tasks={filtered} />
        <Column title="Doing"   status="doing"   tasks={filtered} />
        <Column title="Done"    status="done"    tasks={filtered} />
      </div>
    </>
  )
}
```

### `src/components/board/Column.jsx`

```jsx
import TaskCard from '../task/TaskCard'

export default function Column({ title, status, tasks }) {
  const list = tasks.filter(t => t.status === status)
  return (
    <section>
      <h3>{title} ({list.length})</h3>
      <div style={{ display: 'grid', gap: 8 }}>
        {list.map(t => <TaskCard key={t.id} task={t} />)}
      </div>
    </section>
  )
}
```

### `src/components/task/TaskCard.module.css`

```css
.card {
  border: 1px solid #e5e7eb; border-radius: 10px; padding: 10px; background: #fff;
  display: grid; gap: 6px;
}
.meta { display: flex; gap: 8px; font-size: 12px; color: #6b7280; }
.actions { display: flex; gap: 6px; }
.btn { border: 1px solid #d1d5db; background: #f9fafb; padding: 4px 8px; border-radius: 8px; }
.badge { border: 1px solid #d1d5db; padding: 2px 6px; border-radius: 999px; font-size: 12px; }
```

### `src/components/task/TaskCard.jsx`

```jsx
import { Link } from 'react-router-dom'
import { useTasks } from '../../context/TasksContext'
import PomodoroTimer from '../timer/PomodoroTimer'
import styles from './TaskCard.module.css'

export default function TaskCard({ task }) {
  const { removeTask, moveTask } = useTasks()

  return (
    <article className={styles.card}>
      <strong>{task.title}</strong>
      {task.description && <p>{task.description}</p>}

      <div className={styles.meta}>
        {task.label && <span className={styles.badge}>{task.label}</span>}
        {task.priority && <span className={styles.badge}>prio: {task.priority}</span>}
        {task.pomodoros > 0 && <span className={styles.badge}>🍅 {task.pomodoros}</span>}
      </div>

      <PomodoroTimer taskId={task.id} />

      <div className={styles.actions}>
        <button className={styles.btn} onClick={() => moveTask(task.id, 'left')}>←</button>
        <button className={styles.btn} onClick={() => moveTask(task.id, 'right')}>→</button>
        <Link className={styles.btn} to={`/task/${task.id}`}>Editar</Link>
        <button className={styles.btn} onClick={() => removeTask(task.id)}>Excluir</button>
      </div>
    </article>
  )
}
```

### `src/components/timer/PomodoroTimer.jsx`

```jsx
import { useEffect, useRef, useState } from 'react'
import { useTasks } from '../../context/TasksContext'

const WORK_MIN = 25
const BREAK_MIN = 5

export default function PomodoroTimer({ taskId }) {
  const { incrementPomodoros } = useTasks()
  const [isRunning, setIsRunning] = useState(false)
  const [isBreak, setIsBreak] = useState(false)
  const [seconds, setSeconds] = useState(WORK_MIN * 60)
  const intervalRef = useRef(null)

  useEffect(() => {
    if (!isRunning) return
    intervalRef.current = setInterval(() => {
      setSeconds(prev => {
        if (prev <= 1) {
          clearInterval(intervalRef.current)
          if (!isBreak) incrementPomodoros(taskId)
          const nextIsBreak = !isBreak
          setIsBreak(nextIsBreak)
          setSeconds((nextIsBreak ? BREAK_MIN : WORK_MIN) * 60)
          setIsRunning(false)
          return 0
        }
        return prev - 1
      })
    }, 1000)
    return () => clearInterval(intervalRef.current)
  }, [isRunning, isBreak, incrementPomodoros, taskId])

  const mm = String(Math.floor(seconds / 60)).padStart(2, '0')
  const ss = String(seconds % 60).padStart(2, '0')

  return (
    <div style={{ display: 'flex', gap: 8, alignItems: 'center' }}>
      <span>{isBreak ? 'Break' : 'Work'} {mm}:{ss}</span>
      <button onClick={() => setIsRunning(!isRunning)}>{isRunning ? 'Pausar' : 'Iniciar'}</button>
      <button onClick={() => { setIsRunning(false); setIsBreak(false); setSeconds(WORK_MIN * 60) }}>Reset</button>
    </div>
  )
}
```

### `src/components/task/TaskForm.jsx`

```jsx
import { useNavigate, useParams } from 'react-router-dom'
import { useTasks } from '../../context/TasksContext'
import { useEffect, useState } from 'react'

const EMPTY = { title: '', description: '', label: '', priority: 'medium', dueDate: '' }

export default function TaskForm() {
  const { tasks, addTask, updateTask } = useTasks()
  const navigate = useNavigate()
  const { id } = useParams()
  const [form, setForm] = useState(EMPTY)

  useEffect(() => {
    if (!id) return
    const found = tasks.find(t => t.id === id)
    if (found) setForm({ title: found.title, description: found.description || '', label: found.label || '', priority: found.priority || 'medium', dueDate: found.dueDate || '' })
  }, [id, tasks])

  function onSubmit(e) {
    e.preventDefault()
    if (!form.title.trim()) return
    if (id) updateTask(id, form)
    else addTask(form)
    navigate('/')
  }

  return (
    <form onSubmit={onSubmit} style={{ display: 'grid', gap: 10 }}>
      <h2>{id ? 'Editar tarefa' : 'Nova tarefa'}</h2>
      <input placeholder="Título" value={form.title} onChange={e => setForm({ ...form, title: e.target.value })} />
      <textarea placeholder="Descrição (opcional)" value={form.description} onChange={e => setForm({ ...form, description: e.target.value })} />
      <input placeholder="Label (ex: study, dev, design)" value={form.label} onChange={e => setForm({ ...form, label: e.target.value })} />
      <select value={form.priority} onChange={e => setForm({ ...form, priority: e.target.value })}>
        <option value="low">low</option>
        <option value="medium">medium</option>
        <option value="high">high</option>
      </select>
      <input type="date" value={form.dueDate} onChange={e => setForm({ ...form, dueDate: e.target.value })} />
      <div style={{ display: 'flex', gap: 8 }}>
        <button type="submit">{id ? 'Salvar' : 'Criar'}</button>
        <button type="button" onClick={() => navigate(-1)}>Cancelar</button>
      </div>
    </form>
  )
}
```

---

## 🧪 Checklist funcional

* [ ] Criar tarefa
* [ ] Editar tarefa
* [ ] Excluir tarefa
* [ ] Mover tarefa entre colunas
* [ ] Iniciar/pausar/zerar Pomodoro
* [ ] Persistir no `localStorage`
* [ ] Buscar por texto / filtrar por label

---

## 🧭 Extensões (opcionais)

* Drag & drop com **@dnd-kit** ou **react-beautiful-dnd**
* Tema **dark mode**
* Testes com **Vitest + React Testing Library**
* TypeScript

---

## 📄 Licença

MIT

```

---

se quiser, eu também gero um **repositório inicial (zip)** com essa estrutura e os arquivos já preenchidos pra você só abrir e rodar. Quer?
```
