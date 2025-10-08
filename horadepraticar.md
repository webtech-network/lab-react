# 🚀 Guia passo a passo — React (Vite) + Router + Página Labs (GitHub)

## 0) Preparar o projeto

1. **Verifique Node e npm**

```bash
node -v
npm -v
```

**Rodapé:** checa se o ambiente está pronto.

2. **Crie o projeto com Vite**

```bash
npm create vite@latest web_tech_page -- --template react
cd web_tech_page
code . # Entra no VS Code
npm install # Dentro do VS Code execute isso
npm run dev
```

**Rodapé:** Vite cria o esqueleto React moderno, com `src/`, `main.jsx`, `App.jsx`. O `--template react` aplica o template oficial.

---

## 1) Estrutura base de pastas

> Ação: dentro de `src/`, crie as pastas:

* `src/components/`
* `src/components/layout/`
* `src/components/pages/`
* `src/assets/` (para imagens, ícones, etc.)

**Rodapé:** separar por **responsabilidade** (layout, pages, componentes auxiliares) ajuda a escalar. `layout` = elementos recorrentes de “layout” (Navbar, Footer), `pages` = telas roteáveis.

---

## 2) Primeiro componente: “HelloWorld”

> Ação: crie `src/components/HelloWorld.jsx`:

```jsx
// src/components/HelloWorld.jsx
export default function HelloWorld() {
  return (
    <div>
      <h1>Hello, World!</h1>
    </div>
  );
}
```

**Rodapé:** componente **funcional** (padrão atual). Nome em **PascalCase** (`HelloWorld`) é a convenção para componentes React.

> Ação: (opcional) use no `App.jsx` só para validar o ambiente.

```jsx
import HelloWorld from "./components/HelloWorld";

// src/App.jsx
export default function App() {
  return <HelloWorld />
}
```

**Rodapé:** valida que o bundler e Hot Reload estão funcionando.

---

## 3) Navbar (layout) + CSS Module

> Ação: crie `src/components/layout/Navbar.module.css` (mínimo para ver estilo isolado):

```css
/* src/components/layout/Navbar.module.css */
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  border-bottom: 1px solid #eee;
}
.navbar ul { display: flex; gap: 12px; list-style: none; padding: 0; margin: 0; }
.navbar a { text-decoration: none; }
```

**Rodapé:** CSS Modules dão **escopo local** e evitam conflitos de classe.

> Ação: crie `src/components/layout/Navbar.jsx`:

```jsx
// src/components/layout/Navbar.jsx
import { Link } from 'react-router-dom';
import styles from './Navbar.module.css';

export default function Navbar() {
  return (
    <header className={styles.navbar}>
      <h1><Link to="/">WebTech PUC Minas</Link></h1>
      <nav>
        <ul>
          <li><Link to="/">Home</Link></li>
          <li><Link to="/labs">Labs</Link></li>
        </ul>
      </nav>
    </header>
  );
}
```

**Rodapé:** usa `Link` (Router) para navegação **SPA** sem recarregar a página. Importa `styles` do CSS Module, garantindo nomes únicos de classe.

---

## 4) Footer (layout)

> Ação: crie `src/components/layout/Footer.module.css`:

```css
/* src/components/layout/Footer.module.css */
.footer {
  padding: 16px;
  border-top: 1px solid #eee;
  text-align: center;
  font-size: 0.9rem;
}
```

> Ação: crie `src/components/layout/Footer.jsx`:

```jsx
// src/components/layout/Footer.jsx
import styles from './Footer.module.css';

export default function Footer() {
  return (
    <footer className={styles.footer}>
      © {new Date().getFullYear()} WebTech — Todos os direitos reservados.
    </footer>
  );
}
```

**Rodapé:** componente simples e **reutilizável**. Mantém consistência visual entre páginas.

---

## 5) Instalar e configurar React Router v6

> Abra outro terminal.
> Ação: instale:

```bash
npm install react-router-dom
```

> Ação: edite `src/App.jsx` para incluir rotas e o layout:

```jsx
// src/App.jsx
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import Navbar from './components/layout/Navbar';
import Footer from './components/layout/Footer';
import Home from './components/pages/Home';
import Labs from './components/pages/Labs';

export default function App() {
  return (
    <Router>
      <Navbar />
      <div style={{ padding: 16 }}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/labs" element={<Labs />} />
        </Routes>
      </div>
      <Footer />
    </Router>
  );
}
```

