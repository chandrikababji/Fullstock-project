I’ll provide code snippets for different sections of the project to help you build the To-Do list application. I’ll break it down into backend and frontend components, including the database setup, API routes, React components, and state management.

1. Backend (Express.js + PostgreSQL)
First, install the necessary dependencies for the backend:

bash
Copy code
npm init -y
npm install express pg sequelize body-parser cors
Database Setup (PostgreSQL + Sequelize)
Create a Sequelize model for tasks:

javascript
Copy code
// models/Task.js
const { DataTypes } = require('sequelize');
const sequelize = require('../config/database'); // Your DB configuration

const Task = sequelize.define('Task', {
  title: {
    type: DataTypes.STRING,
    allowNull: false,
  },
  description: {
    type: DataTypes.TEXT,
  },
  due_date: {
    type: DataTypes.DATE,
    allowNull: false,
  },
  is_recurring: {
    type: DataTypes.BOOLEAN,
    defaultValue: false,
  },
  recurrence_type: {
    type: DataTypes.ENUM('daily', 'weekly', 'monthly', 'yearly'),
  },
  recurrence_customization: {
    type: DataTypes.JSON,
    allowNull: true,
  },
});

module.exports = Task;
API Routes for CRUD Operations
Set up the main routes for tasks:

javascript
Copy code
// routes/tasks.js
const express = require('express');
const Task = require('../models/Task');
const router = express.Router();

// Create Task
router.post('/', async (req, res) => {
  try {
    const task = await Task.create(req.body);
    res.json(task);
  } catch (error) {
    res.status(500).json({ error: 'Failed to create task' });
  }
});

// Read Tasks
router.get('/', async (req, res) => {
  try {
    const tasks = await Task.findAll();
    res.json(tasks);
  } catch (error) {
    res.status(500).json({ error: 'Failed to retrieve tasks' });
  }
});

// Update Task
router.put('/:id', async (req, res) => {
  try {
    const task = await Task.update(req.body, {
      where: { id: req.params.id },
    });
    res.json(task);
  } catch (error) {
    res.status(500).json({ error: 'Failed to update task' });
  }
});

// Delete Task
router.delete('/:id', async (req, res) => {
  try {
    await Task.destroy({
      where: { id: req.params.id },
    });
    res.json({ message: 'Task deleted' });
  } catch (error) {
    res.status(500).json({ error: 'Failed to delete task' });
  }
});

module.exports = router;
App.js (Express Server Setup)
javascript
Copy code
// app.js
const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');
const taskRoutes = require('./routes/tasks');
const sequelize = require('./config/database'); // DB connection

const app = express();
app.use(cors());
app.use(bodyParser.json());

// Test DB connection
sequelize.sync().then(() => {
  console.log('Connected to the database!');
});

// Routes
app.use('/api/tasks', taskRoutes);

// Start server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
2. Frontend (React + Next.js)
Next, initialize a Next.js project and install required dependencies:

bash
Copy code
npx create-next-app todo-app
cd todo-app
npm install axios tailwindcss zustand
Set up Tailwind CSS by adding it to your tailwind.config.js and importing it in styles/globals.css.

Zustand State Management Setup
javascript
Copy code
// store/useTaskStore.js
import create from 'zustand';
import axios from 'axios';

const useTaskStore = create((set) => ({
  tasks: [],
  fetchTasks: async () => {
    const response = await axios.get('/api/tasks');
    set({ tasks: response.data });
  },
  addTask: async (task) => {
    const response = await axios.post('/api/tasks', task);
    set((state) => ({ tasks: [...state.tasks, response.data] }));
  },
  updateTask: async (id, updatedTask) => {
    const response = await axios.put(`/api/tasks/${id}`, updatedTask);
    set((state) => ({
      tasks: state.tasks.map((task) =>
        task.id === id ? response.data : task
      ),
    }));
  },
  deleteTask: async (id) => {
    await axios.delete(`/api/tasks/${id}`);
    set((state) => ({
      tasks: state.tasks.filter((task) => task.id !== id),
    }));
  },
}));
export default useTaskStore;
Task List Component
javascript
Copy code
// components/TaskList.js
import useTaskStore from '../store/useTaskStore';
import { useEffect } from 'react';

const TaskList = () => {
  const { tasks, fetchTasks, deleteTask } = useTaskStore();

  useEffect(() => {
    fetchTasks();
  }, [fetchTasks]);

  return (
    <div className="task-list">
      {tasks.map((task) => (
        <div key={task.id} className="task-item p-4">
          <h3>{task.title}</h3>
          <p>{task.description}</p>
          <button onClick={() => deleteTask(task.id)} className="bg-red-500 p-2">Delete</button>
        </div>
      ))}
    </div>
  );
};

export default TaskList;
Task Form with Recurrence Logic
javascript
Copy code
// components/TaskForm.js
import { useState } from 'react';
import useTaskStore from '../store/useTaskStore';

const TaskForm = () => {
  const [task, setTask] = useState({ title: '', description: '', due_date: '', is_recurring: false });
  const { addTask } = useTaskStore();

  const handleSubmit = (e) => {
    e.preventDefault();
    addTask(task);
    setTask({ title: '', description: '', due_date: '', is_recurring: false });
  };

  return (
    <form onSubmit={handleSubmit} className="task-form">
      <input
        type="text"
        placeholder="Task title"
        value={task.title}
        onChange={(e) => setTask({ ...task, title: e.target.value })}
        className="border p-2"
      />
      <textarea
        placeholder="Task description"
        value={task.description}
        onChange={(e) => setTask({ ...task, description: e.target.value })}
        className="border p-2"
      />
      <input
        type="date"
        value={task.due_date}
        onChange={(e) => setTask({ ...task, due_date: e.target.value })}
        className="border p-2"
      />
      <div>
        <label>Recurring:</label>
        <input
          type="checkbox"
          checked={task.is_recurring}
          onChange={(e) => setTask({ ...task, is_recurring: e.target.checked })}
        />
      </div>
      {/* Additional fields for recurrence customization */}
      <button type="submit" className="bg-blue-500 p-2">
        Add Task
      </button>
    </form>
  );
};

export default TaskForm;
3. Reusable Date Picker Component
You can integrate a date picker library like react-datepicker and customize it for recurring tasks.

Date Picker with Mini Calendar
bash
Copy code
npm install react-datepicker
javascript
Copy code
// components/DatePicker.js
import React, { useState } from 'react';
import DatePicker from 'react-datepicker';
import 'react-datepicker/dist/react-datepicker.css';

const CustomDatePicker = ({ selectedDate, onChange }) => {
  const [startDate, setStartDate] = useState(selectedDate || new Date());

  return (
    <div>
      <DatePicker
        selected={startDate}
        onChange={(date) => {
          setStartDate(date);
          onChange(date);
        }}
        inline
      />
      {/* Add mini-calendar preview here */}
    </div>
  );
};

export default CustomDatePicker;
4. Testing Setup (Jest)
You can write unit and integration tests using Jest. Example unit test:

javascript
Copy code
// __tests__/TaskForm.test.js
import { render, screen, fireEvent } from '@testing-library/react';
import TaskForm from '../components/TaskForm';

test('renders task form', () => {
  render(<TaskForm />);
  expect(screen.getByPlaceholderText('Task title')).toBeInTheDocument();
});

test('submits task', () => {
  const { getByPlaceholderText, getByText } = render(<TaskForm />);
  fireEvent.change(getByPlaceholderText('Task title'), { target: { value: 'Test Task' } });
  fireEvent.click(getByText('Add Task'));
  expect(getByText('Test Task')).toBeInTheDocument();
});
