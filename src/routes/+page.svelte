<script lang="ts">
	type TodoType = {
		id: number;
		text: string;
		done: boolean;
	};

	type FiltersType = 'all' | 'active' | 'completed';

	let todos = $state<TodoType[]>([]);

	let filter = $state<FiltersType>('all');

	let filteredTodos = $derived(filterTodos());

	$effect(() => {
		const savedTodosString = localStorage.getItem('todos');
		if (savedTodosString) todos = JSON.parse(savedTodosString);
	});

	$effect(() => {
		localStorage.setItem('todos', JSON.stringify(todos));
	});

	function addTodo(event: KeyboardEvent) {
		if (event.key !== 'Enter') return;

		const input = event.target as HTMLInputElement;
		const text = input.value;
		input.value = '';

		todos = [...todos, { id: todos.length + 1, text, done: false }];
	}

	function editTodo(event: Event) {
		const input = event.target as HTMLInputElement;
		const index = Number(input.dataset.index);
		todos[index].text = input.value;
	}

	function toggleTodo(event: Event) {
		const checkbox = event.target as HTMLInputElement;
		const index = Number(checkbox.dataset.index);
		todos[index].done = !todos[index].done;
	}

	function removeTodo(event: Event) {
		const button = event.target as HTMLButtonElement;
		const index = Number(button.dataset.index);
		todos = todos.filter((todo) => todo.id !== index);
	}

	function setFilter(newFilter: FiltersType) {
		filter = newFilter;
	}

	function filterTodos() {
		switch (filter) {
			case 'active':
				return todos.filter((todo) => !todo.done);
			case 'completed':
				return todos.filter((todo) => todo.done);
			default:
				return todos;
		}
	}

	function remaining() {
		return todos.filter((todo) => !todo.done).length;
	}
</script>

<h1>Todos</h1>

<input type="text" placeholder="Add todo" onkeydown={addTodo} />

<div class="todos">
	{#each filteredTodos as todo, i}
		<div class:completed={todo.done} class="todo">
			<input type="text" value={todo.text} oninput={editTodo} data-index={i} />
			<input type="checkbox" checked={todo.done} onchange={toggleTodo} data-index={i} />
			<button class="remove" onclick={removeTodo} data-index={todo.id}>X</button>
		</div>
	{/each}
</div>

<div class="filters">
	<button onclick={() => setFilter('all')}>All</button>
	<button onclick={() => setFilter('active')}>Active</button>
	<button onclick={() => setFilter('completed')}>Completed</button>
</div>

<p>{remaining()} remaining</p>

<style>
	.todos {
		display: grid;
		gap: 1rem;
		margin-block-start: 1rem;
	}

	.todo {
		position: relative;
		transition: opacity 0.3s;
	}

	.completed {
		opacity: 0.4;
	}

	.remove {
		position: absolute;
		padding: 0;
		margin: 0;
		right: -10%;
		top: 50%;
		transform: translate(50%, -50%);
		background: none;
		border: none;
		outline: none;
		color: hsl(220 10% 98%);
		cursor: pointer;
	}

	input[type='text'] {
		width: 100%;
		padding: 1rem;
	}

	input[type='checkbox'] {
		position: absolute;
		right: 4%;
		top: 50%;
		translate: 0% -50%;
	}

	.filters {
		margin-block: 1rem;
	}
</style>