**Rodapé:** `Routes`/`Route` (v6) substituem `Switch`. `element={<Componente />}` é a nova API. O `Router` engloba toda a app para habilitar navegação SPA.

---

## 6) Página Home

> Ação: crie `src/components/pages/Home.module.css`:

```css
/* src/components/pages/Home.module.css */
.home_page {
  padding: 24px 0;
}
.home_page span {
  color: #3b82f6; /* azul */
  font-weight: 600;
}
```

> Ação: crie `src/components/pages/Home.jsx`:

```jsx
// src/components/pages/Home.jsx
import styles from './Home.module.css';

export default function Home() {
  return (
    <div className={styles.home_page}>
      <h1>Bem-vindo à <span>WebTech!</span></h1>
      <p>Este é o laboratório prático em React.</p>
    </div>
  );
}
```

**Rodapé:** página simples para rota `/`. CSS Module mantém estilos locais; `Home` segue **PascalCase**.

---

## 7) Componente com estado: Contador

> Ação: crie `src/components/Contador.jsx`:

```jsx
// src/components/Contador.jsx
import { useState } from 'react';

export default function Contador() {
  const [contador, setContador] = useState(0);

  return (
    <div>
      <p>Contagem: {contador}</p>
      <button onClick={() => setContador(c => c + 1)}>Incrementar</button>
    </div>
  );
}
```

**Rodapé:** `useState` adiciona **estado local**. A atualização via “função set” (`c => c + 1`) é segura e evita depender de valores antigos.

> Ação: (opcional) use o `Contador` na Home:

```jsx
// src/components/pages/Home.jsx
import styles from './Home.module.css';
import Contador from '../Contador';

export default function Home() {
  return (
    <div className={styles.home_page}>
      <h1>Bem-vindo à <span>WebTech!</span></h1>
      <Contador />
    </div>
  );
}
```

**Rodapé:** demonstra **composição** de componentes (Home usa Contador).

---

## 8) Página Labs com consumo de API (GitHub)

### 8.1) Card e subcomponentes (layout)

> Ação: crie `src/components/layout/LabCard.module.css`:

```css
/* src/components/layout/LabCard.module.css */
.lab_card {
  border: 1px solid #eee;
  border-radius: 12px;
  padding: 12px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.card_top {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.contributors { display: flex; gap: 8px; }
.card_footer {
  display: flex; align-items: center; justify-content: space-between;
}
.labels { display: flex; gap: 6px; }
```

> Ação: crie `src/components/layout/LabLabel.module.css`:

```css
/* src/components/layout/LabLabel.module.css */
.lab_label {
  font-size: 0.75rem;
  padding: 2px 6px;
  border: 1px solid #ddd;
  border-radius: 8px;
}
```

> Ação: crie `src/components/layout/LabContributor.module.css`:

```css
/* src/components/layout/LabContributor.module.css */
.lab_contributor img {
  width: 28px; height: 28px; border-radius: 50%;
  display: block;
}
```

> Ação: crie `src/components/layout/LabLabel.jsx`:

```jsx
// src/components/layout/LabLabel.jsx
import styles from './LabLabel.module.css';

export default function LabLabel({ children }) {
  return <div className={styles.lab_label}>{children}</div>;
}
```

**Rodapé:** `LabLabel` é um **chip**/etiqueta simples e reutilizável.

> Ação: crie `src/components/layout/LabContributor.jsx`:

```jsx
// src/components/layout/LabContributor.jsx
import styles from './LabContributor.module.css';

export default function LabContributor({ contributor }) {
  return (
    <div className={styles.lab_contributor}>
      <a href={contributor.html_url} target="_blank" rel="noopener noreferrer">
        <img src={contributor.avatar_url} alt={contributor.login} />
      </a>
    </div>
  );
}
```

**Rodapé:** exibe avatar e link do contribuidor. `rel="noopener noreferrer"` é prática de **segurança** para links externos.

> Ação: crie `src/components/layout/LabCard.jsx`:

