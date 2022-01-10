<script>
  import FilterButtons from './FilterButtons.svelte';
  import Todo from './Todo.svelte'
  import MoreActions from './MoreActions.svelte'
  import NewTodo from './NewTodo.svelte'
  import TodosStatus from './TodosStatus.svelte'
  import { alert } from '../stores.js'
  
  export let todos = []

  $: newTodoId = todos.length ? Math.max(...todos.map(t => t.id)) + 1 : 1
  let newTodoName = ''
  let todosStatus                   // reference to TodosStatus instance
  
  function removeTodo(todo) {
    todos = todos.filter(t => t.id !== todo.id)
    todosStatus.focus()             // give focus to status heading
    $alert = `Todo '${todo.name}' has been deleted`
  }

  function addTodo(name) {
    todos = [...todos, { id: newTodoId, name, completed: false }]
    $alert = `Todo '${name}' has been added`
  }

  function updateTodo(todo) {
    const i = todos.findIndex(t => t.id === todo.id)
    if (todos[i].name !== todo.name)            $alert = `todo '${todos[i].name}' has been renamed to '${todo.name}'`
    if (todos[i].completed !== todo.completed)  $alert = `todo '${todos[i].name}' marked as ${todo.completed ? 'completed' : 'active'}`
    todos[i] = { ...todos[i], ...todo }
  }

  let filter = 'all'
  const filterTodos = (filter, todos) =>
    filter === 'active' ? todos.filter(t => !t.completed) :
    filter === 'completed' ? todos.filter(t => t.completed) :
    todos

  $: {
    if (filter === 'all')               $alert = 'Browsing all todos'
    else if (filter === 'active')       $alert = 'Browsing active todos'
    else if (filter === 'completed')    $alert = 'Browsing completed todos'
  }

  const removeCompletedTodos = () => {
    $alert = `Removed ${todos.filter(t => t.completed).length} todos`
    todos = todos.filter(t => !t.completed)
  }

  const checkAllTodos = (completed) => {
        /* 
            If we try to modify the todos array using a foreach, then we will need to declase a self-assignment so that svelte can reactively trigger the change(s).
            This is a bit redundant, but gives the same result as the next method. 
            Svelte will not mark todos as changed because it does not know that when we update our t variable inside the forEach() method, we are also modifying the todos array. 
            And that makes sense, because otherwise Svelte would be aware of the inner workings of the forEach() method; the same would therefore be true for any method attached to any object or array
        */
        //todos.forEach(t => t.completed = completed) 
        //todos = todos 

        /*
           Instead we can alter each index of the todos array inside of the forach loop, so that svelte registers the updated t value.
        */
        //todos.forEach( (t,i) => todos[i].completed = completed)

        /*
            Alternatively, we can assign todos = <the new array> using map; This solution has the added benefit of returning a new array with new objects, which avoids mutating the original todos array. 
            Note: The <svelte:options immutable={true}/> option tells the compiler that you promise not to mutate any objects. 
            This allows it to be less conservative about checking whether values have changed and generate simpler and more performant code. 
        */
        todos = todos.map(t => ({...t, completed}))
        $alert = `${completed ? 'Checked' : 'Unchecked'} ${todos.length} todos`
  }
  
</script>
  

<!-- Todos.svelte -->
<div class="todoapp stack-large">

  <!-- NewTodo -->
  <NewTodo autofocus on:addTodo={e => addTodo(e.detail)} />

  <!-- FilterButtons -->
  <FilterButtons bind:filter={filter} />

  <!-- TodosStatus -->
  <TodosStatus bind:this={todosStatus} {todos} />

  <!-- Todos -->
  <ul role="list" class="todo-list stack-large" aria-labelledby="list-heading">
    {#each filterTodos(filter, todos) as todo (todo.id)}
      <li class="todo">
        <Todo {todo}
          on:update={e => updateTodo(e.detail)}
          on:remove={e => removeTodo(e.detail)}
        />
      </li>
    {:else}
      <li>Nothing to do here!</li>
    {/each}
  </ul>

  <hr />

  <!-- MoreActions -->
  <MoreActions {todos}
      on:checkAll={e => checkAllTodos(e.detail)}
      on:removeCompleted={removeCompletedTodos}
  />
  
</div>
  