# Patrones de Servicios Angular

## Tabla de Contenidos

1. [Auth Service Completo](#auth-service-completo)
2. [CRUD Service Genérico](#crud-service-genérico)
3. [Service con Estado Reactivo](#service-con-estado-reactivo)
4. [Service con checkToken](#service-con-checktoken)

---

## Auth Service Completo

```typescript
// src/app/services/auth.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Router } from '@angular/router';
import { BehaviorSubject, Observable, tap, switchMap, of } from 'rxjs';
import { getCookie, setCookie, removeCookie } from 'typescript-cookie';
import jwtDecode, { JwtPayload } from 'jwt-decode';

import { environment } from '@environments/environment';
import { User, LoginResponse, RegisterDto } from '@models/auth.model';
import { checkToken } from '@interceptors/token.interceptor';

@Injectable({
  providedIn: 'root'
})
export class AuthService {
  private apiUrl = `${environment.API_URL}/api/v1/auth`;
  
  private _user$ = new BehaviorSubject<User | null>(null);
  readonly user$ = this._user$.asObservable();

  constructor(
    private http: HttpClient,
    private router: Router
  ) {}

  // --- Getters ---
  
  get currentUser(): User | null {
    return this._user$.getValue();
  }

  getAccessToken(): string | null {
    return getCookie('token') || null;
  }

  getRefreshToken(): string | null {
    return getCookie('refreshToken') || null;
  }

  // --- Auth Methods ---

  login(email: string, password: string): Observable<LoginResponse> {
    return this.http.post<LoginResponse>(`${this.apiUrl}/login`, { email, password })
      .pipe(
        tap(response => this.handleAuthResponse(response))
      );
  }

  register(data: RegisterDto): Observable<User> {
    return this.http.post<User>(`${this.apiUrl}/register`, data);
  }

  logout(): void {
    removeCookie('token');
    removeCookie('refreshToken');
    this._user$.next(null);
    this.router.navigate(['/auth/login']);
  }

  refreshToken(refreshToken: string): Observable<LoginResponse> {
    return this.http.post<LoginResponse>(`${this.apiUrl}/refresh-token`, { refreshToken })
      .pipe(
        tap(response => this.handleAuthResponse(response))
      );
  }

  // --- Profile ---

  getProfile(): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/profile`, { context: checkToken() })
      .pipe(
        tap(user => this._user$.next(user))
      );
  }

  // --- Recovery ---

  requestPasswordReset(email: string): Observable<void> {
    return this.http.post<void>(`${this.apiUrl}/recovery`, { email });
  }

  resetPassword(token: string, newPassword: string): Observable<void> {
    return this.http.post<void>(`${this.apiUrl}/change-password`, { token, newPassword });
  }

  // --- Token Validation ---

  isValidToken(): boolean {
    const token = this.getAccessToken();
    if (!token) return false;

    try {
      const decoded = jwtDecode<JwtPayload>(token);
      if (!decoded.exp) return false;
      
      const tokenDate = new Date(0);
      tokenDate.setUTCSeconds(decoded.exp);
      return tokenDate.getTime() > Date.now();
    } catch {
      return false;
    }
  }

  // --- Private Helpers ---

  private handleAuthResponse(response: LoginResponse): void {
    const { accessToken, refreshToken, user } = response;
    
    setCookie('token', accessToken, { path: '/' });
    setCookie('refreshToken', refreshToken, { path: '/' });
    
    this._user$.next(user);
  }
}
```

### Modelos de Auth

```typescript
// src/app/models/auth.model.ts
export interface User {
  id: string;
  email: string;
  name: string;
  avatar?: string;
  createdAt: string;
  updatedAt: string;
}

export interface LoginResponse {
  accessToken: string;
  refreshToken: string;
  user: User;
}

export interface RegisterDto {
  email: string;
  password: string;
  name: string;
}

export interface ChangePasswordDto {
  token: string;
  newPassword: string;
}
```

---

## CRUD Service Genérico

```typescript
// src/app/services/base-crud.service.ts
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { environment } from '@environments/environment';
import { checkToken } from '@interceptors/token.interceptor';

export abstract class BaseCrudService<T, CreateDto, UpdateDto> {
  protected apiUrl: string;

  constructor(
    protected http: HttpClient,
    protected endpoint: string
  ) {
    this.apiUrl = `${environment.API_URL}/api/v1/${endpoint}`;
  }

  getAll(): Observable<T[]> {
    return this.http.get<T[]>(this.apiUrl, { context: checkToken() });
  }

  getById(id: string): Observable<T> {
    return this.http.get<T>(`${this.apiUrl}/${id}`, { context: checkToken() });
  }

  create(data: CreateDto): Observable<T> {
    return this.http.post<T>(this.apiUrl, data, { context: checkToken() });
  }

  update(id: string, data: UpdateDto): Observable<T> {
    return this.http.put<T>(`${this.apiUrl}/${id}`, data, { context: checkToken() });
  }

  delete(id: string): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`, { context: checkToken() });
  }
}
```

### Implementación Concreta

```typescript
// src/app/services/boards.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { BehaviorSubject, Observable, tap } from 'rxjs';
import { environment } from '@environments/environment';
import { checkToken } from '@interceptors/token.interceptor';
import { Board, CreateBoardDto, UpdateBoardDto } from '@models/board.model';

@Injectable({
  providedIn: 'root'
})
export class BoardsService {
  private apiUrl = `${environment.API_URL}/api/v1/boards`;
  
  // Estado reactivo para background color
  private _backgroundColor$ = new BehaviorSubject<string>('#ffffff');
  readonly backgroundColor$ = this._backgroundColor$.asObservable();

  constructor(private http: HttpClient) {}

  // --- CRUD Operations ---

  getBoards(): Observable<Board[]> {
    return this.http.get<Board[]>(this.apiUrl, { context: checkToken() });
  }

  getBoard(id: string): Observable<Board> {
    return this.http.get<Board>(`${this.apiUrl}/${id}`, { context: checkToken() })
      .pipe(
        tap(board => this.setBackgroundColor(board.backgroundColor))
      );
  }

  createBoard(data: CreateBoardDto): Observable<Board> {
    return this.http.post<Board>(this.apiUrl, data, { context: checkToken() });
  }

  updateBoard(id: string, data: UpdateBoardDto): Observable<Board> {
    return this.http.put<Board>(`${this.apiUrl}/${id}`, data, { context: checkToken() });
  }

  deleteBoard(id: string): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`, { context: checkToken() });
  }

  // --- Background Color ---

  setBackgroundColor(color: string): void {
    this._backgroundColor$.next(color);
  }

  // --- Position Helpers ---

  getPositionNewItem(items: { position: number }[]): number {
    const BUFFER_SPACE = 65535;
    if (items.length === 0) return BUFFER_SPACE;
    
    const lastPosition = Math.max(...items.map(item => item.position));
    return lastPosition + BUFFER_SPACE;
  }

  getPositionBetween(prevPosition: number, nextPosition: number): number {
    return Math.round((prevPosition + nextPosition) / 2);
  }
}
```

---

## Service con Estado Reactivo

```typescript
// src/app/services/lists.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { BehaviorSubject, Observable, tap, map } from 'rxjs';
import { environment } from '@environments/environment';
import { checkToken } from '@interceptors/token.interceptor';
import { List, CreateListDto, UpdateListDto } from '@models/list.model';

