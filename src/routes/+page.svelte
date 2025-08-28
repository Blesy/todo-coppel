<script lang="ts">
  type TodoStatus = 'pendiente' | 'completado';

  type Todo = {
    id: number;
    texto: string;
    fechaInicio: string;
    status: TodoStatus;
  };

  let todos: Todo[] = [];
  let todosFiltrados: Todo[] = [];
  let now = Date.now();
  let texto = '';
  let fechaInicio = `${new Date(now).getFullYear()}-${String(new Date(now).getMonth() + 1).padStart(2, '0')}-${String(new Date(now).getDate()).padStart(2, '0')}`;
  let editId: number | null = null;
  let filtro: TodoStatus | 'todos' = 'todos';

  // Cargar todos desde localStorage al iniciar
  if (typeof window !== 'undefined') {
    const guardados = localStorage.getItem('todos');
    if (guardados) {
      todos = JSON.parse(guardados);
    }
    filtrarTodos(filtro);
  }

  // Guardar todos en localStorage cada vez que cambian
  function setLocalStorage(value: string) {
    filtrarTodos(filtro);
    if (typeof window !== 'undefined') {
      localStorage.setItem('todos', value);
    }
  }

  function agregarTodo() {
    if (!texto || !fechaInicio) return;
    todos = [
      ...todos,
      {
        id: Date.now(),
        texto,
        fechaInicio,
        status: 'pendiente'
      }
    ];
    texto = '';
    fechaInicio = '';
    setLocalStorage(JSON.stringify(todos));
  }

  function editarTodo(todo: Todo) {
    texto = todo.texto;
    fechaInicio = todo.fechaInicio;
    editId = todo.id;
  }

  function guardarEdicion() {
    if (editId === null) return;
    todos = todos.map(t =>
      t.id === editId ? { ...t, texto, fechaInicio } : t
    );
    todos = [...todos];
    texto = '';
    fechaInicio = '';
    editId = null;
    setLocalStorage(JSON.stringify(todos));
  }

  function cambiarStatus(id: number) {
    todos = todos.map(t =>
      t.id === id
        ? { ...t, status: t.status === 'pendiente' ? 'completado' : 'pendiente' }
        : t
    );
    todos = [...todos];
    setLocalStorage(JSON.stringify(todos));
  }

  function eliminarTodo(id: number) {
    const todo = todos.find(t => t.id === id);
    if (
      todo &&
      confirm(`¿Seguro que quieres eliminar la tarea "${todo.texto}"? Esta acción no se puede deshacer.`)
    ) {
      todos = todos.filter(t => t.id !== id);
      setLocalStorage(JSON.stringify(todos));
    }
  }

  function filtrarTodos(status: TodoStatus | 'todos') {
    if (status === 'todos') {
        todosFiltrados = todos;
        return;
    }
    todosFiltrados = todos.filter(t => t.status === status);
    return;
  }

</script>

<div class="max-w-xl mx-auto mt-10 p-6 bg-white rounded shadow">
  <div class="flex items-center justify-between mb-4">
    <h1 class="text-2xl font-bold">Todo App</h1>
    <a
      href="https://github.com/Blesy/todo-coppel"
      target="_blank"
      rel="noopener noreferrer"
      class="flex items-center gap-2 text-gray-700 hover:text-black"
      aria-label="Repositorio en GitHub"
    >
      <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" class="inline-block">
        <path d="M12 .297c-6.63 0-12 5.373-12 12 0 5.303 3.438 9.8 8.205 11.385.6.113.82-.258.82-.577 0-.285-.01-1.04-.015-2.04-3.338.724-4.042-1.61-4.042-1.61-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.84 1.236 1.84 1.236 1.07 1.834 2.809 1.304 3.495.997.108-.775.418-1.304.762-1.604-2.665-.305-5.466-1.332-5.466-5.93 0-1.31.469-2.381 1.236-3.221-.124-.303-.535-1.523.117-3.176 0 0 1.008-.322 3.301 1.23a11.52 11.52 0 0 1 3.003-.404c1.018.005 2.045.138 3.003.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.873.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.61-2.804 5.624-5.475 5.921.43.372.823 1.102.823 2.222 0 1.606-.014 2.898-.014 3.293 0 .322.218.694.825.576C20.565 22.092 24 17.592 24 12.297c0-6.627-5.373-12-12-12"/>
      </svg>
      Repositorio
    </a>
  </div>
  <div class={`mb-2 px-3 py-2 rounded ${editId !== null ? 'bg-yellow-100 border-yellow-400 border' : ''}`}>
    <h2 class="text-lg font-semibold mb-2">
      {editId !== null ? 'Editando tarea' : 'Nueva tarea'}
    </h2>
    <form
      class="flex flex-col gap-2 mb-4"
      on:submit|preventDefault={editId ? guardarEdicion : agregarTodo}
    >
      <input
        class="border rounded px-2 py-1"
        type="text"
        bind:value={texto}
        placeholder="Escribe tu tarea"
        required
      />
      <input
        class="border rounded px-2 py-1"
        type="date"
        bind:value={fechaInicio}
        required
      />
      <button
        class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600"
        type="submit"
        on:click={() => filtro = filtro}
      >
        {editId ? 'Guardar cambios' : 'Agregar'}
      </button>
      {#if editId !== null}
        <button
          class="bg-gray-400 text-white px-4 py-2 rounded hover:bg-gray-500"
          type="button"
          on:click={() => { texto = ''; fechaInicio = ''; editId = null; }}
        >
          Cancelar
        </button>
      {/if}
    </form>
  </div>

  <div class="mb-4 flex gap-2">
    <button
      class="px-3 py-1 rounded border"
      class:selected={filtro === 'todos'}
      on:click={() => {
          filtro = 'todos'
          filtrarTodos('todos')
     }}
    >Todos</button>
    <button
      class="px-3 py-1 rounded border"
      class:selected={filtro === 'pendiente'}
      on:click={() => {
        filtro = 'pendiente'
        filtrarTodos('pendiente')
      }}
    >Pendientes</button>
    <button
      class="px-3 py-1 rounded border"
      class:selected={filtro === 'completado'}
      on:click={() => {
        filtro = 'completado'
        filtrarTodos('completado')
      }}
    >Completados</button>
  </div>

  <ul>
    {#each todosFiltrados as todo (todo.id)}
      <li class="flex items-center justify-between mb-2 p-2 border rounded">
        <div>
          <span class={todo.status === 'completado' ? 'line-through text-gray-400' : ''}>
            {todo.texto}
          </span>
          <span class="ml-2 text-xs text-gray-500">({todo.fechaInicio})</span>
        </div>
        <div class="flex gap-2">
          <button
            class="text-sm px-2 py-1 bg-green-500 text-white rounded hover:bg-green-600"
            on:click={() => cambiarStatus(todo.id)}
          >
            {todo.status === 'pendiente' ? 'Completar' : 'Pendiente'}
          </button>
          <button
            class="text-sm px-2 py-1 bg-yellow-500 text-white rounded hover:bg-yellow-600"
            on:click={() => editarTodo(todo)}
          >
            Editar
          </button>
          <button
            class="text-sm px-2 py-1 bg-red-500 text-white rounded hover:bg-red-600"
            on:click={() => eliminarTodo(todo.id)}
          >
            Eliminar
          </button>
        </div>
      </li>
    {/each}
  </ul>
</div>

<style>
  .selected {
    background-color: #3b82f6;
    color: white;
  }
</style>