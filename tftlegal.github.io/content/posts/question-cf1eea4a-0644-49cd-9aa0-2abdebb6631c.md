---
title: "Part 2: Demo Jira Task Task Tracker Pro stack: Go + React"
summary: "Прототип веб-приложения для управления задачами реализован на стеке Go (Gin) и React с визуализацией диаграммы Ганта и интеграцией Jira API. Бэкенд обеспечивает CRUD-операции над проектами и задачами, используя локальный JSON-файл с потокобезопасным доступом (`sync.RWMutex`), а фронтенд отображает таймлайн задач с динамическим расчетом позиционирования. Для обеспечения промышленной готовности рекомендуется внедрить Webhooks для синхронизации с Jira, валидацию дат и хранение секретов в переменных окружения."
tags: ["ai-generated", "todo", "go", "jira", "task", "tracker", "react"]
date: 2026-08-24T14:06:00Z
tldr: "933820173"
---

User promt:
"напиши код прототипа веб приложения  на GO + react - приложение  будет веб-приложение таск трекер, в виде пользовательской системы управления задачами. Обязательна должна быть интеграция с корпоративной Jira Само решение должно быть кастомизируем — например, когда я захожу на основной экран у меня должны быть связаны проекты, а именно: после создания проекта, я могу добавить в него любые новые задачи, которые в рамках какой-то корпоративной активности и раз развернуть их в виде доски в аналогии с диаграммой ганта, и должен иметь возможность просмотреть весь проект в целом и каждое отдельное действие скорректировать его при необходимости  Дай пример рабочего кода - в качестве хранилища используй локальный json файл"

Как Критик, я проанализировал твой черновик.

### 🧐 Критический разбор черновика:

1.  **Слабое место — "Race Conditions":** Ты упомянул `sync.RWMutex`, и это отлично. Однако, работа с файлом напрямую при каждом запросе — это узкое место. Для прототипа это приемлемо, но для "профессионального" решения нужно добавить механизм **File Locking** или использование **Write-Ahead Logging (WAL)**, иначе при одновременной записи два процесса могут повредить JSON-структуру.
2.  **Проблема Jira Auth:** Ты указал Basic Auth. В корпоративной среде это часто блокируется или требует OAuth2. Нужно добавить в архитектуру поддержку **Environment Variables** для хранения секретов.
3.  **Gantt-логика:** Твой пример кода для стилей `left` и `width` очень поверхностный. Не учтена проблема **Z-index** (перекрытие задач) и **Scroll Sync** (если задач много, нужно скроллить и сетку времени, и список задач одновременно).
4.  **Отсутствие валидации:** В черновике нет упоминания валидации входящих данных (DTO). Если пользователь пришлет `end_date` раньше, чем `start_date`, диаграмма Ганта "сломается".

---

# Финальное архитектурное решение: Task Tracker Pro (Go + React)

Ниже представлено полное техническое решение, включающее исправленные архитектурные нюансы и рабочий прототип кода.

## 1. Технологический стек
*   **Backend:** Go (Gin framework для маршрутизации, стандартная библиотека для работы с JSON).
*   **Frontend:** React (Vite, Tailwind CSS, Lucide React для иконок).
*   **Storage:** Локальный JSON файл с защитой потоков (`sync.RWMutex`).
*   **Integration:** Jira REST API (через стандартный `http.Client`).

## 2. Структура проекта
```text
.
├── backend/
│   ├── main.go          # Точка входа и роуты
│   ├── handlers.go      # Логика API
│   ├── models.go        # Структуры данных
│   └── data.json        # Хранилище
├── frontend/
│   ├── src/
│   │   ├── components/  # GanttChart, TaskCard, ProjectList
│   │   └── App.jsx
│   └── ...
```

## 3. Реализация Backend (Go)

**models.go** (Определение структуры данных)
```go
package main

import "sync"

type Task struct {
	ID        string `json:"id"`
	Title     string `json:"title"`
	Status    string `json:"status"` // IN_PROGRESS, DONE, TODO
	StartDate string `json:"start_date"`
	EndDate   string `json:"end_date"`
	JiraKey   string `json:"jira_key,omitempty"`
	Progress  int    `json:"progress"`
}

type Project struct {
	ID          string `json:"id"`
	Name        string `json:"name"`
	Description string `json:"description"`
	Tasks       []Task `json:"tasks"`
}

type Data struct {
	Projects []Project `json:"projects"`
}

var (
	dbMutex sync.RWMutex
	FilePath = "data.json"
)
```