@Injectable({
  providedIn: 'root'
})
export class ListsService {
  private apiUrl = `${environment.API_URL}/api/v1`;
  
  // Store de listas por board
  private _lists$ = new BehaviorSubject<Map<string, List[]>>(new Map());
  
  constructor(private http: HttpClient) {}

  // Selector para obtener listas de un board específico
  getListsByBoard$(boardId: string): Observable<List[]> {
    return this._lists$.pipe(
      map(listsMap => listsMap.get(boardId) || [])
    );
  }

  // --- API Calls ---

  fetchLists(boardId: string): Observable<List[]> {
    return this.http.get<List[]>(
      `${this.apiUrl}/boards/${boardId}/lists`,
      { context: checkToken() }
    ).pipe(
      tap(lists => {
        const currentMap = this._lists$.getValue();
        currentMap.set(boardId, lists);
        this._lists$.next(new Map(currentMap));
      })
    );
  }

  createList(boardId: string, data: CreateListDto): Observable<List> {
    return this.http.post<List>(
      `${this.apiUrl}/boards/${boardId}/lists`,
      data,
      { context: checkToken() }
    ).pipe(
      tap(newList => {
        const currentMap = this._lists$.getValue();
        const boardLists = currentMap.get(boardId) || [];
        currentMap.set(boardId, [...boardLists, newList]);
        this._lists$.next(new Map(currentMap));
      })
    );
  }

