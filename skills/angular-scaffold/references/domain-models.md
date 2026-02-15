# Modelos de Dominio - Trello Clone

## Tabla de Contenidos

1. [User](#user)
2. [Board](#board)
3. [List](#list)
4. [Card](#card)
5. [Todo/Checklist](#todochecklist)
6. [Relaciones entre Entidades](#relaciones-entre-entidades)

---

## User

```typescript
// src/app/models/user.model.ts
export interface User {
  id: string;
  email: string;
  name: string;
  avatar?: string;
  createdAt: string;
  updatedAt: string;
}

export interface CreateUserDto {
  email: string;
  password: string;
  name: string;
}

export interface UpdateUserDto {
  name?: string;
  avatar?: string;
}

// Auth-related
export interface LoginDto {
  email: string;
  password: string;
}

export interface LoginResponse {
  accessToken: string;
  refreshToken: string;
  user: User;
}
```

---

## Board

```typescript
// src/app/models/board.model.ts
import { List } from './list.model';
import { User } from './user.model';

export interface Board {
  id: string;
  title: string;
  backgroundColor: string;
  members: User[];
  lists: List[];
  createdAt: string;
  updatedAt: string;
}

export interface CreateBoardDto {
  title: string;
  backgroundColor?: string;
}

export interface UpdateBoardDto {
  title?: string;
  backgroundColor?: string;
}

// Para vistas de lista (sin relaciones anidadas)
export interface BoardSummary {
  id: string;
  title: string;
  backgroundColor: string;
  membersCount: number;
}
```

---

## List

```typescript
// src/app/models/list.model.ts
import { Card } from './card.model';

export interface List {
  id: string;
  title: string;
  position: number;
  boardId: string;
  cards: Card[];
  createdAt: string;
  updatedAt: string;
}

export interface CreateListDto {
  title: string;
  position?: number;
  boardId: string;
}

export interface UpdateListDto {
  title?: string;
  position?: number;
}

// Para ordenamiento
export interface ListPosition {
  id: string;
  position: number;
}
```

---

## Card

```typescript
// src/app/models/card.model.ts
import { Todo } from './todo.model';
import { User } from './user.model';

export interface Card {
  id: string;
  title: string;
  description?: string;
  position: number;
  listId: string;
  boardId: string;
  coverImage?: string;
  dueDate?: string;
  labels: Label[];
  assignees: User[];
  todos: Todo[];
  createdAt: string;
  updatedAt: string;
}

export interface CreateCardDto {
  title: string;
  description?: string;
  position?: number;
  listId: string;
}

export interface UpdateCardDto {
  title?: string;
  description?: string;
  position?: number;
  listId?: string;  // Para mover entre listas
  coverImage?: string;
  dueDate?: string;
}

// Para drag & drop
export interface CardPosition {
  id: string;
  position: number;
  listId: string;
}

export interface Label {
  id: string;
  name: string;
  color: string;
}
```

---

## Todo/Checklist

```typescript
// src/app/models/todo.model.ts
export interface Todo {
  id: string;
  title: string;
  isCompleted: boolean;
  position: number;
  cardId: string;
  createdAt: string;
  updatedAt: string;
}

export interface CreateTodoDto {
  title: string;
  cardId: string;
  position?: number;
}

export interface UpdateTodoDto {
  title?: string;
  isCompleted?: boolean;
  position?: number;
}

// Para batch updates
export interface TodoBatchUpdate {
  id: string;
  isCompleted: boolean;
}
```

---

## Relaciones entre Entidades

```
User (1) ────────────────── (N) Board (como owner/member)
  │
  └──── (N) Card (como assignee)

Board (1) ────────────────── (N) List
  │
  └──── (N) Card (relación indirecta via List)

List (1) ─────────────────── (N) Card

Card (1) ─────────────────── (N) Todo
  │
  ├──── (N) Label
  └──── (N) User (assignees)
```

### Index File para Exportar Modelos

```typescript
// src/app/models/index.ts
export * from './user.model';
export * from './board.model';
export * from './list.model';
export * from './card.model';
export * from './todo.model';
```

### Uso con Aliases

```typescript
// En cualquier archivo del proyecto
import { User, Board, Card, CreateCardDto } from '@models/index';
// o
import { User } from '@models/user.model';
```

---

## Tipos Utilitarios

```typescript
// src/app/models/common.model.ts

// Respuesta paginada genérica
export interface PaginatedResponse<T> {
  data: T[];
  total: number;
  page: number;
  limit: number;
  totalPages: number;
}

// Parámetros de paginación
export interface PaginationParams {
  page?: number;
  limit?: number;
  sortBy?: string;
  sortOrder?: 'asc' | 'desc';
}

// Respuesta de API genérica
export interface ApiResponse<T> {
  success: boolean;
  data: T;
  message?: string;
}

// Error de API
export interface ApiError {
  statusCode: number;
  message: string;
  error?: string;
}

// Para items con posición (drag & drop)
export interface Positionable {
  id: string;
  position: number;
}

// Constante para calcular posiciones
export const POSITION_BUFFER = 65535;
```
