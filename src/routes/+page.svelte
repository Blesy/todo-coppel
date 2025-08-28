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
  <h1 class="text-2xl font-bold mb-4">Todo App</h1>
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