  updateList(boardId: string, listId: string, data: UpdateListDto): Observable<List> {
    return this.http.put<List>(
      `${this.apiUrl}/boards/${boardId}/lists/${listId}`,
      data,
      { context: checkToken() }
    ).pipe(
      tap(updatedList => {
        const currentMap = this._lists$.getValue();
        const boardLists = currentMap.get(boardId) || [];
        const index = boardLists.findIndex(l => l.id === listId);
        if (index !== -1) {
          boardLists[index] = updatedList;
          currentMap.set(boardId, [...boardLists]);
          this._lists$.next(new Map(currentMap));
        }
      })
    );
  }

  deleteList(boardId: string, listId: string): Observable<void> {
    return this.http.delete<void>(
      `${this.apiUrl}/boards/${boardId}/lists/${listId}`,
      { context: checkToken() }
    ).pipe(
      tap(() => {
        const currentMap = this._lists$.getValue();
        const boardLists = currentMap.get(boardId) || [];
        currentMap.set(boardId, boardLists.filter(l => l.id !== listId));
        this._lists$.next(new Map(currentMap));
      })
    );
  }

  // Limpiar estado al salir del board
  clearBoardLists(boardId: string): void {
    const currentMap = this._lists$.getValue();
    currentMap.delete(boardId);
    this._lists$.next(new Map(currentMap));
  }
}
```

---

## Service con checkToken

Patrón para endpoints protegidos que requieren autenticación:

```typescript
// Importar la función checkToken del interceptor
import { checkToken } from '@interceptors/token.interceptor';

// Uso en llamadas HTTP que requieren autenticación
getProtectedResource(): Observable<Resource> {
  return this.http.get<Resource>(
    `${this.apiUrl}/protected`,
    { context: checkToken() }  // <-- Activa el interceptor de token
  );
}

// Llamadas sin autenticación (públicas)
getPublicResource(): Observable<Resource> {
  return this.http.get<Resource>(`${this.apiUrl}/public`);
  // Sin context, el interceptor no añade el token
}
```

### Token Interceptor Completo

```typescript
// src/app/interceptors/token.interceptor.ts
import { Injectable } from '@angular/core';
import {
  HttpInterceptor,
  HttpRequest,
  HttpHandler,
  HttpEvent,
  HttpContext,
  HttpContextToken,
  HttpErrorResponse
} from '@angular/common/http';
import { Observable, throwError, BehaviorSubject } from 'rxjs';
import { catchError, filter, take, switchMap, finalize } from 'rxjs/operators';
import { AuthService } from '@services/auth.service';

const CHECK_TOKEN = new HttpContextToken<boolean>(() => false);

export function checkToken() {
  return new HttpContext().set(CHECK_TOKEN, true);
}

@Injectable()
export class TokenInterceptor implements HttpInterceptor {
  private isRefreshing = false;
  private refreshTokenSubject = new BehaviorSubject<string | null>(null);

  constructor(private authService: AuthService) {}

  intercept(req: HttpRequest<unknown>, next: HttpHandler): Observable<HttpEvent<unknown>> {
    // Solo procesar si el contexto tiene CHECK_TOKEN
    if (!req.context.get(CHECK_TOKEN)) {
      return next.handle(req);
    }

    // Añadir token si existe
    const token = this.authService.getAccessToken();
    if (token) {
      req = this.addToken(req, token);
    }

    return next.handle(req).pipe(
      catchError((error: HttpErrorResponse) => {
        if (error.status === 401) {
          return this.handle401Error(req, next);
        }
        return throwError(() => error);
      })
    );
  }

  private addToken(req: HttpRequest<unknown>, token: string): HttpRequest<unknown> {
    return req.clone({
      headers: req.headers.set('Authorization', `Bearer ${token}`)
    });
  }

  private handle401Error(req: HttpRequest<unknown>, next: HttpHandler): Observable<HttpEvent<unknown>> {
    if (!this.isRefreshing) {
      this.isRefreshing = true;
      this.refreshTokenSubject.next(null);

      const refreshToken = this.authService.getRefreshToken();
      
      if (!refreshToken) {
        this.authService.logout();
        return throwError(() => new Error('No refresh token available'));
      }

      return this.authService.refreshToken(refreshToken).pipe(
        switchMap(response => {
          this.refreshTokenSubject.next(response.accessToken);
          return next.handle(this.addToken(req, response.accessToken));
        }),
        catchError(error => {
          this.authService.logout();
          return throwError(() => error);
        }),
        finalize(() => {
          this.isRefreshing = false;
        })
      );
    }

    // Si ya estamos refrescando, esperar al nuevo token
    return this.refreshTokenSubject.pipe(
      filter(token => token !== null),
      take(1),
      switchMap(token => next.handle(this.addToken(req, token!)))
    );
  }
}
```
