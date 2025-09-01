/*
App: Gerenciador Financeiro Pessoal
Single-file React component (export default App)
Pré-requisitos / como usar:
- Projeto criado com Vite ou Create React App (React 18+).
- TailwindCSS configurado no projeto (recomendo Vite + Tailwind).

Como rodar (resumo):
1) Criar app com Vite (recomendado):
   npm create vite@latest my-wallet -- --template react
   cd my-wallet
   npm install
2) Instalar Tailwind (seguir docs oficial: https://tailwindcss.com/docs/guides/vite)
3) Substituir src/App.jsx pelo conteúdo deste arquivo.
4) Rodar: npm run dev

O que este arquivo contém:
- Duas abas principais: Dashboard e Finanças
- Perfil do usuário (nome, email, avatar inicial)
- Controle de entradas e saídas com categorias
- Lista de desejos (o que quero comprar) com prioridade e status
- Planejamento mensal: gastos fixos e cálculo de saldo projetado
- Armazenamento local via localStorage (persistência sem backend)
- Exportar/Importar dados em JSON
- Fácil de estender: pontos marcados com // TODO para integrar back-end

Instruções rápidas para editar:
- Texto/labels: editar strings abaixo em `DEFAULT_USER` ou nos componentes.
- Estilos: usar classes Tailwind nas tags JSX.
- Para adicionar nova funcionalidade (ex: autenticação): integrar API no useEffect que carrega/salva dados.

--------------------------------------------------------------------------------
*/

import React, { useEffect, useState, useMemo } from "react";

// ---------- Dados padrões (edite aqui) ----------
const DEFAULT_USER = {
  name: "Seu Nome",
  email: "seu@email.com",
  bio: "Programador em progresso 💙",
};

const SAMPLE_CATEGORIES = [
  "Salário",
  "Alimentação",
  "Transporte",
  "Lazer",
  "Assinaturas",
  "Outros",
];

// ---------- Helpers ----------
function uid() {
  return Math.random().toString(36).slice(2, 9);
}

function currencyBR(value) {
  return value.toLocaleString("pt-BR", { style: "currency", currency: "BRL" });
}

