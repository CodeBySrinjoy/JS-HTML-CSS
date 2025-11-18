# JS-HTML-CSS
//to do list
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Basic To‑Do List (Red Background)</title>
  <style>
    /* Red background as requested */
    :root{ --max-width:1100px; }
    html,body{ height:100%; }
    body{
      margin:0;
      font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
      background: linear-gradient(to bottom right, blue, red); /* deep red */
      display:flex;
      align-items:center;
      justify-content:center;
      padding:24px;
      color:#fff;
      -webkit-font-smoothing:antialiased;
    }

    .app{
      width:100%;
      max-width:var(--max-width);
      height:85vh;
      background: linear-gradient(to bottom right, blue, red);
      border-radius:12px;
      padding:18px;
      box-shadow: 0 8px 30px rgba(0,0,0,0.35);
      backdrop-filter: blur(4px);
      overflow:auto;
    }

    h1{ margin:0 0 12px 0; font-size:28px; }

    .controls{
      display:flex; gap:8px; margin-bottom:12px;
    }
    input[type="text"]{
      flex:1; padding:10px 12px; border-radius:8px; border:0; outline:none;
      font-size:20px;
    }
    button{
      padding:10px 12px; border-radius:8px; border:0; cursor:pointer; font-weight:600;
      background: rgba(255,255,255,0.12); color: #fff;
    }

    ul#todoList{ list-style:none; margin:0; padding:0; max-height:44vh; overflow:auto; }
    li.item{
      display:flex; align-items:center; gap:10px; padding:10px; border-radius:8px; margin-bottom:8px;
      background: rgba(0,0,0,0.04);
    }
    li.item .text{ flex:1; word-break:break-word; font-size:20px; }
    li.item.completed .text{ text-decoration:line-through; opacity:0.7; }
    .actions{ display:flex; gap:6px; }
    .small{ font-size:13px; padding:6px 8px; border-radius:6px; background:transparent; border:0; cursor:pointer; }

    .meta{ display:flex; justify-content:space-between; align-items:center; margin-top:12px; font-size:13px; opacity:0.95; }

    @media (max-width:480px){ .app{ padding:12px } h1{ font-size:18px } }
  </style>
</head>
<body>
  <main class="app" role="application" aria-labelledby="todoTitle">
    <h1 id="todoTitle">To‑Do List</h1>

    <div class="controls">
      <input id="newTodo" type="text" placeholder="Add a new task and press Enter" aria-label="New task" />
      <button id="addBtn">Add</button>
      <button id="clearCompleted" title="Remove all completed">Clear Completed</button>
    </div>

    <ul id="todoList" aria-live="polite"></ul>

    <div class="meta">
      <div id="counts">0 items</div>
      <div>
        <button id="filterAll" class="small">All</button>
        <button id="filterActive" class="small">Active</button>
        <button id="filterCompleted" class="small">Completed</button>
      </div>
    </div>
  </main>

  <script>
    // Simple, small to-do list with localStorage persistence
    const STORAGE_KEY = 'basic_todo_red_bg_v1';

    const newTodoEl = document.getElementById('newTodo');
    const addBtn = document.getElementById('addBtn');
    const todoListEl = document.getElementById('todoList');
    const countsEl = document.getElementById('counts');

    const clearCompletedBtn = document.getElementById('clearCompleted');
    const filterAllBtn = document.getElementById('filterAll');
    const filterActiveBtn = document.getElementById('filterActive');
    const filterCompletedBtn = document.getElementById('filterCompleted');

    let todos = []; // { id, text, done, createdAt }
    let filter = 'all';

    // load
    function load(){
      try{
        const raw = localStorage.getItem(STORAGE_KEY);
        todos = raw ? JSON.parse(raw) : [];
      }catch(e){ todos = []; }
    }

    // save
    function save(){
      localStorage.setItem(STORAGE_KEY, JSON.stringify(todos));
    }

    function render(){
      todoListEl.innerHTML = '';
      const list = todos.filter(t => {
        if(filter === 'active') return !t.done;
        if(filter === 'completed') return t.done;
        return true;
      });

      list.forEach(todo => {
        const li = document.createElement('li');
        li.className = 'item' + (todo.done ? ' completed' : '');

        const checkbox = document.createElement('input');
        checkbox.type = 'checkbox';
        checkbox.checked = !!todo.done;
        checkbox.addEventListener('change', () => toggleDone(todo.id));
        checkbox.setAttribute('aria-label','Mark task as done');

        const span = document.createElement('div');
        span.className = 'text';
        span.textContent = todo.text;
        span.title = 'Double-click to edit';
        span.tabIndex = 0;
        // allow editing on double click or Enter when focused
        span.addEventListener('dblclick', () => editTodoPrompt(todo.id));
        span.addEventListener('keydown', (e) => { if(e.key === 'Enter') editTodoPrompt(todo.id); });

        const actions = document.createElement('div');
        actions.className = 'actions';

        const editBtn = document.createElement('button');
        editBtn.className = 'small';
        editBtn.textContent = 'Edit';
        editBtn.addEventListener('click', () => editTodoPrompt(todo.id));

        const delBtn = document.createElement('button');
        delBtn.className = 'small';
        delBtn.textContent = 'Delete';
        delBtn.addEventListener('click', () => deleteTodo(todo.id));

        actions.appendChild(editBtn);
        actions.appendChild(delBtn);

        li.appendChild(checkbox);
        li.appendChild(span);
        li.appendChild(actions);

        todoListEl.appendChild(li);
      });

      const total = todos.length;
      const remaining = todos.filter(t => !t.done).length;
      countsEl.textContent = `${remaining} active / ${total} total`;

      save();
    }

    function addTodo(text){
      const trimmed = (text||'').trim();
      if(!trimmed) return;
      todos.unshift({ id: Date.now() + Math.random().toString(16).slice(2), text: trimmed, done:false, createdAt: new Date().toISOString() });
      render();
    }

    function toggleDone(id){
      todos = todos.map(t => t.id === id ? {...t, done: !t.done} : t);
      render();
    }

    function deleteTodo(id){
      todos = todos.filter(t => t.id !== id);
      render();
    }

    function editTodoPrompt(id){
      const t = todos.find(x => x.id === id);
      if(!t) return;
      const newText = prompt('Edit task', t.text);
      if(newText === null) return; // cancelled
      t.text = newText.trim() || t.text;
      render();
    }

    clearCompletedBtn.addEventListener('click', () => {
      todos = todos.filter(t => !t.done);
      render();
    });

    filterAllBtn.addEventListener('click', () => { filter = 'all'; render(); });
    filterActiveBtn.addEventListener('click', () => { filter = 'active'; render(); });
    filterCompletedBtn.addEventListener('click', () => { filter = 'completed'; render(); });

    addBtn.addEventListener('click', () => { addTodo(newTodoEl.value); newTodoEl.value = ''; newTodoEl.focus(); });

    newTodoEl.addEventListener('keydown', (e) => {
      if(e.key === 'Enter'){
        addTodo(newTodoEl.value);
        newTodoEl.value = '';
      }
    });

    // initialize
    load();
    render();
  </script>
</body>
</html>
