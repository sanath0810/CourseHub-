# CourseHub-
import React, { useEffect, useRef, useState } from 'react';

const STORAGE_KEY = 'coursehub:todo';

function makeId() {
  return Math.random().toString(36).slice(2, 9);
}

export default function TodoList() {
  const [items, setItems] = useState(() => {
    try {
      const raw = localStorage.getItem(STORAGE_KEY);
      return raw ? JSON.parse(raw) : [];
    } catch {
      return [];
    }
  });
  const [text, setText] = useState('');
  const [filter, setFilter] = useState('all'); // all | active | completed
  const inputRef = useRef(null);

  useEffect(() => {
    try {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(items));
    } catch {
      // ignore storage errors
    }
  }, [items]);

  const add = (event) => {
    if (event) event.preventDefault();
    const value = text.trim();
    if (!value) return;
    const next = [{ id: makeId(), text: value, done: false }, ...items];
    setItems(next);
    setText('');
    inputRef.current?.focus();
  };

  const toggle = (id) => setItems(items.map((it) => (it.id === id ? { ...it, done: !it.done } : it)));
  const remove = (id) => setItems(items.filter((it) => it.id !== id));
  const clearCompleted = () => setItems(items.filter((it) => !it.done));
  const updateText = (id, newText) =>
    setItems(items.map((it) => (it.id === id ? { ...it, text: newText } : it)));

  const visible = items.filter((it) => (filter === 'all' ? true : filter === 'completed' ? it.done : !it.done));

  return (
    <section className="max-w-3xl mx-auto my-12 p-6 bg-white rounded-lg shadow">
      <h2 className="text-lg font-semibold mb-3">My To‑Do</h2>

      <form onSubmit={add} className="flex gap-2 mb-4">
        <label htmlFor="todo-input" className="sr-only">Add task</label>
        <input
          id="todo-input"
          ref={inputRef}
          value={text}
    onChange={(evt) => setText(evt.target.value)}
          placeholder="Add a new task (press Enter)"
          className="flex-1 px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-primary-500"
        />
        <button
          type="submit"
          className="px-4 py-2 bg-primary-600 text-white rounded hover:bg-primary-700"
          aria-label="Add task"
        >
          Add
        </button>
      </form>

      <div className="flex items-center justify-between mb-3">
        <div className="flex gap-2">
          {['all', 'active', 'completed'].map((f) => (
            <button
              key={f}
              type="button"
              onClick={() => setFilter(f)}
              className={`px-2 py-1 rounded text-sm ${filter === f ? 'bg-primary-100 text-primary-700' : 'text-gray-600 hover:bg-gray-100'}`}
            >
              {f === 'all' ? 'All' : f === 'active' ? 'Active' : 'Completed'}
            </button>
          ))}
        </div>
        <div className="text-sm text-gray-600">{items.length} total</div>
      </div>

      <ul className="space-y-2">
        {visible.length === 0 && <li className="text-gray-500">No tasks here — add something!</li>}
        {visible.map((it) => (
          <li key={it.id} className="flex items-center gap-3">
            <input
              id={`chk-${it.id}`}
              type="checkbox"
              checked={it.done}
              onChange={() => toggle(it.id)}
              className="w-4 h-4"
            />
            <EditableText value={it.text} onSave={(newText) => updateText(it.id, newText)} done={it.done} />
            <div className="ml-auto flex items-center gap-2">
              <button onClick={() => toggle(it.id)} aria-label={it.done ? 'Mark as active' : 'Mark as completed'} className="text-sm text-gray-600 hover:text-gray-800">
                {it.done ? 'Undo' : 'Done'}
              </button>
              <button onClick={() => remove(it.id)} aria-label="Delete task" className="text-sm text-red-600 hover:underline">Delete</button>
            </div>
          </li>
        ))}
      </ul>

      <div className="mt-4 flex justify-between items-center">
        <div className="text-sm text-gray-600">{items.filter((i) => !i.done).length} remaining</div>
        <div className="flex gap-2">
          <button onClick={() => setItems([])} className="text-sm text-gray-600 hover:underline">Clear all</button>
          <button onClick={clearCompleted} className="text-sm text-gray-600 hover:underline">Clear completed</button>
        </div>
      </div>
    </section>
  );
}

function EditableText({ value, onSave, done }) {
  const [editing, setEditing] = useState(false);
  const [val, setVal] = useState(value);
  const ref = useRef(null);

  useEffect(() => setVal(value), [value]);

  useEffect(() => {
    if (editing) ref.current?.focus();
  }, [editing]);

  const submit = () => {
    const next = val.trim();
    if (next.length === 0) setVal(value); // don't save empty
    else onSave(next);
    setEditing(false);
  };

  return (
    <div className="flex-1">
      {editing ? (
        <input
          ref={ref}
          value={val}
          onChange={(evt) => setVal(evt.target.value)}
          onBlur={submit}
          onKeyDown={(evt) => {
            if (evt.key === 'Enter') submit();
            if (evt.key === 'Escape') {
              setVal(value);
              setEditing(false);
            }
          }}
          className="w-full px-2 py-1 border rounded focus:outline-none"
        />
      ) : (
        <button onClick={() => setEditing(true)} className={`text-left w-full ${done ? 'line-through text-gray-500' : 'text-gray-800'}`}>
          {value}
        </button>
      )}
    </div>
  );
}