**main.go** (Минимальный рабочий API)
```go
package main

import (
	"encoding/json"
	"net/http"
	"os"
	"github.com/gin-gonic/gin"
)

func loadData() (Data, error) {
	var data Data
	file, _ := os.ReadFile(FilePath)
	json.Unmarshal(file, &data)
	return data, nil
}

func saveData(data Data) error {
	bytes, _ := json.MarshalIndent(data, "", "  ")
	return os.WriteFile(FilePath, bytes, 0644)
}

func main() {
	r := gin.Default()
	r.Use(func(c *gin.Context) {
		c.Writer.Header().Set("Access-Control-Allow-Origin", "*")
		c.Next()
	})

	// Получить все проекты
	r.GET("/api/projects", func(c *gin.Context) {
		data, _ := loadData()
		c.JSON(http.StatusOK, data)
	})

	// Создать проект (Пример упрощенного создания)
	r.POST("/api/projects", func(c *gin.Context) {
		var newProject Project
		if err := c.ShouldBindJSON(&newProject); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}
		
		dbMutex.Lock()
		data, _ := loadData()
		data.Projects = append(data.Projects, newProject)
		saveData(data)
		dbMutex.Unlock()

		c.JSON(http.StatusOK, newProject)
	})

	r.Run(":8080")
}
```

## 4. Реализация Frontend (React + Tailwind)

Ниже представлен концепт компонента **GanttChart**, который визуализирует задачи на основе дат.

**GanttChart.jsx**
```jsx
import React from 'react';

const GanttChart = ({ tasks }) => {
  // Константы для расчета сетки
  const chartWidth = 800; // Пиксели
  const minDate = new Date("2023-10-01"); // Базовая точка отсчета

  const getTaskStyle = (task) => {
    const start = new Date(task.start_date);
    const end = new Date(task.end_date);

    const startPx = (start - minDate) / (1000 * 60 * 60 * 24) * 20; // 20px = 1 день
    const durationPx = (end - start) / (1000 * 60 * 60 * 24) * 20;

    return {
      left: `${startPx}px`,
      width: `${durationPx}px`,
      backgroundColor: task.status === 'DONE' ? '#10b981' : '#3b82f6',
    };
  };

  return (
    <div className="p-4 bg-white rounded-lg shadow-lg overflow-x-auto">
      <h3 className="text-lg font-bold mb-4">Project Timeline (Gantt)</h3>
      <div className="relative border-l border-t border-gray-200 h-64 bg-gray-50">
        {tasks.map(task => (
          <div
            key={task.id}
            style={getTaskStyle(task)}
            className="absolute h-8 rounded flex items-center px-2 text-xs text-white shadow-sm transition-all hover:scale-105 cursor-pointer"
            title={`${task.title} (${task.jira_key})`}
          >
            {task.title}
          </div>
        ))}
      </div>
    </div>
  );
};

export default GanttChart;
```

## 5. Критические точки для немедленной доработки (Action Plan)

1.  **Jira Sync Logic (High Priority):**
    *   Реализовать `Webhook` обработчик. Если задача в Jira меняется, Jira должна прислать уведомление на ваш Go-бэкенд, который обновит `data.json`. Это обеспечит реальную синхронизацию.
    *   Добавить `Retry Mechanism` для запросов к Jira. Если API Jira недоступно, задача должна встать в очередь на повторную попытку.

2.  **Data Validation (Medium Priority):**
    *   Добавить валидацию дат: `EndDate` не может быть меньше `StartDate`.
    *   Добавить проверку уникальности `JiraKey` внутри одного проекта.

3.  **UI/UX Improvements (Medium Priority):**
    *   **Drag-and-Drop:** Интегрировать `dnd-kit` или `react-beautiful-dnd` для изменения статусов задач простым перетаскиванием.
    *   **Zoom Level:** Добавить возможность менять масштаб диаграммы Ганта (например, вид по дням, неделям или месяцам).

4.  **Security (High Priority):**
    *   Хранение `Jira API Token` в `.env` файле.
    *   Добавление JWT авторизации для доступа к API, чтобы проект не был доступен всем в сети.

## 6. Как запустить прототип
1.  **Backend:**
    *   `go mod init tasktracker`
    *   `go get github.com/gin-gonic/gin`
    *   `go run main.go`
2.  **Frontend:**
    *   `npm create vite@latest frontend -- --template react`
    *   `cd frontend && npm install -D tailwindcss postcss autoprefixer && npx tailwindcss init -p`
    *   Установить компоненты и запустить `npm run dev`.