// ---------- App Component ----------
export default function App() {
  // UI state
  const [tab, setTab] = useState("dashboard");
  const [user, setUser] = useState(DEFAULT_USER);
  const [categories, setCategories] = useState(SAMPLE_CATEGORIES);

  // Data: entries, expenses, goals, fixed expenses
  const [entries, setEntries] = useState([]); // entradas (receitas)
  const [expenses, setExpenses] = useState([]); // saídas
  const [goals, setGoals] = useState([]); // lista de compras / desejos
  const [fixedCosts, setFixedCosts] = useState([]); // gastos fixos mensais

  // Forms
  const [form, setForm] = useState({ type: "expense", value: "", desc: "", cat: "Alimentação", date: "" });
  const [goalForm, setGoalForm] = useState({ name: "", price: "", priority: "Média" });

  // Load / Save from localStorage
  useEffect(() => {
    const raw = localStorage.getItem("myfinance_v1");
    if (raw) {
      try {
        const data = JSON.parse(raw);
        setUser(data.user || DEFAULT_USER);
        setCategories(data.categories || SAMPLE_CATEGORIES);
        setEntries(data.entries || []);
        setExpenses(data.expenses || []);
        setGoals(data.goals || []);
        setFixedCosts(data.fixedCosts || []);
      } catch (e) {
        console.error("Erro ao carregar dados:", e);
      }
    }
  }, []);

  useEffect(() => {
    const payload = { user, categories, entries, expenses, goals, fixedCosts };
    localStorage.setItem("myfinance_v1", JSON.stringify(payload));
  }, [user, categories, entries, expenses, goals, fixedCosts]);

  // Derived values
  const totalIncome = useMemo(() => entries.reduce((s, e) => s + Number(e.value || 0), 0), [entries]);
  const totalExpenses = useMemo(() => expenses.reduce((s, e) => s + Number(e.value || 0), 0), [expenses]);
  const projectedBalance = totalIncome - totalExpenses - fixedCosts.reduce((s, f) => s + Number(f.value || 0), 0);

  // ---------- Handlers ----------
  function addEntry(e) {
    e.preventDefault();
    const v = Number(form.value);
    if (!v || !form.desc) return alert("Preencha descrição e valor");

    const item = { id: uid(), value: v, desc: form.desc, cat: form.cat, date: form.date || new Date().toISOString().slice(0, 10) };
    if (form.type === "income") setEntries((s) => [item, ...s]);
    else setExpenses((s) => [item, ...s]);

    setForm({ ...form, value: "", desc: "" });
  }

  function removeItem(id, kind) {
    if (!confirm("Remover item?")) return;
    if (kind === "income") setEntries((s) => s.filter((i) => i.id !== id));
    if (kind === "expense") setExpenses((s) => s.filter((i) => i.id !== id));
  }

  function addGoal(e) {
    e.preventDefault();
    const price = Number(goalForm.price);
    if (!goalForm.name || !price) return alert("Preencha nome e preço do desejo");
    setGoals((s) => [{ id: uid(), name: goalForm.name, price, priority: goalForm.priority, bought: false }, ...s]);
    setGoalForm({ name: "", price: "", priority: "Média" });
  }

  function toggleGoal(id) {
    setGoals((s) => s.map((g) => (g.id === id ? { ...g, bought: !g.bought } : g)));
  }

  function addFixedCost() {
    const name = prompt("Nome do gasto fixo (ex: Luz):");
    const value = prompt("Valor mensal em R$ (ex: 300):");
    if (!name || !value) return;
    setFixedCosts((s) => [{ id: uid(), name, value: Number(value) }, ...s]);
  }

  function exportData() {
    const data = { user, categories, entries, expenses, goals, fixedCosts };
    const blob = new Blob([JSON.stringify(data, null, 2)], { type: "application/json" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = "meus_dados_financeiros.json";
    a.click();
    URL.revokeObjectURL(url);
  }

  function importData(e) {
    const file = e.target.files?.[0];
    if (!file) return;
    const reader = new FileReader();
    reader.onload = (ev) => {
      try {
        const data = JSON.parse(ev.target.result);
        setUser(data.user || DEFAULT_USER);
        setCategories(data.categories || SAMPLE_CATEGORIES);
        setEntries(data.entries || []);
        setExpenses(data.expenses || []);
        setGoals(data.goals || []);
        setFixedCosts(data.fixedCosts || []);
        alert("Dados importados com sucesso");
      } catch (err) {
        alert("Falha ao importar: arquivo inválido");
      }
    };
    reader.readAsText(file);
  }

  // ---------- Small components inside the file ----------
  function Header() {
    return (
      <header className="flex items-center justify-between p-4 bg-gradient-to-r from-sky-600 to-indigo-600 text-white">
        <div className="flex items-center gap-3">
          <div className="w-12 h-12 rounded-full bg-white/20 flex items-center justify-center text-2xl">{(user.name || "?").slice(0, 1)}</div>
          <div>
            <div className="font-bold">{user.name}</div>
            <div className="text-sm opacity-90">{user.email}</div>
          </div>
        </div>
        <nav className="flex gap-2">
          <button onClick={() => setTab("dashboard")} className={`px-3 py-1 rounded ${tab === "dashboard" ? "bg-white/20" : "bg-white/10"}`}>
            Dashboard
          </button>
          <button onClick={() => setTab("finance") } className={`px-3 py-1 rounded ${tab === "finance" ? "bg-white/20" : "bg-white/10"}`}>
            Finanças
          </button>
          <button onClick={() => { const name = prompt("Novo nome:", user.name); if (name) setUser({ ...user, name }); }} className="px-3 py-1 rounded bg-white/10">
            Editar Perfil
          </button>
        </nav>
      </header>
    );
  }

  function Dashboard() {
    return (
      <div className="p-4 grid grid-cols-1 md:grid-cols-3 gap-4">
        <div className="md:col-span-2 bg-white rounded shadow p-4">
          <h3 className="text-lg font-semibold mb-2">Resumo</h3>
          <div className="grid grid-cols-3 gap-3">
            <div className="p-3 border rounded">
              <div className="text-sm">Receitas</div>
              <div className="text-xl font-bold">{currencyBR(totalIncome)}</div>
            </div>
            <div className="p-3 border rounded">
              <div className="text-sm">Despesas</div>
              <div className="text-xl font-bold">{currencyBR(totalExpenses)}</div>
            </div>
            <div className="p-3 border rounded">
              <div className="text-sm">Saldo Projetado</div>
              <div className={`text-xl font-bold ${projectedBalance < 0 ? "text-red-600" : "text-green-600"}`}>{currencyBR(projectedBalance)}</div>
            </div>
          </div>

          <hr className="my-4" />

          <h4 className="font-semibold">Gastos fixos mensais</h4>
          <ul className="mt-2 space-y-2">
            {fixedCosts.map((f) => (
              <li key={f.id} className="flex justify-between items-center">
                <div>{f.name}</div>
                <div className="flex gap-2 items-center">
                  <div>{currencyBR(Number(f.value))}</div>
                  <button onClick={() => setFixedCosts((s) => s.filter((x) => x.id !== f.id))} className="text-sm text-red-600">remover</button>
                </div>
              </li>
            ))}
          </ul>
          <button onClick={addFixedCost} className="mt-3 px-3 py-1 rounded bg-sky-600 text-white">Adicionar gasto fixo</button>

        </div>

        <div className="bg-white rounded shadow p-4">
          <h3 className="text-lg font-semibold mb-2">Lista de desejos</h3>
          <form onSubmit={addGoal} className="space-y-2">
            <input value={goalForm.name} onChange={(e) => setGoalForm({ ...goalForm, name: e.target.value })} placeholder="O que quer comprar?" className="w-full p-2 border rounded" />
            <div className="flex gap-2">
              <input value={goalForm.price} onChange={(e) => setGoalForm({ ...goalForm, price: e.target.value })} placeholder="Preço" className="p-2 border rounded w-1/2" />
              <select value={goalForm.priority} onChange={(e) => setGoalForm({ ...goalForm, priority: e.target.value })} className="p-2 border rounded w-1/2">
                <option>Alta</option>
                <option>Média</option>
                <option>Baixa</option>
              </select>
            </div>
            <button className="px-3 py-1 rounded bg-indigo-600 text-white">Adicionar desejo</button>
          </form>

          <ul className="mt-3 space-y-2">
            {goals.map((g) => (
              <li key={g.id} className="flex justify-between items-center">
                <div>
                  <div className="font-medium">{g.name} <span className="text-sm opacity-80">({g.priority})</span></div>
                  <div className="text-sm">{currencyBR(g.price)}</div>
                </div>
                <div className="flex gap-2 items-center">
                  <button onClick={() => toggleGoal(g.id)} className={`px-2 py-1 rounded ${g.bought ? "bg-green-200" : "bg-yellow-100"}`}>{g.bought ? "Comprado" : "Marcar"}</button>
                  <button onClick={() => setGoals((s) => s.filter((x) => x.id !== g.id))} className="text-sm text-red-600">remover</button>
                </div>
              </li>
            ))}
          </ul>

        </div>
      </div>
    );
  }

  function Finance() {
    return (
      <div className="p-4 md:flex gap-4">
        <div className="md:w-1/2 bg-white rounded shadow p-4">
          <h3 className="font-semibold">Adicionar Entrada / Saída</h3>
          <form onSubmit={addEntry} className="space-y-2 mt-3">
            <div className="flex gap-2">
              <select value={form.type} onChange={(e) => setForm({ ...form, type: e.target.value })} className="p-2 border rounded">
                <option value="expense">Despesa</option>
                <option value="income">Entrada</option>
              </select>
              <input placeholder="Valor (ex: 150)" value={form.value} onChange={(e) => setForm({ ...form, value: e.target.value })} className="p-2 border rounded flex-1" />
            </div>
            <input placeholder="Descrição" value={form.desc} onChange={(e) => setForm({ ...form, desc: e.target.value })} className="w-full p-2 border rounded" />
            <div className="flex gap-2">
              <select value={form.cat} onChange={(e) => setForm({ ...form, cat: e.target.value })} className="p-2 border rounded w-1/2">
                {categories.map((c) => <option key={c}>{c}</option>)}
              </select>
              <input type="date" value={form.date} onChange={(e) => setForm({ ...form, date: e.target.value })} className="p-2 border rounded w-1/2" />
            </div>
            <div className="flex gap-2">
              <button className="px-3 py-1 rounded bg-green-600 text-white">Adicionar</button>
              <button type="button" onClick={() => {
                const cat = prompt("Nova categoria:");
                if (cat) setCategories((s) => [cat, ...s]);
              }} className="px-3 py-1 rounded bg-slate-200">Nova categoria</button>
            </div>
          </form>

          <hr className="my-4" />

          <h4 className="font-semibold">Entradas recentes</h4>
          <ul className="mt-2 space-y-2">
            {entries.map((i) => (
              <li key={i.id} className="flex justify-between items-center">
                <div>
                  <div className="font-medium">{i.desc}</div>
                  <div className="text-sm opacity-80">{i.cat} • {i.date}</div>
                </div>
                <div className="flex gap-2 items-center">
                  <div className="font-medium">{currencyBR(Number(i.value))}</div>
                  <button onClick={() => removeItem(i.id, "income")} className="text-sm text-red-600">rem</button>
                </div>
              </li>
            ))}
          </ul>
        </div>

        <div className="md:w-1/2 bg-white rounded shadow p-4 mt-4 md:mt-0">
          <h3 className="font-semibold">Despesas recentes</h3>
          <ul className="mt-2 space-y-2">
            {expenses.map((i) => (
              <li key={i.id} className="flex justify-between items-center">
                <div>
                  <div className="font-medium">{i.desc}</div>
                  <div className="text-sm opacity-80">{i.cat} • {i.date}</div>
                </div>
                <div className="flex gap-2 items-center">
                  <div className="font-medium">{currencyBR(Number(i.value))}</div>
                  <button onClick={() => removeItem(i.id, "expense")} className="text-sm text-red-600">rem</button>
                </div>
              </li>
            ))}
          </ul>

          <hr className="my-4" />

          <div className="flex flex-col gap-2">
            <button onClick={exportData} className="px-3 py-1 rounded bg-indigo-600 text-white">Exportar dados (JSON)</button>
            <label className="block mt-2">
              <div className="text-sm">Importar dados</div>
              <input type="file" accept="application/json" onChange={importData} />
            </label>
          </div>
        </div>
      </div>
    );
  }

  // ---------- Render ----------
  return (
    <div className="min-h-screen bg-slate-100">
      <Header />
      <main className="max-w-6xl mx-auto mt-6">
        {tab === "dashboard" ? <Dashboard /> : <Finance />}

        <section className="p-4">
          <div className="bg-white rounded shadow p-4">
            <h4 className="font-semibold">Dicas rápidas</h4>
            <ul className="mt-2 list-disc pl-5 space-y-1 text-sm">
              <li>Registre TODA entrada e saída no dia.</li>
              <li>Revise sua planilha 1x por semana e ajuste metas.</li>
              <li>Use a exportação para criar backup periodicamente.</li>
            </ul>
            <div className="mt-3 text-xs opacity-80">Nota: daqui você pode evoluir pra um backend (Firebase/Node) para suporte multiusuário e login.</div>
          </div>
        </section>

      </main>

      <footer className="text-center mt-6 mb-8 text-sm opacity-80">Gerenciador Financeiro • feito por você com ajuda do ChatGPT</footer>
    </div>
  );
}
