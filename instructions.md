1. Architecture Overview
Frontend (React SPA)
        |
        |  HTTP REST API (JSON)
        v
Backend (Spring Boot)
        |
        |  JPA / Hibernate
        v
PostgreSQL Database

2. PostgreSQL Setup
CREATE DATABASE todo_app;

3. Spring Boot Backend
Step 1 — Create Project
Use:

Spring Web

Spring Data JPA

PostgreSQL Driver

Lombok (optional)

You can generate from: start.spring.io

Step 2 — application.yml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/todo_app
    username: postgres
    password: yourpassword

  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        
Step 3 — Entity
@Entity
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Todo {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    private boolean completed;
}

Step 4 — Repository
@Repository
public interface TodoRepository extends JpaRepository<Todo, Long> {
}

Step 5 — Service Layer (Clean Architecture)
@Service
public class TodoService {

    private final TodoRepository repository;

    public TodoService(TodoRepository repository) {
        this.repository = repository;
    }

    public List<Todo> findAll() {
        return repository.findAll();
    }

    public Todo save(Todo todo) {
        return repository.save(todo);
    }

    public void delete(Long id) {
        repository.deleteById(id);
    }
}
Step 6 — REST Controller
@RestController
@RequestMapping("/api/todos")
@CrossOrigin(origins = "http://localhost:3000")
public class TodoController {

    private final TodoService service;

    public TodoController(TodoService service) {
        this.service = service;
    }

    @GetMapping
    public List<Todo> getAll() {
        return service.findAll();
    }

    @PostMapping
    public Todo create(@RequestBody Todo todo) {
        return service.save(todo);
    }

    @DeleteMapping("/{id}")
    public void delete(@PathVariable Long id) {
        service.delete(id);
    }
}

4. React Frontend
   npx create-react-app todo-frontend
   cd todo-frontend
   npm start
   Install axios:

  Install axios:
  npm install axios

API Service (api.js)
import axios from "axios";

const API_URL = "http://localhost:8080/api/todos";

export const getTodos = () => axios.get(API_URL);
export const createTodo = (todo) => axios.post(API_URL, todo);
export const deleteTodo = (id) => axios.delete(`${API_URL}/${id}`);

App.js
import React, { useEffect, useState } from "react";
import { getTodos, createTodo, deleteTodo } from "./api";

function App() {
  const [todos, setTodos] = useState([]);
  const [title, setTitle] = useState("");

  useEffect(() => {
    fetchTodos();
  }, []);

  const fetchTodos = async () => {
    const response = await getTodos();
    setTodos(response.data);
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    await createTodo({ title, completed: false });
    setTitle("");
    fetchTodos();
  };

  const handleDelete = async (id) => {
    await deleteTodo(id);
    fetchTodos();
  };

  return (
    <div>
      <h1>Todo App</h1>

      <form onSubmit={handleSubmit}>
        <input 
          value={title}
          onChange={(e) => setTitle(e.target.value)}
          placeholder="New todo"
        />
        <button type="submit">Add</button>
      </form>

      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            {todo.title}
            <button onClick={() => handleDelete(todo.id)}>
              Delete
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;
