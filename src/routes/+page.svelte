<script lang="ts">
  import { onMount } from "svelte";
  import { slide } from "svelte/transition";

  export let todos: { id: number, name: string }[] = [];

  onMount(() => {
    todos = JSON.parse(localStorage.getItem("todos") ?? "[]");
  });

  function saveTodos() {
    localStorage.setItem("todos", JSON.stringify(todos));
  }

  function addTodo(): void {
    const taskNameInput = document.getElementById("taskNameInput") as HTMLInputElement;
    const name = taskNameInput.value;
    if (name.trim().length === 0) return;
    taskNameInput.value = "";
    todos.forEach((todo) => todo.id++);
    todos.unshift({ id: 0, name });
    todos = todos;
    saveTodos();
    taskNameInput.focus();
  }

  function completeTodo(todo: { id: number, name: string }, e: MouseEvent | KeyboardEvent): void {
    todos = todos.filter((item) => item !== todo);
    (e.target as HTMLElement).classList.add("completed");
    saveTodos();
  }

  function inputKeydown(e: KeyboardEvent): void {
    if (e.key === "Enter") addTodo();
  }

  function inputFocus(): void {
    document.getElementById("newTaskContainer")?.classList.add("focused");
  }

  function inputBlur(): void {
    document.getElementById("newTaskContainer")?.classList.remove("focused");
  }

  let draggedTaskId: number | undefined;

  function dragStart(event: DragEvent): void {
    draggedTaskId = parseInt((event.target as HTMLElement).id.replace("task", ""));
    event.dataTransfer?.setData("text/plain", ""); // Required for Firefox to enable dragging
  }

  function dragOver(event: DragEvent): void {
    event.preventDefault();
    (event.target as HTMLElement).classList.add("dragover");
  }

  function drop(event: DragEvent): void {
    event.preventDefault();
    const target = event.target as HTMLElement;
    const dropTaskId = parseInt(target.id.replace("task", ""));

    if (draggedTaskId !== null && dropTaskId !== draggedTaskId) {
      const draggedTask = todos.find((todo) => todo.id === draggedTaskId);
      const dropTask = todos.find((todo) => todo.id === dropTaskId);
      if (draggedTask && dropTask) {
        todos = todos.map((todo) => {
          if (todo.id === draggedTaskId) {
            return { ...todo, id: dropTask.id };
          } else if (todo.id === dropTaskId) {
            return { ...todo, id: draggedTask.id };
          } else {
            return todo;
          }
        });
        todos = [...todos.sort((a, b) => a.id - b.id)];
        saveTodos();
      }
    }

    target.classList.remove("dragover");
  }
</script>

<div id="todosContainer" role="rowgroup">
  <div id="newTaskContainer" class="todo" role="row" tabindex="-1">
    <button on:mouseup={() => addTodo()}>
      <svg viewBox="0 0 18 18" xmlns="http://www.w3.org/2000/svg">
        <circle cx="9" cy="9" r="8" fill="none" stroke="#868686" stroke-width="2">
      </svg>
    </button>
    <input
      id="taskNameInput"
      placeholder="Add a task..."
      on:focus={inputFocus}
      on:blur={inputBlur}
      on:keydown={inputKeydown}
    />
  </div>
  {#each todos as todo (todo)}
    <div
      id="task{todo.id}"
      class="todo"
      transition:slide={{ duration: 250 }}
      role="row"
      tabindex="-1"
      draggable="true"
      on:dragstart={dragStart}
      on:dragover={dragOver}
      on:drop={drop}
    >
      <button on:mouseup={(e) => completeTodo(todo, e)}>
        <svg viewBox="0 0 18 18" xmlns="http://www.w3.org/2000/svg">
          <circle cx="9" cy="9" r="8" fill="none" stroke="#dfdfdf" stroke-width="2">
        </svg>
      </button>
      {todo.name}
    </div>
  {/each}
</div>

<style lang="scss">
  @import "@fontsource/inter";

  $accent: #7784f0;

  #todosContainer {
    width: 100%;
    max-width: 750px;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
    gap: 10px;

    .todo {
      padding: 16px 8px;
      background-color: #222222;
      border-radius: 10px;
      display: flex;
      align-items: center;
      gap: 10px;
      font-size: 16px;
      height: 21px;
      transition: box-shadow 0.05s;
      user-select: none;

      input, button {
        background-color: #222222;
        border: none;
      }

      input {
        flex-grow: 1;
        color: #dcdcdc;
        padding: 5px;
        font-size: 16px;
        margin-left: -5px;
        font-family: "Inter", sans-serif;
        height: 13px;
        outline: none;
      }

      button {
        margin-bottom: -2px;
        padding-right: 0;
      }

      svg {
        width: 18px;
        height: 18px;

        &.completed circle {
          fill: $accent;
        }
      }
    }
  }

  :global(.focused) {
    box-shadow: 0 0 12px 0 $accent;
  }
</style>