```jsx
// src/components/layout/LabCard.jsx
import LabLabel from './LabLabel';
import LabContributor from './LabContributor';
import styles from './LabCard.module.css';

export default function LabCard({ repo }) {
  return (
    <div className={styles.lab_card}>
      <div>
        <div className={styles.card_top}>
          <h5>{repo.name}</h5>
          <div className={styles.contributors}>
            {repo.contributors?.slice(0, 3).map(c => (
              <LabContributor key={c.id} contributor={c} />
            ))}
          </div>
        </div>
        <p>{repo.description}</p>
      </div>
      <div className={styles.card_footer}>
        <div className={styles.labels}>
          {repo.language && <LabLabel>{repo.language}</LabLabel>}
          {repo.stargazers_count > 0 && <LabLabel>⭐ {repo.stargazers_count}</LabLabel>}
        </div>
        <a href={repo.html_url} target="_blank" rel="noreferrer">Saiba mais</a>
      </div>
    </div>
  );
}
```

**Rodapé:** `LabCard` recebe `repo` (objeto da API) e compõe subcomponentes. `?.slice` evita erro quando `contributors` ainda não chegou.

---

### 8.2) Página Labs (fetch + ordenação + filtro)

> Ação: crie `src/components/pages/Lab.module.css`:

```css
/* src/components/pages/Lab.module.css */
.lab_page { padding: 24px 0; }
.description { color: #555; margin: 8px 0 16px; }
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
  gap: 12px;
}
```

> Ação: crie `src/components/pages/Labs.jsx`:

```jsx
// src/components/pages/Labs.jsx
import { useEffect, useState } from 'react';
import LabCard from '../layout/LabCard';
import styles from './Lab.module.css';

export default function Labs() {
  const [repos, setRepos] = useState([]);
  const githubUsername = 'seu-github';

  useEffect(() => {
    async function fetchData() {
      const res = await fetch(`https://api.github.com/users/${githubUsername}/repos`);
      const data = await res.json();

      // Busca contribuidores de cada repositório
      const reposWithContributors = await Promise.all(
        data.map(async (repo) => {
          const contributorsRes = await fetch(repo.contributors_url);
          const contributorsData = await contributorsRes.json();
          return { ...repo, contributors: contributorsData };
        })
      );

      // Filtra e ordena labs
      const filtered = reposWithContributors
        .filter((repo) => repo.name.startsWith('lab-'))
        .sort((a, b) => new Date(b.updated_at) - new Date(a.updated_at));

      setRepos(filtered);
    }

    fetchData();
  }, [githubUsername]);

  return (
    <div className={styles.lab_page}>
      <section>
        <h1>Labs</h1>
        <p className={styles.description}>
          Labs públicos da organização no GitHub (filtrados por prefixo <code>lab-</code>).
        </p>
        <div className={styles.grid}>
          {repos.map((repo) => (
            <LabCard key={repo.id} repo={repo} />
          ))}
        </div>
      </section>
    </div>
  );
}
```

**Rodapé:** `useEffect` faz o **fetch** na montagem. Em seguida, busca **contributors** de cada repo, filtra só nomes que começam com `lab-` e ordena por `updated_at`. A página renderiza uma **grid** de `LabCard`.

---

## 9) Rodar o projeto

> Ação:

```bash
npm run dev
```

Acesse: `http://localhost:5173/`

**Rodapé:** porta padrão do Vite. O Hot Reload atualiza a UI a cada edição.

---

## 10) Estrutura final esperada

```
src/
├── App.jsx
├── assets/
├── components/
│   ├── Contador.jsx
│   ├── HelloWorld.jsx
│   ├── layout/
│   │   ├── Footer.jsx
│   │   ├── Footer.module.css
│   │   ├── LabCard.jsx
│   │   ├── LabCard.module.css
│   │   ├── LabContributor.jsx
│   │   ├── LabContributor.module.css
│   │   ├── LabLabel.jsx
│   │   ├── LabLabel.module.css
│   │   ├── Navbar.jsx
│   │   └── Navbar.module.css
│   └── pages/
│       ├── Home.jsx
│       ├── Home.module.css
│       ├── Lab.module.css
│       └── Labs.jsx
└── main.jsx
```

**Rodapé:** organização por **feature/uso**: `layout` para elementos estruturais, `pages` para rotas, componentes pontuais na raiz de `components/`.

---

se quiser, eu compactei tudo isso em um **README.md pronto** (com esses mesmos blocos e rodapés). quer que eu gere o arquivo para você colar direto no repositório?
